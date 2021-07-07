#   Differential Equation Solvers

- Distinguish categories of differential equation solvers
- Write the strong and weak forms of governing equations in the scramjet system

MIRGE-Com has “higher” goals specifiable only in human language—the simulation of scramjets—but in its internals, it is a differential equation solver, one more in a long and illustrious line.

We will use the Navier-Stokes equation for $x$-momentum as the basis of our discussion:

$$
\begin{align}
  \rho \left(\frac{\partial u_x}{\partial t} + u_x \frac{\partial u_x}{\partial x} + u_y \frac{\partial u_x}{\partial y} + u_z \frac{\partial u_x}{\partial z}\right)
    =& -\frac{\partial p}{\partial x} + \mu \left(\frac{\partial^2 u_x}{\partial x^2} + \frac{\partial^2 u_x}{\partial y^2} + \frac{\partial^2 u_x}{\partial z^2}\right) \\
    & - \mu \frac{\partial}{\partial x} \left( \frac{\partial u_x}{\partial x} + \frac{\partial u_y}{\partial y} + \frac{\partial u_z}{\partial z} \right) + \rho g_x
\end{align}
$$

In three dimensions, the conserved quantity $p = \rho u_x$ is a function of several independent variables:  $p(x, y, z, t)$.  This is a partial differential equation in four independent variables, possibly with varying viscosity $\mu$ and body forces $\vec{g}$.  As written, temperature does not vary these quantities but may be another independent variable of consequence.

The solution of ordinary differential equations is the most fundamental tool:  numerical methods for partial differential equations are typically rooted in understanding ODE techniques well.  Furthermore, of the three types of ODEs (initial value problems, boundary value problems, and eigenvalue problems), IVPs form the basis for understanding other methods.

