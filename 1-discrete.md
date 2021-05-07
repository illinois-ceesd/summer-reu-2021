#   Discretization

Most numerical simulations require the modeled system to be broken into discrete components along a grid so that individual points and fields can be calculated and interpolated to produce an overall system state.  Let's look at how MIRGE-Com handles the problem discretization and represents the grid.


##  Nodal Galerkin Methods

MIRGE-Com implements a nodal discrete Galerkin finite element method.  This blends aspects of nodal methods (using Lagrange polynomials on each element), finite element methods (TODO), and finite volume methods (weak forms of the conservative form of the governing equations).  Since you probably haven't done much numerical work yet, let's break down what these things mean.

**Nodal Methods**.
**Finite Element Methods**.
**Finite Volume Methods**.
**Putting It All Together**.

"nodal FEM means you’re using a nodal basis (like lagrange polynomials on each element versus Legendre which would be hierarchical and don’t need nodes).  The DG part is basically finite volume but in “weak form” (wrapping integrals around the conservation law)."

- [Hesthaven & Warburton, _Nodal Discontinuous Galerkin Methods:  Algorithms, Analysis, and Applications_](https://link.springer.com/book/10.1007%2F978-0-387-72067-8)

##  Geometry

(lots of good stuff from ME498CF1)

MIRGE-Com specifies problem geometry using the `meshmode.mesh.generation` module.  There are a number of 2D and 3D volumetric primitives such as `generate_regular_rect_mesh` and `make_curve_mesh` which can be used to compose a mesh.

For instance, a basic isolator shape can be implemented using this setup:

```py

```

##  Structure

##  Special Requirements

border discontinuities
shock waves

<blockquote>
The strategies used to solve coupled sets of physics equations can be generally categorized as loose coupling and tight coupling. In loose coupling, the individual physics in a coupled problem are solved individually, keeping the solutions for the other physics fixed. After a solution is obtained for an individual physics, it is transferred to other physics that depend on it, and solutions are obtained for those physics.

These fixed-point iterations are repeated until convergence is obtained. If there is not a strong two-way feedback between the physics involved, convergence can be obtained quickly with a minimal number of loose-coupling iterations. An advantage of this approach is that it allows for independent codes to be coupled with relatively minor modifications to those codes, and they can each use their own solution strategies that are tailored for their solution domain. The disadvantage of loose coupling is that if  there is strong two-way feedback between the physics, that approach can have an unacceptably slow convergence rate and is more likely to encounter convergence difficulty.

In tight coupling solution methods, a single system of equations is assembled and solved for the full set of coupled physics. The nonlinear iterations operate on the full system of equations simultaneously, taking into account the interactions between the equations for the coupled physics in each iteration. In cases where there is strong coupling between the physics, this approach can have faster convergence rates than loose coupling. The primary disadvantage of this approach is that it necessitates tighter coordination between the codes to solve the individual physics.

https://inldigitallibrary.inl.gov/sti/5842302.pdf
</blockquote>
