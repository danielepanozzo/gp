# Assignment 5: Shape Deformation

In this exercise, you will implement an algorithm to interactively deform 3D models. You will construct a two-level multi-resolution surface representation and use naive Laplacian editing to deform it.

---
## Multiresolution mesh editing
For this task, you will compute a mesh deformation based on the rotations and translations applied interactively to a subset of its vertices via the mouse. Let $H$ be the set of "handle" vertices that the user can manipulate (or leave fixed). We want to compute a deformation for the remaining vertices, denoted as $R$.

Let $\mathcal{S}$ be our input surface, represented as a triangle mesh. We want to compute a new surface that contains:
- the vertices in $H$ translated/rotated using the user-provided transformation $t$, and
- the vertices in $R$ properly deformed using the algorithm described next.

The algorithm is divided in three phases:

1. removing high-frequency details,
2. deforming the smooth mesh, and
3. transferring high-frequency details to the deformed surface.

![](img/schema.jpg)
*Fig. 1: Algorithm Overview*

### Selecting the handles
A minimal, lasso-based interface for selecting vertices has been implemented for you. To use it, enable the `SELECT` mouse mode from the menu (or hit 'S'), click somewhere `on` the mesh and drag with your mouse to draw a stroke around the area of the mesh you want to select as a handle. The vertices inside the stroked region are saved in the `selected_v` variable. You have the options to: (a) accept the selected vertices as a new handle region (only if the vertices are not assigned to a handle already) by hitting the relevant button on the menu or key 'A' (Fig.~\ref{fig:smoothing}(a)), (b) discard the selection and make a new one by drawing another stroke somewhere on the mesh. Once a selection is accepted, you can add additional handles by drawing more strokes.

As selections are accepted, their vertices are saved in the `handle_vertices` variable. We also store the handle index for each vertex in \texttt{handle\_id} (-1 if the vertex belongs to no handle). The handle region centroids are stored in \texttt{handle\_centroids}.

The selected handles can be transformed by selecting the appropriate mouse mode (`TRANSLATE` / `ROTATE`, shortcuts: ALT+'T', ALT+'R') and dragging with the mouse. While handles are dragged, the updated handle vertex positions are stored in \texttt{handle\_vertex\_positions}.

---
### Step 1: Removal of high-frequency details

<img align="left" width="200" src="img/hand.png"> 
<img align="left" width="200" src="img/hand_b.png">
<img align="left" width="200" src="img/woody.png">
<img align="left" width="200" src="img/woody_b.png">
<br clear="both"/>

*Fig. 2: Input and Smoothed Meshes*

We remove the high-frequency details from the vertices $R$ in $\mathcal{S}$ by minimizing the thin-plate energy, which involves solving a bi-Laplacian system arising from the quadratic energy minimization:

$$
\begin{aligned} \min_\textbf{v} & \quad\textbf{v}^T \textbf{L}_\omega \textbf{M}^{-1} \textbf{L}_\omega \textbf{v} \\
 \text{subject to}&
 \quad \textbf{v}_H = \textbf{o}_H,
\end{aligned}
$$

where $\textbf{o}_H$ are the handle $H$'s vertex positions, $\textbf{L}_\omega$ is the cotan Laplacian of $\mathcal{S}$, and $\textbf{M}$ is the mass matrix of $\mathcal{S}$.
Notice that $\textbf{L}_\omega$ is the symmetric matrix consisting of the cotangent weights ONLY (without the division by Voronoi areas). In other words, it evaluates an "integrated" Laplacian rather than an "averaged" laplacian when applied to a vector of vertices. The inverse mass matrix appearing in the formula above then applies the appropriate rescaling so that the laplacian operator can be applied again (i.e., so that the Laplacian value computed at each vertex can be interpreted as a piecewise linear scalar field whose Laplacian can be computed).
This optimization will produce a mesh similar to the one in Figure. Note that the part of the surface that we want to deform is now free of high-frequency details. We call this mesh $\mathcal{B}$.

---
### Step 2: Deforming the smooth mesh
<img width="200" src="img/hand_t.png"> 
<img width="200" src="img/woody_t.png">
<br clear="both"/>

*Fig. 3: Deformed/Smoothed Meshes*

The new deformed mesh is computed similarly to the previous step, by solving the minimization:
$$
\begin{aligned} \min_\textbf{v}& \quad \textbf{v}^T \textbf{L}_\omega \textbf{M}^{-1} \textbf{L}_\omega \textbf{v} \\
 \text{subject to}&
 \quad \textbf{v}_H = t(\textbf{o}_H),
\end{aligned}
$$
where $t(\textbf{o}_H)$ are the new handle vertex positions after applying the user's transformation. We call this mesh $\mathcal{B}'$.

---
### Step 3: Transferring high-frequency details to the deformed surface
<img align="left" width="200" src="img/hand_bd.png"> 
<img align="left" width="200" src="img/hand_td.png">
<img align="left" width="200" src="img/woody_bd.png">
<img align="left" width="200" src="img/woody_td.png">
<br clear="both"/>

*Fig 4: Displacements on $\mathcal{B}$ and $\mathcal{B}'$*

The high-frequency details on the original surface are extracted from $\mathcal{S}$ and transferred to $\mathcal{B}'$. We first encode the high-frequency details of $\mathcal{S}$ as displacements w.r.t. $\mathcal{B}$.
We define an orthogonal reference frame on every vertex $v$ of $\mathcal{B}$ using:
1. The unit vertex normal
2. The normalized projection of one of $v$'s outgoing edges onto the tangent plane defined by the vertex normal. A stable choice is the edge whose projection onto the tangent plane is longest.
3. The cross-product between (1) and (2)

For every vertex $v$, we compute the displacement vector that takes $v$ from $\mathcal{B}$ to $\mathcal{S}$ and represent it as a vector in $v$'s reference frame. 
For every vertex of $\mathcal{B}'$, we also construct a reference frame using the normal and the SAME outgoing edge we selected for $\mathcal{B}$ (not the longest in $\mathcal{B}'$; it is important that the edges used to build both reference frames are the same). We can now use the displacement vector components computed in the previous paragraph to define transferred displacement vectors in the new reference frames of $\mathcal{B}'$. See Figure 4 for an example.
Applying the transferred displacements to the vertices of $\mathcal{B}'$ generates the final deformed mesh $\mathcal{S}'$. See Figure 5 for an example.

<img align="left" width="200" src="img/hand_f.png">
<img align="left" width="200" src="img/woody_f.png">
<br clear="both"/>

*Fig 5: Final Deformation Results*

---
## Performance
To achieve real-time performance, you must prefactor the sparse bi-Laplacian matrix appearing in both linear systems. After the user specifies vertex sets $H$ and $F$, you can factorize the matrix $\textbf{L}_\omega \textbf{M}^{-1} \textbf{L}_\omega$ (using a Cholesky "$LL^T$" factorization) and then re-use the factorization to solve both linear systems efficiently. This is a mandatory part of the exercise; if your implementation does not achieve interactive frame-rates (10+ fps) on the provided meshes it will not receive the full score.


Required output of this section:
- Provide screenshots for 4 different deformed meshes. For each example, provide a rendering of $\mathcal{S}$, $\mathcal{B}$, $\mathcal{B}'$ and $\mathcal{S}'$.

---
### Optional task (bonus 5%)
- *(5 points)* Implement the high-frequency detail transfer using "deformation transfer", as explained the [paper](https://lgg.epfl.ch/publications/2006/botsch_2006_DTD.pdf) *Deformation Transfer for Detail-Preserving Surface Editing*, Botsch et al. 2006.