Our archetypal IVP for this tutorial is a relationship between the change in a quantity in time and some expression of $y$ and $t$.  (More generally, $f$ can also be a function of $y'$, but this tends to be quite prodigious to solve; we call these _implicit_ ODEs.)

$$
\frac{dy}{dt}
=
f(y, t)
$$

Moin:  “The aim of all numerical methods for solution of this differential equation is to obtain the solution at time $t_{n+1} = t_{n} + \Delta t$, given the solution for $0 \leq t \leq t_{n}$.”

A direct method approximates the derivatives such as $dy/dt$ using a finite difference approximation over a discretized grid.  This is exemplified by the Runge-Kutta fourth-order solution method employed in MIRGE-Com today.

We typically end up with a system of coupled differential equations when we have either multiple derivatives or multiple points in the grid present.  (Which is almost always the case.)

$$
\begin{pmatrix}
y_1^{(n)} \\
y_2^{(n)} \\
\vdots \\
y_m^{(n)}
\end{pmatrix}
=
\begin{pmatrix}
f_1 \left (x,\mathbf{y},\mathbf{y}',\mathbf{y}'',\ldots, \mathbf{y}^{(n-1)} \right ) \\
f_2 \left (x,\mathbf{y},\mathbf{y}',\mathbf{y}'',\ldots, \mathbf{y}^{(n-1)} \right ) \\
\vdots \\
f_m \left (x,\mathbf{y},\mathbf{y}',\mathbf{y}'',\ldots, \mathbf{y}^{(n-1)} \right)
\end{pmatrix}
$$

We have three basic families of solution approaches available:

1. Explicit matrix solution
2. Implicit matrix solution
3. Direct stepping methods

##  Explicit

If the system of coupled differential equations can be written separately for each time step, and a few mathematical desiderata are satisfied, then the system of equations can be written as a matrix.  Formally,

formal
$$
\underline{\underline{A}} \vec{y} = \vec{f}
$$

where $\underline{\underline{A}}$ is the matrix of dependencies between points on the grid and $\vec{f}$ is the (known) solution vector

$$
\begin{pmatrix}
f_1 \\
f_2 \\
\vdots \\
f_m
\end{pmatrix}
$$

$\vec{y}$ is, of course, the vector of unknowns to be solved for.

The particular form of $\underline{\underline{A}}$ is entirely determined by the finite difference approximation employed.  For instance, for a second-order central-difference equation

$$
\frac{y_{j+1} - 2 y_{j} + y_{j-1}}{h^2} + a_j \frac{y_{j+1} - y_{j-1}}{2h} _ b_j y_j = f_j
\text{,}
$$

rearranged as

$$
\left( \frac{1}{h^2} + {a_j}{2h} \right) y_{j+1} +
\left( b_j - \frac{2}{h^2} \right) y_{j} +
\left( \frac{1}{h^2} - \frac{a_j}{2h} \right) y_{j-1} +
=
f_j
$$

the resulting matrix becomes

$$
\underline{\underline{A}}
=
\begin{pmatrix}
\left( b_1 - \frac{2}{h^2} \right) & \left( \frac{1}{h^2} + {a_1}{2h} \right) & 0 & \hdots \\
\left( \frac{1}{h^2} - \frac{a_2}{2h} \right) & \left( b_2 - \frac{2}{h^2} \right) & \left( \frac{1}{h^2} + {a_2}{2h} \right) & \hdots \\
0 & \left( \frac{1}{h^2} - \frac{a_3}{2h} \right) & \left( b_3 - \frac{2}{h^2} \right) & \hdots \\
\vdots &  & \ddots & \\
\end{pmatrix}
$$

Note the structure of this matrix.  What features do you observe?

Because finite difference schemes are local, depending only on grid-adjacent mesh points, explicit matrices tend to be diagonal and sparse.  (This requires constructing the _right_ grid in the first place, too.)

Sparse matrices contain mainly zeroes, and only the occupied points are stored in memory for efficiency.

So why don't we solve all matrices explicitly?  Math.

_Representation_

Formally, given the matrix equation

$$
\underline{\underline{A}} \vec{y} = \vec{f}
$$

the solution should be obtained as

$$
\underline{\underline{A}}^{-1} \underline{\underline{A}} \vec{y}
=
\underline{\underline{A}}^{-1} \vec{b} \rightarrow \vec{y}
\rightarrow
\underline{\underline{I}} \vec{y}
=
\underline{\underline{A}}^{-1} \vec{f}
\rightarrow
\vec{y}
=
\underline{\underline{A}}^{-1} \vec{f}
$$

This is correct, but is quite often computationally inefficient to achieve.  For instance, the matrix $\underline{\underline{A}}$ is typically sparse (mostly zeroes), but the inverse matrix $\underline{\underline{A}}^{-1}$ is dense (requiring vastly more storage for large matrices describing systems with many degrees of freedom).

Enter LU and QR factorization.  A number of ways of algebraically modifying both sides of the equation have been devised.

_Existence_

Given a matrix equation, does it _have_ a solution?  It's not guaranteed to.  The numerical methods that we prefer, such as finite difference schemes, have been chosen to yield matrices that are not singular (meaning that there is no inverse matrix).

_Stiffness_

The differential equations themselves may be _stiff_, meaning that they resolve on different scales in the independent variables.  That is, the solution being sought may vary slowly while adjacent solutions vary quickly.

- [Moler, “Stiff Differential Equations”](https://www.mathworks.com/company/newsletters/articles/stiff-differential-equations.html)

_Conditioning_

Another computational challenge relates to the representation of the problem as a matrix as well.  The [matrix condition number](https://en.wikipedia.org/wiki/Condition_number) refers to how well-conditioned (or ill-conditioned) a problem formulation is to be solved:

> The condition number associated with [a] linear equation gives a bound on how inaccurate the solution x will be after approximation.  (Note that this is before the effects of round-off error are taken into account; conditioning is a property of the matrix, not the algorithm or floating-point accuracy of the computer used to solve the corresponding system.)  In particular, one should think of the condition number as being (very roughly) the rate at which the solution $y$ will change with respect to a change in $b$. Thus, if the condition number is large, even a small error in $b$ may cause a large error in $y$. On the other hand, if the condition number is small, then the error in $y$ will not be much bigger than the error in $b$.  ¶The condition number is defined more precisely to be the maximum ratio of the relative error in $y$ to the relative error in $b$.

Finally, not all DE solution methods can be written in a matrix form, or there may be other reasons to avoid an explicit solution, such as efficiency gains in working with already-solved (same-time-step) answers.

##  Implicit

If we are not interested or able to solve the explicit matrix formulation, we may adopt an implicit iterative approach.  In this case, we look for an approximate solution at a particular accuracy.  Typically, we take a candidate solution, re-evaluate the outcome, check for the change (as error), and iterate on this process until the global error is small enough.  We anticipate that increasing the number of iterations will sooner or later solve the problem (converging solution).

$$
\underline{\underline{A}} \vec{y} = \vec{f}
\rightarrow
\left(\underline{\underline{A_{1}}} - \underline{\underline{A_{2}}}\right) \vec{y} = \vec{f}
\rightarrow
\underline{\underline{A_{1}}} \vec{y} = \underline{\underline{A_{2}}} \vec{y} + \vec{f}
$$

We can impose a number of requirements on $\underline{\underline{A_{1}}}$, such as that it be invertible, that the solution converges, etc.

We define error as

$$
\vec{\vareps}^{(k)} = \vec{y}^{(k)} - \vec{y}^{(k-1)}
$$

and convergence as

$$
\lim_{k\rightarrow \infty} \vec{\vareps}^{(k)}
=
\vec{0}
\text{.}
$$

(Look up _spectral radius_ for more on this topic.)

##  Direct Methods

MIRGE-Com currently solves by explicitly time-stepping forward, which is basically a piecemeal explicit solution.  Steady-state flow would be detected similar to how iterative convergence for an implicit method was just described, except that $k$ would refer to timestep not iteration.

A simple approximation to the IVP may be had by selecting a finite difference (rather than an infinitesimal) and then rearranging the result for an explicit statement for the next value.

$$
\frac{dy}{dt}
=
f(y, t)
\approx
\frac{y_{n+1} - y_{n}}{\Delta t}
$$

∴

$$
y_{n+1} = \Delta t \left( f(y, t) + y_{n} \right)
$$

This approximation is only stable for extremely small $\Delta t$.

$$
y_{n+1} = y_n + \frac{1}{6}h\left(k_1 + 2k_2 + 2k_3 + k_4 \right),\\
$$

where

$$
\begin{align}
 k_1 &= \ f(y_n, t_n), \\
 k_2 &= \ f\left(y_n + \Delta t\frac{k_1}{2}, t_n + \frac{\Delta t}{2}\right), \\
 k_3 &= \ f\left(y_n + \Delta t\frac{k_2}{2}, t_n + \frac{\Delta t}{2}\right), \\
 k_4 &= \ f\left(y_n + \Delta t k_3, t_n + \Delta t \right).
\end{align}
$$

(There are many other solvers.  RK4 is the workhorse of time-stepping solvers but it is not suitable for stiff equations.)

`leap` provides a library of implicit and explicit time-stepping methods in this vein.

- Examine `steppers.py:advance_state`.
- Examine `integrators.py:rk4_step`.

##  Governing Equations

Now that you've seen a lot more of the numerical machinery underlying MIRGE-Com, let's revisit the physical expressions introduced in the first tutorial and find a suitable discretization in space as well as in time.

---

We begin with the _strong form_ $\mathbb{S}$ of a differential equation in space.  This is the canonical formulation of an ODE, including boundary conditions.  As an example, let us use the following 1D expression:

$$
\frac{\partial^2 y}{\partial x^2}
=
x + 1
$$

with $y(x=0) = y(x=1) = 0$ (which can be achieved with a linear transformation).

---

By integrating the equations over a volume (thus, a control volume or element), we write the conservation law over the volume rather than over each point.  We refer to this formulation as the _weak form_ $\mathbb{W}$.  We employ a weighting function $w(x)$ which satisfies the same boundary conditions as $y(x)$.

$$
\int _{x=0}^{1} dx\, \frac{\partial^2 y}{\partial x^2} w(x)
=
\int _{x=0}^{1} dx\, (x+1) w(x)
$$

Integrate by parts to obtain a statement

$$
\int _{x=0}^{1} dx\, \frac{\partial^2 y}{\partial x^2} w(x)
=
\left. \frac{dy}{dx} w(x) \right|_{x=0}^{1} - \int _{x=0}^{1} dx\, \frac{dy}{dx} \frac{dw}{dx}
=
- \int _{x=0}^{1} dx\, \frac{dy}{dx} \frac{dw}{dx}
$$

Strong form and weak form are isomorphic.

---

The _discretized form_ or _Galerkin form_ $\mathbb{G}$ requires us to make an assumption about the form of $w(x)$.  Many possible basis functions have been suggested and used in practice; here we use piecewise linear statements to illustrate the solution.

Divide the domain into $n$ pieces and assign a piecewise linear tent function to each piece.

$$
W_{k}(x)=\begin{cases} {x-x_{k-1} \over x_k\,-x_{k-1}} & \mbox{ if } x \in [x_{k-1},x_k], \\
{x_{k+1}\,-x \over x_{k+1}\,-x_k} & \mbox{ if } x \in [x_k,x_{k+1}], \\
0 & \mbox{ else}\end{cases}
$$

for $k \in {1 ... n}$.

![](https://upload.wikimedia.org/wikipedia/commons/8/85/Finite_element_method_1D_illustration1.png)

Besides linear tent functions, one may use Chebyshev polynomials, Lagrange polynomials, Legendre polynomials, all manner of splines, and so forth.

![Chebyshev polynomials](https://upload.wikimedia.org/wikipedia/commons/thumb/5/5c/Chebyshev_Polynomials_of_the_First_Kind.svg/1920px-Chebyshev_Polynomials_of_the_First_Kind.svg.png)

![Lagrange polynomials](https://upload.wikimedia.org/wikipedia/commons/thumb/5/5a/Lagrange_polynomial.svg/1920px-Lagrange_polynomial.svg.png)

![Legendre polynomials](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c8/Legendrepolynomials6.svg/1920px-Legendrepolynomials6.svg.png)

(In many cases, this resulting $\mathbb{G}$ is then recast into a _matrix form_ $\mathbb{M}$.  Discretized form and matrix form are isomorphic.  The mapping from weak form to discretized form is where inaccuracy and approximation are introduced.)

In two or three dimensions, the volume is spanned by polynomials in each independent spatial variable.

- [“Finite Element Method”](https://en.wikipedia.org/wiki/Finite_element_method)

`grudge` discretizes and solves the discontinuous Galerkin operators in space.  From MIRGE-Com's perspective, this is a basically invisible process, and decisions about how to best handle a given formulation are left up to `grudge`.  `grudge` uses `pymbolic` to handle quantities in the grid.

The general solution process in MIRGE-Com utilizes both time-stepping with RK4 and space-resolved solution with `grudge`'s finite element formulation.
