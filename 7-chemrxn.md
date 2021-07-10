#   Chemical Reactivity in Flows

* [Slides](./slides/7-chemrxn.pdf)

---

-   Understand various models of chemical reactivity and combustion
-   Model a reacting flow


##  Combustion Chemistry

Anyone out of high school chemistry can write and understand the basic expression of burning hydrogen:

$$
2\text{H}_2 + \text{O}_2 → 2\text{H}_2\text{O}
$$

wherein fuel and oxidizer combine (in the presence of sufficient heat) to produce water.

This reaction requires heat, which is another way of saying that it has a characteristic _activation energy_, necessary for two molecules in near contact to actually react.

![](https://lightcat-files.s3.amazonaws.com/problem_images/d74d1e86ee432859-1592923071592.jpg)

Given sufficient energy, the likelihood that two molecules jostle each other with enough force to mutually cross the activation energy barrier and react increases exponentially in temperature.

What's missing from this straightforward picture is a panoply of intermediate stages and reactions that occur in a fraction of a second but drive the actual mechanics of fuel consumption and utilization.  The primary chemical equations involved in hydrogen combustion can be written as an array of chain initiation, propagation, and termination reactions:

$$
\begin{array}{rlr}
\text{H}_2 + \text{O}_2 & → \text{H}· + \text{HO}_2· & \text{initiation} \\ \\
\text{H}· + \text{O}_2 & → \text{O}· + \text{OH}· & \text{branching} \\
\text{O}· + \text{H}_2 & → \text{OH}· + \text{H}· & \text{branching} \\
\text{OH}· + \text{H}_2 & → \text{H}_2\text{O} + \text{H}· & \text{propagation}
\text{H}_2\text{O}_2 + \text{M} & → 2 \text{OH}· + \text{M} & \text{propagation} \\
\text{H}· + \text{HO}_2· & → 2 \text{OH}· & \text{propagation} \\
\text{H}· + \text{HO}_2· & → \text{H}_2 + \text{O}_2 & \text{termination} \\
\text{OH}· + \text{HO}_2· & → \text{H}_2\text{O} + \text{O}_2 & \text{termination} \\ \\
\text{H}· & → \text{wall} & \text{termination} \\
\text{H}· + \text{O}_2 + \text{M} & → \text{HO}_2· + \text{M} & \text{termination} \\
\text{HO}_2· + \text{HO}_2· + \text{M} & → \text{H}_2\text{O}_2 + \text{O}_2 + \text{M} & \text{termination} \\ \\
\end{array}
$$

where $\text{M}$ is any mediating species preserved by the reaction, such as $\text{O}_2$.

Once hydrocarbons become involved as fuel, the above combustion reactions appear simple in comparison; an overview of some net reactions should get your mind going:

$$
\begin{array}{rll}
\text{CH}_4 + 2\text{O}_2 & → \text{CO}_2 + 2\text{H}_2\text{O} & \text{methane} \\
2\text{C}_2\text{H}_6 + 7\text{O}_2 & → 4\text{CO}_2 + 6\text{H}_2\text{O} & \text{ethane} \\
2\text{C}_3\text{H}_8 + 7\text{O}_2 & → 6\text{CO}_2 + 8\text{H}_2\text{O} & \text{propane} \\
2\text{CH}_3\text{OH} + 3\text{O}_2 & → 2\text{CO}_2 + 4\text{H}_2\text{O} & \text{methanol}
\end{array}
$$

There are actually dozens of relations which can be written over a whole host of minor species such as radicals which arise in real combustion:  $\text{CHO}·$ and many others.  Some nitrogen compounds arise, but these are typically negligible.  To fully model even $\text{H}_2–\text{CO}$ combustion, about thirty reactions are necessary to consider.

Such a complex system consideration of both chemical equilibria (the energetic favorable states at a given temperature and pressure) and chemical kinetics (the rates of reaction given species concentrations and transport).

**Thermodynamic equilibrium**.  The [JANAF](https://janaf.nist.gov/) tables are customarily used to calculate chemical equilibria based on Gibbs free energy $\Delta G$.  This is a _relatively_ straightforward calculation, about junior-level in physical chemistry or chemical engineering.

TODO thermodynamics
$$
\sum_{\text{min}}
$$

**Chemical kinetics**.

TODO kinetics

- [Argonne National Laboratory, “The Complex Chemistry of Combustion”](https://www.anl.gov/article/the-complex-chemistry-of-combustion)
- [Davis et al., “An optimized kinetic model of H2/CO combustion”  _Proceedings of the Combustion Institute, 30_, pp. 1283–1292](http://ignis.usc.edu/Mechanisms/H2-CO/h2-co.pdf), esp. the table on p. 1284
- [Green, “Combustion & Fuels:  Chemistry & Kinetics”, _Combustion Summer School 2017_](https://cefrc.princeton.edu/sites/cefrc/files/combustion-summer-school/lecture-notes/2017_Green_All_combined.pdf)
- [Wang, “Combustion Chemistry”, _Combustion Summer School 2015_](https://cefrc.princeton.edu/sites/cefrc/files/Files/2015%20Lecture%20Notes/Wang/Lecture-3-Basic-Chemical-Kinetics.pdf)


##  Modeling Combustion

Reacting flows incorporate coupled equations with effective source terms and species balances; at minimum, an enthalpy term needs to be included in the transport equations.  Several approaches have been employed:

1. Species Transport and Finite-Rate Chemistry Approach
2. Mixture Fractions Approach
3. Reaction Progress Variable Approach
4. Composition Probability Density Function Transport Approach
5. Multiphase Species Transport
6. Aggregate Modeling (e.g., Internal Combustion Engine)

We are early in the story of MIRGE-Com at this point, so the chemical reaction modeling is relatively immature.  Let's consider what is involved with modeling each of these.

### Species Transport

This approach solves the conservation equations for convection, diffusion, and reaction sources for multiple component species.  The user may specify multiple chemical reactions to model simultaneously, with reactions occurring either in the bulk flow, at wall or particle surfaces, or in the porous region.

$$
\frac{\partial}{\partial t} \left( \rho Y_{i} \right) + \nabla \cdot \left( \rho \arrow{v} Y_{i} \right)
=
-\nabla \cdot \arrow{J}_{i} + R_{i} + S_{i}
$$

This conservation equation describes the convection and diffusion of the local mass fraction of a species $Y_{i}$.  $R_{i}$ is the rate of production by chemical reaction, and $S_{i}$ is the rate of creation by addition from the dispersed phase and user-defined sources.

The diffusion flux $\arrow{J}_{i}$ occurs due to gradients of concentration and temperature.  Fick's law is the default:

$$
\arrow{J}_{i}
=
-\rho D_{i,m} \nabla Y_{i} - D_{T,i} \frac{\nabla T}{T}
$$

where $D_{i,m}$ is the mass diffusion coefficient and $D_{T,i}$ is the thermal diffusion coefficient.  This approximation is generally good.

With turbulence, further accommodation is necessary because mixing must be explicitly included as a function of turbulence at small length scales.

$$
\arrow{J}_{i}
=
-\left( \rho D_{i,m} + \frac{\mu_t}{\text{Sc}_{t}} \right) \nabla Y_{i} - D_{T,i} \frac{\nabla T}{T}
$$

Naturally, multicomponent transport introduces a number of significant physical effects into the system, including diffusion, enthalpy transport, and temperature gradients.

The reaction rate $R_i$ may be specified as:

-   Laminar finite-rate model (Arrhenius kinetics)

-   Eddy-dissipation model (turbulence-controlled; cheap)

-   Eddy-dissipation-concept model (both; very expensive)

Pressure effects may be included as well, if significant.

Surface reactions may be diffusion-limited or kinetics-limited; in the latter case, surface coverage may also be significant.  One option is to track molar concentrations of wall-adsorbed species in the latter case.  Particle surface reactions can also be modeled (the char particle burning model).

Reaction stoichiometry can be expressed quite generally:

$$R_{\text{kin},r} = \frac{A_r T^\beta \exp\left( \frac{E_r}{RT} \right)}{p_{r,d}^{N_{r,n}}} \product_{n=1}^{n_\text{max}} p_n^{N_{r,n}}$$

as a function of partial pressures $p_n$ and reaction orders $N_{r,n}$.

### Mixture Fractions Approach (Non-Premixed Combustion)

In this approach, transport equations are written and solved for *mixture fractions* rather than for individual species.  Species concentrations are calculated from the resulting mixture fraction field.

The mixture fractions approach assumes that you are modeling a combusting system with discrete fuel and oxidizer inlets—so a diffuser, spray combustion chamber, or pulverized fuel flame.

The non-premixed combustion model rests on the assumption that the instantaneous thermochemical state of a fluid can be related to a conserved scalar quantity, the mixture fraction

$$
f
=
\frac{Z_i - Z_{i,\text{ox}}}{Z_{i,\text{fuel}} - Z_{i,\text{ox}}}
$$

Given what we know of likely combustion components, we may assume equal diffusivities for all species and write the density-averaged mixture fraction equation

$$
\frac{\partial}{\partial t} \left( \rho \bar{f} \right) + \nabla \cdot \left( \rho \arrow{v} \bar{f} \right)
=
-\nabla \cdot \left( \frac{\mu_\text{lam} + \mu_\text{turb}}{\sigma_t} \nabla \bar{f} \right) + S_{m}
$$

The source term $S_m$ arises from the transfer of mass into the gaseous phase from liquid fuel droplets or reacting particles.

This model then uses look-up tables to calculate thermal and chemical effects from the fluctuation in mixture fraction $f$.

### Reaction Progress Variable Approach (Premixed Combustion)

<blockquote>
In premixed combustion, fuel and oxidizer are mixed at the molecular level prior to ignition. Combustion occurs as a flame front propagating into the unburnt reactants. Examples of premixed combustion include aspirated internal combustion engines, lean-premixed gas turbine combustors, and gas-leak explosions.
</blockquote>

Premixed combustion occurs as a thin propagating flame along the (possibly turbulent) boundary.  In contract, non-premixed combustion is typically a mixing problem.  For premixed combustion, the major challenge is to track the turbulent flame speed as a function of the laminar flame speed and the turbulence at the interface.

Mathematically, this model ends up mixing together pieces of what we've already discussed.  The simplest version reduces to a reaction progress variable $c$ written in a conservation equation as

$$
\frac{\partial}{\partial t} \left( \rho \bar{c} \right) + \nabla \cdot \left( \rho \arrow{v} \bar{c} \right)
=
-\nabla \cdot \left( \frac{\mu_\text{turb}}{\text{Sc}_{t}} \nabla \bar{c} \right) + \rho S_{c}
$$

The progress variable $c$ is defined as a normalized sum of product species mass fractions,

$$
c = \frac{\sum_k \alpha_k \left( Y_k - Y_k^u \right)}{\sum_k \alpha_k Y_k^\text{eq} }
=
\frac{Y_c}{Y_c^\text{eq}}
$$

The mean reaction rate is

$$
\rho S_c
=
\rho_u U_t \left| \nabla c \right|
$$

We have many options available to us for tracking the turbulent flame speed.  For instance, the $G$-equation is a premixed flame-front tracking model.  The Zimont turbulent flame speed closure model, which accommodates wrinkled or thickened flame fronts, is often a good option.

> The model is based on the assumption of equilibrium small-scale turbulence inside the laminar flame, resulting in a turbulent flame speed expression that is purely in terms of the large-scale turbulent parameters.
>
> The model is strictly applicable when the smallest turbulent eddies in the flow (the Kolmogorov scales) are smaller than the flame thickness, and penetrate into the flame zone.
>
> The model is valid for premixed systems where the flame brush width increases in time, as occurs in most industrial combustors. Flames that propagate for a long period of time equilibrate to a constant flame width, which cannot be captured in this model.
>
> (_ANSYS Fluent Theory Guide_)

- [Premixed Flame Propagation in a Tunnel](https://youtu.be/CjGuHbsi3a8?t=371)

### Composition Probability Density Function Transport Approach (Finite-Rate Chemistry)

We can use a probability density function (PDF) to describe finite-rate chemical kinetics in turbulent reacting flows.  For instance, we can use this approach to simulate effects such as flame extinction and ignition including species such as CO and NO<sub>x</sub>.  The advantage of this model (although computationally complex) is that it accommodates turbulent flow more accurately than the previous methods in some cases.

With the full species-transport approach discussed previously, we frequently employ Reynolds time-averaged flow to model turbulence.  This leads to unknown terms for turbulent scalar flux and mean reaction rate, which are modeled as "enhanced diffusion" and with a chemistry model, respectively.  This last in particular is difficult and prone to error.

Another approach is to derive a transport equation for their single-point joint probability density function $P$.  $P$ represents the fraction of the time that the fluid spends at each species, temperature, and pressure state.  Only the accessible portions of parameter space need be modeled, and obviously probability is conserved.

$$
\frac{\partial}{\partial t} \left( \rho P \right) + \frac{\partial}{\partial x_{i}} \left( \rho u_{i} P \right) + \frac{\partial}{\partial \psi_{k}} \left( \rho S_{k} P \right)
=
- \frac{\partial}{\partial x_{i}} \left[ \rho \left< u_{i}'' | \psi \right> P \right] + \frac{\partial}{\partial \psi_{k}} \left[ \rho \left< \frac{1}{\rho} \frac{\partial J_{i,k}}{\partial x_{i}} | \psi \right> P \right]
$$

The terms are, respectively, the change in the joint PDF of composition over time, the change due to species momentum transport, the change due to the reaction rate in composition space, the change in probability distribution due to the fluid velocity fluctuation, and the change due to molecular diffusive flux.  All terms on the right-hand side require modeling since they do not have closed-form solutions.

A Monte Carlo method is then used to solve this transport equation.  Notional particles are used which move randomly through physical and compositional space.  Particle convection, mixing, and reaction are treated separately.  The largest modeling error is particle mixing at the molecular level.  This is similar to the particle-in-cell method used in plasma dynamics.

- [Li & Modest, “A hybrid finite volume/PDF Monte Carlo method to capture sharp gradients in unstructured grids”](https://eng.ucmerced.edu/people/mmodest/portal/publications/proceedings-articles/pdfimece00.pdf)

The other two approaches mentioned are more complicated than our current treatment, such as involving multiphase flow behavior.

- Which of these do you think is most apt for MIRGE-Com?  Why?
- Examine the [MIRGE-Com Pyrometheus model](https://github.com/ecisneros8/pyrometheus) and classify it per the above schema.

---

https://mirgecom.readthedocs.io/en/latest/operators/gas-dynamics.html?highlight=initializer
https://mirgecom.readthedocs.io/en/latest/support/thermochem.html
