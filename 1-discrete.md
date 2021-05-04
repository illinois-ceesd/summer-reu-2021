#   Discretization

Most numerical simulations require the modeled system to be broken into discrete components along a grid so that individual points and fields can be calculated and interpolated to produce an overall system state.  Let's look at how MirgeCom handles the problem discretization and represents the grid.


##  Nodal Galerkin Methods

MirgeCom implements a nodal discrete Galerkin finite element method.  This blends aspects of nodal methods (using Lagrange polynomials on each element), finite element methods (TODO), and finite volume methods (weak forms of the conservative form of the governing equations).  Since you probably haven't done much numerical work yet, let's break down what these things mean.

**Nodal Methods**.
**Finite Element Methods**.
**Finite Volume Methods**.
**Putting It All Together**.

"nodal FEM means you’re using a nodal basis (like lagrange polynomials on each element versus Legendre which would be hierarchical and don’t need nodes).  The DG part is basically finite volume but in “weak form” (wrapping integrals around the conservation law)."

- [Hesthaven & Warburton, _Nodal Discontinuous Galerkin Methods:  Algorithms, Analysis, and Applications_](https://link.springer.com/book/10.1007%2F978-0-387-72067-8)

##  Geometry

(lots of good stuff from ME498CF1)

##  Structure

##  Special Requirements

border discontinuities
shock waves
