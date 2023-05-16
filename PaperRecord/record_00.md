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

### Current Techniques for Sharing Aniamtion  
- animation texture: a row for a single pose of a single animation, a bone's transformation need a 4 * 3 matrix, a texel can store 4 floats  
- weight $w_i$ and $M_{ref_i}^{-1}$ matrix are constant and provided at the initialization time, and $M_i$ can be sampled frome animation texture accroding to current time and animation index assigned to each vertex.
- using skinning matrix to calculate every skinning vertex transformation. 
- offset system: using to share animation among agents.  
    if 2 agents share the same animation, then the agents in that animation can have different time offsets assigned randomly.  

### Current Limitations and Requirements  
- new requirements:  
    1. states machine become common, with transitiona among clips decided by a decision-maker layer on the simulation side.  
    2. 2D blending depending on parameters fed in at runtime  
    3. animation events support  
    4. animation layers support  

