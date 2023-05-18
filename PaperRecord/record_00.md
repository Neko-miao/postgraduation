# Techniques for Skeletal-Based Animation in Massive Crowd Simulations

## abstraction
- improving the performance, variety, and usablity of crowd annimation systems.

## 1. contribution
- Improving the memory footprint to support different skeletons and animation offsets.  
- Animation Transiton allows states and reactions  
- GPU blendng  
- Modularized agent animation  
- Improving the authoring time for agent animations  

## 2. Related Work  
- vertex skinning -- skinned instancing  
- animation texture  
- low LOD  
- Image-based approaches to simplify the large crowd rendering by in an urban simulation.  
- cage-based method for optimizing  
- decision mechanism at the simulaton layer feeds thier animation blending mechanism with concrete parameters.  
- motion warping & motion graph  
- registration curve used to identify the portion between two sufficiently similar motion frames.  
- Style Machines  
- animation compression -- PCA, clustering  
- defect of paper[1], unresponsive to behaviors and emergent actions, only a single type of skeleton  
- automatic skinning and weiight retargeting  
- a method to imprive skining time with a loss of quality, not suitable for characters that are animated close to the camera  


## 3. Current Mehtods for Crowd Animation  
- how skeletal animation works  
- how implement crowd animation with existing solution  
- new requirements for different crowd systems  

### 3.1 Basics  
- pose and binding pose  
- skinning matrix  
    $$v' = \sum_i^nw_iM_iM_{ref_i}^{-1}v,\quad where \sum_i^nw_i = 1$$

### 3.2 Current Techniques for Sharing Aniamtion  
- animation texture: a row for a single pose of a single animation, a bone's transformation need a 4 * 3 matrix, a texel can store 4 floats  
- weight $w_i$ and $M_{ref_i}^{-1}$ matrix are constant and provided at the initialization time, and $M_i$ can be sampled frome animation texture accroding to current time and animation index assigned to each vertex.
- using skinning matrix to calculate every skinning vertex transformation. 
- offset system: using to share animation among agents.  
    if 2 agents share the same animation, then the agents in that animation can have different time offsets assigned randomly.  

### 3.3 Current Limitations and Requirements  
- new requirements:  
    1. states machine become common, with transitiona among clips decided by a decision-maker layer on the simulation side.  
    2. 2D blending depending on parameters fed in at runtime  
    3. animation events support  
    4. animation layers support  

