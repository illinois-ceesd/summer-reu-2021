#   Differential Equation Solvers

* [Slides](./slides/4-solvers.pdf)

---

- Distinguish categories of differential equation solvers
- Write the strong and weak forms of governing equations in the scramjet system

MIRGE-Com has “higher” goals specifiable only in human language—the simulation of scramjets—but in its internals, it is a differential equation solver, one more in a long and illustrious line.

##  Explicit

TODO

formal
$$
\underline{\underline{A}} \vec{x} = \vec{b}
$$

sparse matrices

So why don't we solve all matrices explicitly?  Two reasons:  computational issues and numerical issues.

**Computational Issues**.  Formally, given the matrix equation

$$
\underline{\underline{A}} \vec{x} = \vec{b}
$$

the solution should be obtained as

$$
\underline{\underline{A}}^{-1} \underline{\underline{A}} \vec{x}
=
\underline{\underline{A}}^{-1} \vec{b} \rightarrow \vec{x}
\rightarrow
\underline{\underline{I}} \vec{x}
=
\underline{\underline{A}}^{-1} \vec{b}
\rightarrow
\vec{x}
=
\underline{\underline{A}}^{-1} \vec{b}
$$

This is correct, but is quite often computationally inefficient to achieve.  For instance, the matrix $\underline{\underline{A}}$ is typically sparse (mostly zeroes), but the inverse matrix $\underline{\underline{A}}^{-1}$ is dense (requiring vastly more storage for large matrices describing systems with many degrees of freedom).

LU/QR factorization

Matrix conditioning

TODO

**Numerical Issues**.  not all DE solution methods can be written in a matrix form

##  Implicit

TODO


##  Governing Equations

Now that you've seen a lot more of the numerical machinery underlying MIRGE-Com, let's revisit the physical expressions introduced in the first tutorial.

strong/weak/etc.
