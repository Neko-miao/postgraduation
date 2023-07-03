# Real-Time Large Crowd Rendering with Efficient Character and Instance Management on GPU  

## Abstraction  
A real-time crowd rendering system on GPU, integrating with dynamic LoD, visibility culling.

## Design & Realization  
### System Overview  
- preprocess: fine-to-coarse progressive mesh simplification algorithm  
- runtime task: on GPU through 5 parallel stage  
    - View-Frustum Culling  
    - LoD Selection  
    - LoD Mesh Generation  
    - Animating Meshes  
    - Rednering  

### LoD Selection  
$$vNum_i=N\frac{w_i^{1/\alpha}}{\sum_{i=1}^K{w_i^{1/\alpha}}},\quad where\quad w_i=\beta\frac{A_i}{D_i}P_i^\beta,\quad\beta=\alpha-1$$
- $K$: total number of instance  
- $w_i$: weight of amount of vertices  
- $A_i$: area of the bounding shpere
- $D_i$: distance between instance $i$ and viewpoint  
- $P_i$: number of primitive of instance $i$  
- $\alpha,\beta$: controllable parameter
- $vNum_i$: vertices number of instance $i$  
- $N$: maximum capacity of vertices on the GPU  

### Animation  
- No new vertex added after LoD Selection.
- Calculate skinned vetices by Skinning Matrix  

### Packing Skeleton, Animation, and Skinning Weights into Texture  

### UV-Guided Mesh Rebuilding  



