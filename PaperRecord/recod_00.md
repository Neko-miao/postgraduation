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