- a mixed model in paper[1] -- streaming animations data from CPU to GPU and storing only a part of the animation dataset on the GPU memory.
![](https://s3.bmp.ovh/imgs/2023/05/16/32401de2df72e942.png)

- drawbacks:
    1. the pose storage was duplicated  
    2. animation texture width size should set to maximum for multiple skeletons

### 3.4 Animation Controllers  
- a pose evaluated by animation controller at time $t$ is a conbination of the poses recursively evaluated by its children.  
- a controller can blend two animations or more animations with annimation layers  
- state machines is supported and runing at CPU sides  

## 4. Crowd Animation in Our Framework  
### 4.1 Animation Streams  
- a visulal animation: $ AnimDef = (Skeleton, Skin mesh) $
- an animation stream A: $ AnimStream_A = (AnimaDef_A, P_A, State_A, Ctrl_A, M_A, PH_A) $  
    - $P_A$ is a pose buffer representing the transfomation for each bone at the current time of animation, and $P_A(i)$ represents the transformation of bone at index of $i$.  
    - $Ctrl_A$ represents the root animation controller of animation A.  
    - $PH_A$ is a circular buffer storing the history of the last $M_A$ values evaluated for $P_A$
    - $M_A$ is user configurable and represents the maximum number of frames to be stored in the $PH_A$ buffer  
    - $State_A$ is in $(Running, Stopped)$, and when $Running$ state is active, its animation controller is updated by writing the **pose output** in $P_A$ and copying it to the current head of $PH_A$.  
- $UserConfig = (StreamPool:AnimStream[], UniquePool:Anim[])$  
    - $StreamPool$ contains animations that can be shared among multiple agents  
    - $UniquePool$ is a dynamic array of animation definitions that are not intended to be shared among agents.  

### 4.2 Splitting Agent and Animation into Parts  
- defects of single skeleton animation for the entire crowd:
    - addition memory needed
    - hard to extent skeleton structure  
- splitting agent parts in clusters and allowing each cluster to be animated by a different animation stream.  
- each crowd agent has a unique $id$, then its animation reference can be defined as tuple:  
    $$ AgentAnim_{ID} = {(ClusterPart_k : (ID_{SI}, ID_O)) | for \, each \, parts \, cluster \, k \, defined} $$  
    - $ID_{SI}$ represents the animation index of a cluster  
    - $ID_O$ is the offsets in the history of stored poses of that animation stream($PS_{SI}$)  
- the pose history is uesd for animation diversity by randomization at both the stream index and offset levels, which is levered by the amount of animations streams and $M_A$  
- to correctly synchronize different parts of a agent, each part has a static definition of its parent hook position and transformation (like vitual bone in UE), and the culling of parts at shader level will be as easy as seting the corresponding bit of the cluster in the agent's GPU instance buffer entry to $0$  

### 4.3 Transitions and Time Dilation
- transition from the current animation of part cluster to another stream loaded in the pools can be requested by APIs:
    - <code>TransitionToStreamed(ID, streamIndex)</code>  
    - <code>TransitionToUnique(ID, uniqueIndex)</code>  
- at transition beginning, animation offset cannot be greater than 0, which may limit the visual appearance of the crowd  
- *time dilation* solves  
   - $animFrameRate$ : animation sampling frequency for pose buffers  
   - $M_A$ : current maximum offset  
   - $M_{new_A}$ : maximum new offset  
    ```
    /* Starting a new animation stream and updating agent offsets for each simulation frame */  
    OnNewAnimStreamStart(A:AnimStream, timeToDilate)
        // The maximum amount of offset to change  
        // per agent and simulation frame  
        A.changeRate = (A.Mnew - A.M) / (timeToDilate * animFrameRate)  
        for each agent part Ag using A
            Ag.offset = 0
            Ag.targetOffest = random(0, A.Mnew)

    OnAnimSystemUpdate()
        // ...
        for each changing anim stream A
            for each agent part Ag using A
                Ag.offset = min(Ag.offset + A.changeRate, Ag.targetOffset)
        // ...

    ```

### 4.4 Blending between Animation Streams  
- perform the computations of blending on GPU instead of CPU, meaning the interpolations are computed in the skinning vertex sahder code  
- the information of the pose history buffer of each animation state is synchronized between the CPU and GPU  
- using fixed offset N, transition starts with the source animation stream and the original offset, and ends up with target animation stream and the target offset  

```
// compute the skinning vertex position at in shader code
VertexPos skinVertex(VeretexPos inputVertex, agentId, clusterId)
    data = instanceBuffer[agentId, clusterId]

    // get the vertext skinned according to the source animation  
    VertexPos srcVertex = skinVertex(data.srcAnimStream, data.srcOffset)  
    if agentData.isBlending == 0
        return srcVertex  
    
    VertexPos targetVertex = skinVertex(data.targetAnimStream, data.targetOffset)  
    timeElapsed = (globalTimer - data.transitionStartTime)  
    alpha = clamp(timeElapsed * invBlendTime, 0, 1)  
    VertexPos res = lerp(scrVertex, targetVertex, alpha)
    return res
```
![](https://s3.bmp.ovh/imgs/2023/05/17/4115262b669b4b7e.png)

### 4.5 Memory Layout and the CPU-GPU Data Flow  
![](https://s3.bmp.ovh/imgs/2023/05/17/42df4c9fd0dc5887.png)
![](https://s3.bmp.ovh/imgs/2023/05/17/b4f36affd09a2c82.png)

### 4.6 Skinning Shader and Custom Animations per Agent  
- mark bones so that they are overridden by a custom vertex shader code authored by the user, and the vertices depend on these bones will not read the animation data from the ose buffer  

## 5. Evaluation  
## 6. Conclusions and Future Work  
- dynamic LoD  
- transition between skeletal and cage/impostor aniamtion  
