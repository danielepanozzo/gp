# Assignment 4: Mesh Parametrization

In this exercise you will

 * Familiarize yourself with vector field design on surfaces.
 * Create scalar fields whose gradients align with given vector fields as closely as possible.
 * Experiment with the `igl` implementation of harmonic and least-squares conformal parameterizations.


## 1. Tangent vector fields for scalar field design

Our first task is to design smooth tangent vector fields on a surface; these will be "integrated" later to define a scalar field.
A (piecewise constant) vector field on a triangle mesh is defined as an assignment of a single vector to each triangle such that each vector lies in the tangent plane containing the triangle.
We will design fields to follow a set of alignment constraints provided by the user: the user specifies the field vectors at a subset of the
mesh triangles (the constraints) and those constraints are interpolated smoothly throughout the surface to define a field.

![](img/vf.png?raw=true)

### Creating vector constraints
The provided assignment data already contains a selection of faces and vector constraints at such faces.

### Interpolating the constraints
Since each field vector `uf` lies in the plane of its respective triangle `f`, it can be decomposed into the triangle's local basis and represented with two real coefficients: `uf = (xf, yf)` in R<sup>2</sup>.
Alternatively, we can identify the triangle plane with the complex plane, expressing the vector as a single complex number `uf = xf + i yf` in C.

The interpolation produces a smooth vector field from the constraints by trying to make each vector as similar as possible to the adjacent triangles' vectors.
However, since vectors `uf`, `ug` at adjacent triangles `f`, `g` are expressed with respect to the triangles' local bases, and the triangles generally have different bases, their complex expressions cannot be compared directly (see figure).
So we must first express the vectors with respect to a common basis.

![](img/2.png?raw=true)

