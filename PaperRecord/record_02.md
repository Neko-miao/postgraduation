# Parallel LOD for CAD model rendering with efective GPU memory usage

## GPU-accelerated mesh simplification  
- parallel
- dependency-free  
- continuous LODs

### GPU-friendly Preprocessing  
- **edge-collapsing**: decrease amount of edges by combining the end vertics of the edge    
- **boundary edge**: edge that is only belong to one triangle  
- **boundary vertex**: vertices of boundary edges  
- **no permission for boundary edges collapsing**  
- data struct  
    - ECols: array representing edge collapsing operation. ECols[i] is target vertex of vertex i
    - PermuteT: array of all triagnles recording the permuting triangle after edge collapsing. The first removed triangle is put to the last position in the triangle set. For instance, if edge E is collapsed, and consequently triangle T_i is removed, then permuteT[i]=m, m is current amount of triangles.  
    - PermuteV: similiar to PermuteT  
    - Map: array to record amout of triangles according to the amount of vertices. (e.g. If there are 7 vertices and 8 triangles, then Map[7]=8)
- Mesh Simplification Procedures:
1. **choose a edge to collapse**: calculate the cost of each edge, and choose the edge of minimous cost to collapse.  If we merge v_src to v_target, then ECols[v_src]=v_target 
2. **Permutation for vertices**: assume m vertices remaining for now, and PermuteV[v_src]=m after edge collapsing by procedure 1.  
3. **Vertex-Triangle count**: m is current amount of vertices, n is current amount of triangles, after one edge is collapsed, k triangles decrease, so Map[m - 1] = n - k  
4. **Permutation for Triangles**: similar to prcedure 2. If 2 triangles are removed, call them t1 and t2, and Index(t1) < Index(t2). We record PermuteT[t1] = n, and PermuteT[t2] = n - 1.  
5. **Iteration**: Iteration of above procedures util all there rest are boundary edges.  

### Triangle Refomation  
- data rearrangement:  
- triangle reformation:  
- when there are m vertices, we get Map[m] = n triangles.  And then we identify the 3 vertices of each triangles by the result of data rearrangement. And then we reform these n triangles and finnaly get the simplified mesh.  

**And all is to determine the numerber of vertices.**  

### GPU out-of-core  
technology for data transferring from CPU to GPU  

## LOD selection  
### First-pass algorithm  
- equation (1) to determin the number of each mesh  
- equation (2) to valid the numbers  

### Second-pass algorithm  
to make sure the efficient usage of GPU memory  



AFTTER ALL, WE RENDER MESH