#   Transport and Flux on a Grid

- Write inter-element flux equations.
- Enumerate reasons for drift in conserved state variables.
- Diagnose transport issues for a particular mesh–element scheme.

##  

To monitor this situation, we frequently employ global balance equations, which watch for drift in the sum of state variables.


Euler's equations of gas dynamics:

.. math::

    \partial_t \mathbf{Q} = -\nabla\cdot{\mathbf{F}} +
    (\mathbf{F}\cdot\hat{n})_{\partial\Omega} + \mathbf{S}

where:

-   state $\mathbf{Q} = [\rho, \rho{E}, \rho\vec{V} ]$
-   flux $\mathbf{F} = [\rho\vec{V},(\rho{E} + p)\vec{V},
    (\rho(\vec{V}\otimes\vec{V}) + p*\mathbf{I})]$,
-   domain boundary $\partial\Omega$,
-   sources $\mathbf{S} = [{(\partial_t{\rho})}_s,
    {(\partial_t{\rho{E}})}_s, {(\partial_t{\rho\vec{V}})}_s]$


TODO mirgecom/euler.py:_facial_flux

##  Surface Flux

Finally, we apply the Gaussian divergence theorem to replace the volume integrals over the control volume by surface integrals over the control surface:

$$
\oint_{\partial V_C} d\vec{S} \cdot \nabla \cdot \left( \rho \vec{v} \varphi \right)
= \oint_{\partial V_C} d\vec{S} \cdot \left( \Gamma ^{\varphi} \nabla \varphi \right)
+ \int_{V_C} dV\, Q ^{\varphi} \text{.}
$$

This final form represents the conservative form of the PDE, and FVM focuses on efficiently solving this statement.  (This is what we mean by cell-centered FVM; LongND summarizes alternative formulations.)  There are two types of quantities represented:  surface flux terms and a volume source term.

<img src="./img/fvm-cell-terms.png" width="80%"/>

**Figure**.  Partial differential equation terms in a finite volume cell.

### Conservative Surface Flux

The key challenge in solving the semi-discretized equation arises from the flux integration requirement over the element faces.  For simplicity, we write the convective, diffusive, and total flux terms as

$$
\begin{eqnarray}
\vec{J}^{\varphi,C} & = \rho \vec{v} \varphi \\
\vec{J}^{\varphi,D} & = - \Gamma^{\varphi} \nabla \varphi \\
\vec{J}^{\varphi}   & = \vec{J}^{\varphi,C} + \vec{J}^{\varphi,D} \text{.}
\end{eqnarray}
$$

Replace the surface integral over C by a summation of flux terms at each face:

$$
\oint_{\partial V_C} d\vec{S} \cdot \vec{J}^{\varphi}
= \sum_{\text{faces}(V_C)} \left( \int_{\text{faces}} \vec{J}^{\varphi}_{\text{face}} \right)
$$

*This result conserves quantities extremely well, the primary advantage of FVM.*

#### Gaussian Quadrature

A conventional means of solution for the surface flux exploits the technique of Gaussian quadrature.  *Gaussian quadrature* integrates across the domain by evaluating the integrand at selected points and weighting the values by certain rules.  For instance, to evaluate

$$
\int_{-1}^{+1} dx \, e^{-x}
$$

with a 2-point quadrature, one calculates as follows:

| $x_i$         | $e^{-x}$ | $w_i$ | $w_i e^{-x}$ |
|---------------|----------|-------|--------------|
| $-\sqrt{1/3}$ | $0.5614$ | 1     | $0.5614$     |
| $+\sqrt{1/3}$ | $1.7813$ | 1     | $1.7813$     |
|               |          |       | $\sum_i w_i e^{-x} = 2.3427$ |

The absolute error is $2.3504 - 2.3427 = 0.0077$, and the method is far more efficient for most practical integrals than other rules of integration.

A very similar technique holds for surface integration of the flux.


<img src="./img/fvm-cell-quadrature-face.png" width="80%"/>

**Figure**.  Surface flux integration with one, two, and three quadrature points per cell face.



##  Error Sources

When we calculate a quantity numerically, we typically introduce some sort of numerical error into the result.  Frequently this is the result of either _roundoff error_, which occurs due to the finite nature of floating-point mathematics; or _truncation error_, which occurs from terms in the Taylor series of our calculation which are ignored.

### Roundoff Error

If we have an incomplete understanding of how numbers are represented on the machine, we may be surprised by certain results.  For instance, a trivial example suffices to show that something is going on:

```py
(1.1 - 0.8) == 0.3
```

To fully understand numerical error, we must first make a foray into numerical representation.  Recall that computer values like `int`s are stored as binary numbers in the machine.  If we wish to represent a fractional part, we can assign an arbitrary "binary point" (in analogy with the decimal point).

![](repo:./img/binary-point.png)

This sort of approach has major consequences.  For instance, consider the commands:

```py
print( 1.1 - 0.8 )
print( 0.3 )
print( 1.1 - 0.8 == 0.3 )
```

What happened?  If we examine the representation of each term, we can see the source of the discrepancy:

| Decimal Number | Binary Number |
| -------------- | ------------- |
| $1.1$ | `0001100110011001100110011001100110011001100110011010` |
| $0.8$ | `1001100110011001100110011001100110011001100110011010` |
| $1.1-0.8$ | `0011001100110011001100110011001100110011001100110100` |
| $0.3$ | `0011001100110011001100110011001100110011001100110011` |

So the difference of these last two quantities, expressed in binary, is `0000000000000000000000000000000000000000000000000001`!  But this is enough to prevent equality.

The right answer is to use a range, or the library functions `np.isclose` and `np.allclose` when assessing floating-point values.  _Never_ test equality on floating-point numbers.

```py
np.isclose( a, b, rtol=1e-05, atol=1e-08)
np.allclose(a, b, rtol=1e-05, atol=1e-08)
```

This is how floating-point values are _actually_ represented in the machine.  It's rather complicated, and mostly handled by the hardware.

![](repo:./img/floating-point.png)

-   D. Goldberg, [What every computer scientist should know about floating-point arithmetic](http://perso.ens-lyon.fr/jean-michel.muller/goldberg.pdf)

### Truncation Error

When we produce a finite-difference approximation, we are in essence taking a Taylor series expansion about a particular point and chopping off the higher-order terms in $h$:

$$

$$

This means that the expression carries a certain amount of error, sometimes left implicit but always present:

$$

$$

Truncation error means that numerical calculations produce pseudo-physical results, or behaviors that appear to mimic physical phenomena but are not in fact real.  We call the most important of these _numerical dispersion_ and _numerical diffusion_.

Physical dispersion arises under circumstances where waves of a fluid do not travel at the same speed, so they separate gradually.  (This can happen due to frequency or amplitude differences.)  In some media, we expect this of waves, but numerical dispersion mimics this phenomenon incorrectly.

http://www.mathematik.uni-dortmund.de/~kuzmin/cfdintro/lecture10.pdf

Physical diffusion


The most common way to think about truncation error is that we are inexactly solving an exact expression.  It's also possible to flip the statement:  we are exactly solving an inexact expression.  Sometimes this latter approach is fruitful in reasoning about how and whether to worry about numerical error sources in a calculation.

---
From the numerical point of view, numerical diffusion and dispersion reflect on the properties of the spatial discretisation employed:

- numerical diffusion indicates that the space discretisation operator will tend to smooth out sharp front/discontinuities, i.e. instead of having a sharp interface over 1 cell the space discretisation operator will spread it over a few cells;

- numerical dispersion refers to the properties of the space discretisation operator in not generating too high gradients, i.e. if you have a scalar between 0 and 1 with a sharp interface, the space discretisation operator will leads to value below 0 or exceeding 1.

From a practical point of view:
- an upwind discretisation scheme will have high numerical diffusion and low dispersion;
- a central or high order discretisation scheme (with no limiter) will have low numerical diffusion and high dispersion;
- a limited discretisation scheme tries to have the best of both world.

From mathematics, diffusion arises from highest term in truncation error is a factor of even order difference (\frac{\partial ^2}{\partial x^2} و \frac{\partial ^4}{\partial x^4} , ...) , however in numerical dispersion that is in odd order of difference (\frac{\partial ^3}{\partial x^3} و \frac{\partial ^5}{\partial x^5} , ...).


Based on h.Jasak (1999), high resolution NVD differencing scheme for arbitrarily unstructured mesh paper, we would have a numerical diffusion if:
1.the highest of the truncation error includes ODD-ORDER spatial derivatives, the solution will be affected by a certain amount of numerical diffusion
2. If on the other hand, the leading truncation term include EVEN-ORDER spatial derivatives numerical dispersion occurs.
[I really think this is backwards!]

https://www.cfd-online.com/Forums/main/84744-what-difference-between-diffusion-dispersion.html

in mathematical view, numerical diffusion is created when the highest term in truncation error is a factor of even order difference (\frac{\partial ^2}{\partial x^2} و \frac{\partial ^4}{\partial x^4} , ...) , however in numerical dispersion that is in odd order of difference (\frac{\partial ^3}{\partial x^3} و \frac{\partial ^5}{\partial x^5} , ...).
hadian is offline  	Reply With Quote

Old   February 10, 2011, 20:11
Default
  #7
dut_thinker
New Member

Join Date: Feb 2010
Posts: 12
Rep Power: 13
dut_thinker is on a distinguished road
Thanks for your reply, hadian.
Maybe it could be understood in this way. Just as you said, while the trunction error is even order difference, like diffusion equation it has diffusion characteristic; while the trunciton error is odd order difference, like advection equaiton, it will be propagated like wave.

Dear Hadian,
I guess you made a mistake in describing of EVEN-ODD ORDERS
Based on h.Jasak (1999), high resolution NVD differencing scheme for arbitrarily unstructured mesh paper, we would have a numerical diffusion if:
1.the highest of the truncation error includes ODD-ORDER spatial derivatives, the solution will be affected by a certain amount of numerical diffusion
2. If on the other hand, the leading truncation term include EVEN-ORDER spatial derivatives numerical dispersion occurs.

---

##  Boundary Conditions

`boundary.py`
`diffusion.py`
