# Assignment 2: Implicit Surface Reconstruction

In this exercise you will:

 * Compute an implicit MLS function approximating a 3D point cloud with given (but possibly unnormalized) normals.
 * Sample the implicit function on a 3D volumetric grid.
 * Apply the marching tets algorithm to extract a triangle mesh of this zero level set.
 * Experiment with various MLS reconstruction parameters.

Your main task is to construct an implicit function `f(x)` defined on all `x`  R<sup>3</sup> whose zero level set contains (or at least passes near) each input point. That is, for every point `pi` in the point cloud, we want `f(pi) = 0`. Furthermore, `∂f` (the isosurface normal) evaluated at each point cloud location should approximate the point's normal provided as input.

You will construct `f` by interpolating a set of target values, `di`, at "constraint locations", `ci`.
The MLS interpolant is defined by minimization of the form<br/>
![](https://latex.codecogs.com/svg.latex?f(x)=\mathrm{argmin}_\phi\sum_{i}w(ci,&space;x)(\phi(ci)-di)^2,)<br/>
<!-- $f(\x) = \argmin_\phi \sum_{i} w(\c_i, \x) (\phi(\c_i) -
d_i)^2,$ -->
where `phi(x)` lies in the space of admissible function (e.g.,
multivariate polynomials up to some degree) and `w` is a weight function that prioritizes each constraint equation depending on the evaluation point, `x`.

*Note:* The datasets provided actually already include triangles. Ignore them for this assignment.

## Setting up the Constraints

Your first step is to build the set of constraint equations by choosing constraint locations and values. Naturally, each point `pi` in the input point cloud should contribute a constraint with target value `f(pi) = di = 0`. But these constraints alone provide no information to distinguish the object's inside (where we want `f < 0`) from its outside (where we want `f > 0`). Even worse, the minimization is likely to find the trivial solution `f = 0` (if it lies in the space of admissible functions). To address these problems, we introduce additional constraints incorporating information from the normals as follows:

  * For each point `pi` in the point cloud, add a constraint of the form `f(pi) = di = 0`.
  * Fix an `eps` value, for instance `eps = 0.01 bounding_box_diagonal`.
  * For each point `pi` compute `pi+ = pi + eps ni`, where `ni` is the normalized normal of `pi`. Check that `pi` is the closest point to `pi+` if not, halve `esp` and recompute `pi+` until this is the case.
    Then, add another constraint equation: `f(pi+) = eps`.
  * Repeat the same process for `-eps`, i.e., add equations of the form `f(pi-) = -eps`. Do not forget to check each time that `pi` is the closest point to `pi-`.
  * Append the tree vectors `pi`, `pi+`, and `pi-` and corresponding `f` to a unique vector `p` and `f`.

After these steps, you should have `3n` equations for the implicit function `f(x)`.

**Important**: explicitty write a function `find_closed_point(point, points)` that retreives the index of the colosest point to `point` in `points`.

![](img/cat.png?raw=true)
Input point cloud for the *cat* mesh and inward/outward value constraints. The green, red and blue labels correspond to inside, outside and on the surface respectively. The blue points in the left figure are the same as the black ones in right figure.

*Relevant `numpy` functions:* `argmin`, `linalg.norm`.

Required output of this section:

 * Plot of the provided point cloud shaded with green, blue, and red dots.



## Use MLS interpolation to extend to function `f`

### Create a grid sampling the 3D space
Create a regular volumetric grid around your point cloud: compute the axis-aligned bounding box of the point cloud, enlarge it slightly, and divide it into uniform cells (tets). The grid resolution is configured by
the global variable `resolution`, which can be changed. *Note* the funciton to generate the grid is provided. We call the grid vertices `x` and the tets connecting them `T`.

### MLS Interpolation
We now use MLS interpolation to construct an implicit function satisfying the constraints as nearly as possible.
We won't define the function with an explicit formula; instead we characterize it as the linear combination of polynomial basis functions that best satisfies the constraints in some sense. At a given point `xi` in `x`, you evaluate this function by finding the "optimal" basis function coefficients (which will vary from point to point!) and using these to combine the basis function values at `xi`.

Complete the appropriate source code sections to evaluate the MLS function at every node `xi` of a regular volumetric
grid containing the input point cloud. As an example, the provided code computes the grid values for an implicit function representing a sphere (MLS wasn't used in
this case since the formula is known analytically).


More specifically, for each grid node `xi` of the grid, evaluate the implicit function `f(xi)`, whose zero level set approximates the point cloud.
Use the moving least squares approximation presented in class and in the tutoring session. You should use the
Wendland weight function with radius configured by `wendlandRadius` and degree `k = 0, 1, 2` polynomial basis functions configured by `polyDegree`. Only use the constraint points with nonzero weight (i.e., points `p` with `||xi - p|| < wendlandRadius`). *Note* if the number of constraint points within `wendlandRadius` is less than twice the number of polynomial coefficients (i.e., 1 for `k = 0`, 4 for `k = 1`, and 10 for `k = 2`), you can assign a large positive (outside) value to the grid point.

Store the field value  `fx = f(xi)` in a  `numpy.array`, using the same ordering as in `x`. Render these values by coloring each grid point red/green depending if they are inside outside (i.e., `fx < 0` or `fx ≥ 0`). You can use the `meshplot.plot(..., c=color)` where `color` is a `n x 3` matrix containing rgb values.

**Important**: explicitty write a function `closest_points(point, points, h)` that retreives the indices all points in `points` that are at distance less than `h` from `point`.

*Relevant `numpy` functions:* `argwhere`, `linalg.solve`.

Required output of this section:

 * Plot of the grid points `x` colored according of being inside or outside the input cloud.



## Implementing a spatial index to accelerate neighbor calculations

To construct the MLS equations, you will perform queries
to find, for a query point `q`:

    * the closest input point to `q` (needed while constructing inside/outside offset points); and
    * all input points within distance `h` of `q` (needed to select constraints with nonzero weight).


Although a simple loop over all points could answer these queries, it would be slow for large point clouds.
Improve the efficiency by implementing a simple spatial index (a uniform grid at some resolution). By this, we mean binning vertices into their enclosing grid cells and restricting the neighbor queries to visit only the grid cells that could possibly satisfy the query. You can debug
this data structure by ensuring that it agrees with the brute-force for loop implementation.

This part requires changing the two fucntion `find_closed_point` and `closest_points`.


## Using a non-axis-aligned grid.
The point cloud `luigi.off` is not aligned with the canonical axes.
Running reconstruction on an axis-aligned grid is wasteful in this case: many of the grid points will lie far outside the object. Devise an automatic (and general) way to align the grid to the data and implement it.

Required output of this section:

* Plot of the grid with nodes colored according to their implicit function values.


## Extracting the surface
You can now use marching tets to extract the zero isosurface from your grid.
The extraction has already been implemented and the surface is displayed. The implicit function obtained from MLS might be noisy and the reconstructed mesh will contain several pieces. Filter out and keep the largest component.

![](img/cat-recon.png?raw=true)

*Relevant `igl` functions:* `marching_tets`, `face_components`.


Required output of this section:

* Plot of the reconstructed surfaces. Experiment with different parameter settings: grid resolution (also anisotropic in the 3 axes), Wendland function radius, and polynomial degree.

## Optional tasks

<!-- * *(2 points)* Compute the closed-form gradient of the MLS approximation. Suggestion: A good strategy to solve this exercise is to write MLS explicitly in matrix form and then compute its gradient (a good reference for differentiating expressions with matrices can be found in [The Matrix Cookbook](http://orion.uwaterloo.ca/~hwolkowi/matrixcookbook.pdf).
 -->
* *(2 points)* In [Interpolating and Approximating Implicit Surfaces from Polygon Soup](http://graphics.berkeley.edu/papers/Shen-IAI-2004-08/index.html) normals are used differently to define the implicit surface. Instead of generating new sample points offset in the positive and negative normal directions, the paper uses the normal to define a linear function for each point cloud point: the signed distance to the tangent plane at the point.
Then the values of these linear functions are interpolated by MLS. Implement Section 3.3 of the paper and append to your report a description of the method and how it compares to the original point-value-based approach.
Estimate a normal for results obtained with single dataset.

* *(1 points)* [Screened Poisson Surface Reconstruction](http://www.cs.jhu.edu/~misha/MyPapers/ToG13.pdf) is a more modern technique that avoids some of the pitfalls of local reconstruction methods. An implementation is provided in [MeshLab](http://www.meshlab.net). A standalone implementation of this method is also provided by the authors [here](http://www.cs.jhu.edu/~misha/Code/PoissonRecon/Version9.01/) with accompanying usage instructions and datasets. Compare your MLS        reconstruction results to the surfaces obtained with this method, and try to understand the differences. Report your findings.