A simple way to choose a common basis is to use the shared (directed) edge vector as the new real (`x`) axis for both triangles. This implicitly also chooses the 90°-counterclockwise-rotated edge vector as the imaginary axis, forming a complete basis.
In this new common basis, the two triangles' vectors can be
written as<br/>
![](https://latex.codecogs.com/svg.latex?\tilde&space;u_f=u_fe_f^{-1}=u_f\overline{e_f},\qquad\tilde&space;u_g=u_g\overline{e_g})<br/>
<!-- $\tilde u_f = u_f e_f^{-1} = u_f \overline{e_f}$, $\tilde u_g = u_g
\overline{e_g}$. -->
Where `ef`, `eg` are the shared edge vector expressed in each local basis. Here the overline denotes the complex conjugate of a complex number, *and we assume `ef`, `eg` are normalized so that the conjugate is actually the inverse* - normalize them!

The difference ("non-smoothness") of the two vectors across the edge  `(f,g)` can now be computed as
<br/>
![](https://latex.codecogs.com/svg.latex?E_{fg}(u_f,u_g)=\|u_f\overline{e_f}-u_g\overline{&space;e_g}\|^2)<br/>
 <!-- $E_{fg}(u_f, u_g) = \| u_f \overline{e_f} - u_g \overline{ e_g}\|^2$ -->
(recall that `||z||^2 = z z.conjugate()` is a complex number `z`'s squared magnitude).
This is a quadratic form on the complex variables `uf`, `ug` and can be manipulated into the form
<br/>
![](https://latex.codecogs.com/svg.latex?E_{fg}(u_f,u_g)=\begin{bmatrix}\overline{u_f}&\overline{u_g}\end{bmatrix}Q_{fg}\begin{bmatrix}u_f&#92;&#92;u_g\end{bmatrix})<br/>
 <!-- $E_{fg}(u_f, u_g) =
\mtrx{\overline{u_f}&\overline{u_g}}  Q_{fg} \mtrx {u_f\\u_g}$  -->
for a particular
complex matrix `Q_fg`. The full field's smoothness can then be written as the sum of all these per-edge energies:
<br/>
![](https://latex.codecogs.com/svg.latex?\sum_{\textrm{edge}(f,g)}E_{fg}(u_f,u_g).)<br/>
<!-- $\sum_{\textrm{edge}(f,g) }E_{fg}(u_f, u_g)$.  -->
This yields a sparse quadratic form `u^* Q u`, where the complex column vector `u` encodes each each per-face vector. The row vector `u^*` is `u`'s adjoint (conjugate transpose), and `Q` is the appropriate combination of the matrices `Q_{fg}`.
Thus, finding the smoothest field under the prescribed constraints is equivalent solving
<br/>
![](https://latex.codecogs.com/svg.image?\begin{align*}\min&space;u^*&space;Q&space;u&space;\\&space;u|_{cf}&space;&=&space;c,\end{align*})<br/>
<!-- \begin{align*}
\min u^* Q u \\
 u|_{cf} &= c,
\end{align*} -->
where `cf` are the constrained face indices and `c` are the prescribed vectors at those faces. We can differentiate the smoothness energy to find its minimum, thus obtaining a (complex) linear system
<br/>
![](https://latex.codecogs.com/svg.image?\begin{align*}&space;&space;&space;&space;Q&space;u&space;&=&space;0&space;\\&space;&space;&space;&space;u|_{cf}&space;&=&space;c.\end{align*})<br/>
<!-- \begin{align*}
    Q u &= 0 \\
    u|_{cf} &= c.
\end{align*} -->
This system's solution describes the smoothly interpolated vector field.

Your task is

 * Determine how to construct the complex matrix `Q`,
 * Solve the system under the prescribed constraints (which are loaded as described above),
* Display the constraints and the interpolated field


*Note:* Numpy and Scipy support sparse complex matrices and can factorize/solve linear systems with
```python
import scipy.sparse as sp
u = sp.linalg.spsolve(A, b)
```
but feel free to convert this problem into real variables if you find that easier.

*Relevant `igl` functions:* `triangle_triangle_adjacency`, `local_basis` computes a basis for each triangle plane.

*Relevant `meshplot` functions:* `add_lines` plot lines between two points.

*Relevant `numpy` functions:* `zeros((rows, cols), dtype=np.complex)`, `v.astype(...)`, `dot`, `cross`

*Relevant `scipy` functions:* `M.H` adjoint of `M`, `sp.coo_matrix((data, (i, j)))` to initialize a matrix from triplets, `M.asformat("csr")` to compress a matrix, `M.asformat("lil")` to edit a matrix.

*Relevant `python` functions:* `1j` complex unit, `x.conjugate()` complex conjugate, `x.real`/`x.imag` real and imaginary part of a complex number.

Required output for this section:

 * Visualization of the constraints and interpolated field.
 * An ASCII dump of the interpolated field (#F x 3 matrix, one vector per row) for the mesh `irr4-cyl2.off` and the input constraints in the provided file `irr4-cyl2.constraints`.

## 2. Reconstructing a scalar field from a vector field
Your task is now to find a scalar function `S(x)` defined over the surface whose gradient fits a given vector field as closely as possible.
The scalar field is defined by values on the mesh vertices that are linearly interpolated over each triangle's interior: for given vertex values `si`, the function `S(x)` inside a triangle `t` is computed as
<br/>
![](https://latex.codecogs.com/svg.latex?S_t(x)=\sum\limits_{\textrm{vertex}~i~\in~t}^3s_i\phi_i^t(x))<br/>
<!-- $S_t(\x) =
\sum\limits_{\textrm{vertex}~i~\in~t}^3 s_i \phi_i^t(\x)$,  -->
where `phi_i^t(x)` are the linear "hat" functions associated with each triangle vertex (i.e. `phi_i^t(x)` is linear and takes the value 1 at vertex i and 0 at all other vertices).
Then the scalar function's (vector-valued) gradient is
<br/>
![](https://latex.codecogs.com/svg.latex?g_t=\nabla&space;S_t=\sum\limits_{\textrm{vertex}~i~\in~t}^3s_i\nabla\phi_i^t.)<br/>
<!-- $\vec g_t = \nabla S_t = \sum\limits_{\textrm{vertex}~i~\in~t}^3 s_i \nabla\phi_i^t$. -->

Since the "hat" functions are piecewise linear, their  gradients
<!-- $\nabla\phi_i^t$-->
are constant within each triangle, and so is `gt` (the full scalar function's gradient). Specifically, `gt` is a linear combination of the constant hat function gradients with the (unknown) values `si` as coefficients, meaning that we can write an expression of the form `g  = G s`, where `s` is a #V x 1 column vector holding each `si`, `g` is a column vector of size 3#F x 1 consisting of the vectors `gt` "flattened" and vertically stacked (i.e., using `gt.T.reshape(gt.size, 1)`), and `G` is the so-called "gradient matrix" of size 3#F x #V.

Since there is no guarantee that our interpolated face-based field is actually the gradient of some function, we cannot attempt to integrate it directly.
Instead, we will try to find `S(x)` by asking its gradient to approximate the vector field `u` in the least-squares sense:
<br/>
![](https://latex.codecogs.com/svg.latex?\min\quad\sum\limits_{\textrm{face}~t}A_t\|g_t-u_t\|^2,)<br/>
<!-- \bdm{
\min ~~\sum\limits_{\textrm{face}~t} A_t \|\vec g_t-\vec u_t\|^2,\\
} -->
where `A_t` is triangle `t`'s area, `g_t` is the (unknown) function
gradient on the triangle, and `u_t` is the triangle's vector assigned by
the guiding vector field.
Using the linear relationship `g = Gs`, we can write this least-squares error as a (real) quadratic form:
<br/>
![](https://latex.codecogs.com/svg.latex?s^TKs+s^Tb+c)<br/>
<!-- $$s^TKs + s^Tb + c$$ -->
and minimize it by solving a linear system for the unknown `s`.

Your task is

 * Determine the matrix `K` and vector `b` in the above minimization (by expanding the least-squares error expression).
 * Minimize by differentiating and equating the gradient to zero; this gives you a linear system to solve.
 * Display the scalar function on the surface using a color map and overlay its gradient vectors.
 * Plot the deviation between the input vector field and the solution scalar function's gradient (the "Poisson reconstruction error").


*Note:* the linear system is not full rank; `K` has a one dimensional nullspace corresponding to the constant function. This is because a scalar field can be offset by any constant value without altering its gradient. You will need to fix the value at one vertex (e.g., to zero) to solve the system.

<div align=center><img width=50% height=50% src=img/sf_1.png/><img width=50% height=50% src=img/sf_2.png/></div>


A vector field and the reconstructed scalar function. Note the scalar function gradient's deviation from the input field.

*Relevant `igl` functions:* `grad` calculates the gradient matrix explained above. `doublearea` can be used to compute triangle areas.

*Relevant `numpy` functions:* `reshape`, `matlib.repmat`

*Relevant `scipy` functions:* `diags`, `linalg.spsolve`

Required output for this section:

 * Visualization of computed scalar function and its gradient.
 * Plots of the Poisson reconstruction error.
 * An ASCII dump of the reconstructed scalar function (#V x 1 vector one vertex value per row) for the mesh `irr4-cyl2.off` and the input constraints in `irr4-cyl2.constraints`.

## 3. Harmonic and LSCM Parameterizations

For this task, you will experiment with flattening a mesh with a boundary onto the plane using two parameterization methods: `harmonic` and `Least Squares Conformal` (LSCM) parameterization. In both cases, two scalar fields, `U` and `V`, are computed over the mesh. The per-vertex (`u`, `v`) scalars defining these coordinate functions determine the vertices' flattened positions in the plane (the flattening is linearly interpolated within each triangle).

For the harmonic parametrization example, you will first map the mesh boundary to a unit circle in the UV plane centered at the origin. The boundary `U` and `V` coordinates are then "harmonically interpolated" into the interior by solving the Laplace equation with Dirichlet boundary conditions (setting the Laplacian of `U` equal to zero at each interior vertex, then doing the same for `V`). This involves two separate linear system solves (each with the same system matrix).

In LSCM, the boundary is free, with the exception of two vertices that must be fixed at two different locations in the UV-plane (to pin down a global position, rotation, and scaling factor). These vertices can be chosen arbitrarily. The process is again a linear system solve, but in this case the `U` and `V` functions are entwined into a single linear system.


Your task is

 * Use the `igl` implementation of harmonic and LSCM mappings to visualize of one of the two mapping functions (`U` or `V`) as a texture over the surface.
 * Compute the gradient of the selected mapping function and overlay it.

*Note* we suggest to use subplot in `meshplot`
```python
p = mp.subplot(v, f, uv=uv, s=[1, 2, 0])
mp.subplot(uv, f, uv=uv, shading={"wireframe": True}, data=p, s=[1, 2, 1])
```

![](img/param.png?raw=true)
From left to right: a harmonic parameterization, the gradient of its `V` function, a visualization of the flattened mesh on the `UV` plane, LSCM parameterization, and its `UV` domain.

<!-- \emph{Note:} you may need to scale up the parameterization to better visualize the texture lines (parametric curves). -->

*Relevant `igl` functions:* `harmonic_weights`, `lscm`, `boundary_loop`,`map_vertices_to_circle`.


*Relevant `meshplot` functions:* `plot(v, f, uv=uv)`


Required output for this section:

 * Visualization of the computed mapping functions and their gradients for LSCM and harmonic mapping.



## 4. Editing a parameterization with vector fields
A parameterization consists of two scalar coordinate functions on the surface.
As such, we can use vector fields to guide the parameterization: we can design a vector field and fit one of the coordinate functions' gradients to it.

### Editing the parameterization
Starting with a harmonic/LSCM parameterization, use the results of the previous steps to replace one of the `U`, `V` functions with a function obtained from a smooth user-guided vector field. Visualize the resulting `U` or `V` replacement function and its gradient atop the mesh, and texture the mesh with the new parameterization.

![](img/paramedit.png?raw=true)
An initial harmonic parameterization (1) is edited by designing a vector field. The user-provided constraints (2) are first interpolated (3), then a scalar function is reconstructed and used to replace the parameterization's 'V' coordinate function. The new scalar function and its gradient are shown in (4) together with the newly textured mesh.


### Detecting problems with the parameterization
It is possible for a parameterization created in this way to cause triangles to flip over as they are mapped into the `UV` plane. Determine a reliable criterion for detecting flipped triangles and visualize the planar mapped mesh with the flipped faces highlighted in red.

Required output for this section:

 * Visualization of the edited parameterization.
 * Visualization of flipped elements.
 * An ASCII dump of the flipped triangle indices (if any) resulting from an edited harmonic parameterization of the mesh `irr4-cyl2.off`, where the parameterization's `V` coordinate is replaced with a scalar field designed from the gradient vector constraints provided in `irr4-cyl2.constraints`.


