#   Chemical Reactivity in Flows

-   Understand the various models of chemical reaction available in Fluent.
-   Model a chemically reactive system.
-   Use adaptive mesh refinement in a simulation.

#### Lecture
-   Chemical reactivity
-   Adaptive mesh refinement

#### Labs
-   Chemical reactivity (guided?) [need to review]
-   AMR [15:  Using Dynamic Meshes] [transient jet nozzle]

#### Order of Class
*   Chemistry lecture   20m 14h00
*   Reactivity lab      25m 14h20
*   Break               5m  14h45
-   AMR lecture         10m 14h50
*   AMR lab             20m 15h00  [transient jet nozzle redux]

#   Chemistry in Fluent

Fluent can model species transport and reaction using several independent models.

-   Species Transport and Finite-Rate Chemistry Approach

-   Mixture Fractions Approach

-   Reaction Progress Variable Approach

-   Composition PDF Transport Approach

-   Multiphase Species Transport

    This is a subset of multiphase flow behavior, which lies beyond our scope.

A few other models are also supported in detail, such as the internal combustion engine model.

In addition, the enthalpy term in the original transport equation needs to be included.

##  Species Transport Approach

This approach solves the conservation equations for convection, diffusion, and reaction sources for multiple component species.  The user may specify multiple chemical reactions to model simultaneously, with reactions occurring either in the bulk flow, at wall or particle surfaces, or in the porous region.

$$\frac{\partial}{\partial t} \left( \rho Y_{i} \right) + \nabla \cdot \left( \rho \arrow{v} Y_{i} \right) = - \nabla \cdot \arrow{J}_{i} + R_{i} + S_{i}$$

This conservation equation describes the convection and diffusion of the local mass fraction of a species $Y_{i}$.  $R_{i}$ is the rate of production by chemical reaction, and $S_{i}$ is the rate of creation by addition from the dispersed phase and user-defined sources.

The diffusion flux $\arrow{J}_{i}$ occurs due to gradients of concentration and temperature.  Fick's law is the default:

$$\arrow{J}_{i} = - \rho D_{i,m} \nabla Y_{i} - D_{T,i} \frac{\nabla T}{T}$$

where $D_{i,m}$ is the mass diffusion coefficient and $D_{T,i}$ is the thermal diffusion coefficient.  This approximation is generally good.

With turbulence, further accommodation is necessary because mixing must be explicitly included as a function of turbulence at small length scales.

$$\arrow{J}_{i} = - \left( \rho D_{i,m} + \frac{\mu_t}{\text{Sc}_{t}} \right) \nabla Y_{i} - D_{T,i} \frac{\nabla T}{T}$$

Naturally, multicomponent transport introduces a number of significant physical effects into the system, including diffusion, enthalpy transport, and temperature gradients.

The reaction rate $R_i$ may be specified as:

-   Laminar finite-rate model (Arrhenius kinetics)

-   Eddy-dissipation model (turbulence-controlled; cheap)

-   Eddy-dissipation-concept model (both; very expensive)

Pressure effects may be included as well, if significant.

Surface reactions may be diffusion-limited or kinetics-limited; in the latter case, surface coverage may also be significant.  Fluent can track molar concentrations of wall-adsorbed species in the latter case.  Particle surface reactions can also be modeled (the char particle burning model).  (Incidentally, my experience is that Fluent is very much focused on modeling combustion rather than, say, the inside of a distillation tower.)

Reaction stoichiometry can be expressed quite generally:

$$R_{\text{kin},r} = \frac{A_r T^\beta \exp\left( \frac{E_r}{RT} \right)}{p_{r,d}^{N_{r,n}}} \product_{n=1}^{n_\text{max}} p_n^{N_{r,n}}$$

as a function of partial pressures $p_n$ and reaction orders $N_{r,n}$.

-   ANSYS Fluent Theory Guide, pp. 187–214


##  Mixture Fractions Approach (Non-Premixed Combustion)

In this approach, transport equations are written and solved for *mixture fractions* rather than for individual species.  Species concentrations are calculated from the resulting mixture fraction field.

The mixture fractions approach assumes that you are modeling a combusting system with discrete fuel and oxidizer inlets—so a diffuser, spray combustion chamber, or pulverized fuel flame.

The non-premixed combustion model rests on the assumption that the instantaneous thermochemical state of a fluid can be related to a conserved scalar quantity, the mixture fraction

$$f = \frac{Z_i - Z_{i,\text{ox}}}{Z_{i,\text{fuel}} - Z_{i,\text{ox}}}$$

Given what we know of likely combustion components, we may assume equal diffusivities for all species and write the density-averaged mixture fraction equation

$$\frac{\partial}{\partial t} \left( \rho \bar{f} \right) + \nabla \cdot \left( \rho \arrow{v} \bar{f} \right) = - \nabla \cdot \left( \frac{\mu_\text{lam} + \mu_\text{turb}}{\sigma_t} \nabla \bar{f} \right) + S_{m}$$

The source term $S_m$ arises from the transfer of mass into the gaseous phase from liquid fuel droplets or reacting particles.

This model then uses look-up tables to calculate thermal and chemical effects from the fluctuation in mixture fraction $f$.

-   ANSYS Fluent Theory Guide, pp. 215–252


##  Reaction Progress Variable Approach (Premixed Combustion)

<blockquote>
In premixed combustion, fuel and oxidizer are mixed at the molecular level prior to ignition. Combustion occurs as a flame front propagating into the unburnt reactants. Examples of premixed combustion include aspirated internal combustion engines, lean-premixed gas turbine combustors, and gas-leak explosions.
</blockquote>

Premixed combustion occurs as a thin propagating flame along the (possibly turbulent) boundary.  In contract, non-premixed combustion is typically a mixing problem.  For premixed combustion, the major challenge is to track the turbulent flame speed as a function of the laminar flame speed and the turbulence at the interface.

Mathematically, this model ends up mixing together pieces of what we've already discussed.  The simplest version reduces to a reaction progress variable $c$ written in a conservation equation as

$$\frac{\partial}{\partial t} \left( \rho \bar{c} \right) + \nabla \cdot \left( \rho \arrow{v} \bar{c} \right) = - \nabla \cdot \left( \frac{\mu_\text{turb}}{\text{Sc}_{t}} \nabla \bar{c} \right) + \rho S_{c}$$

The progress variable $c$ is defined as a normalized sum of product species mass fractions,

$$c = \frac{\sum_k \alpha_k \left( Y_k - Y_k^u \right)}{\sum_k \alpha_k Y_k^\text{eq} } = \frac{Y_c}{Y_c^\text{eq}}$$

The mean reaction rate is

$$\rho S_c = \rho_u U_t \left| \nabla c \right|$$

We have many options available to us for tracking the turbulent flame speed.  For instance, the $G$-equation is a premixed flame-front tracking model.  We will use the Zimont turbulent flame speed closure model, which accommodates wrinkled or thickened flame fronts.

<blockquote>
The model is based on the assumption of equilibrium small-scale turbulence inside the laminar flame, resulting in a turbulent flame speed expression that is purely in terms of the large-scale turbulent parameters.

The model is strictly applicable when the smallest turbulent eddies in the flow (the Kolmogorov scales) are smaller than the flame thickness, and penetrate into the flame zone.

The model is valid for premixed systems where the flame brush width increases in time, as occurs in most industrial combustors. Flames that propagate for a long period of time equilibrate to a constant flame width, which cannot be captured in this model.
</blockquote>

-   ANSYS Fluent Theory Guide, pp. 253–272

-   [Premixed Flame Propagation in a Tunnel](https://youtu.be/CjGuHbsi3a8?t=371)


##  Composition PDF Transport Approach (Finite-Rate Chemistry)

Fluent can use a probability density function (PDF) to describe finite-rate chemical kinetics in turbulent reacting flows.  For instance, you can use this approach to simulate effects such as flame extinction and ignition including species such as CO and NO<sub>x</sub>.  The advantage of this model (although computationally complex) is that it accommodates turbulent flow more accurately than the previous methods in some cases.

With the full species-transport approach discussed previously, Fluent uses Reynolds time-averaged flow to model turbulence.  This leads to unknown terms for turbulent scalar flux and mean reaction rate, which are modeled as "enhanced diffusion" and with a chemistry model, respectively.  This last in particular is difficult and prone to error.

Another approach is to derive a transport equation for their single-point joint probability density function $P$.  $P$ represents the fraction of the time that the fluid spends at each species, temperature, and pressure state.  Only the accessible portions of parameter space need be modeled, and obviously probability is conserved.

$$\frac{\partial}{\partial t} \left( \rho P \right) + \frac{\partial}{\partial x_{i}} \left( \rho u_{i} P \right) + \frac{\partial}{\partial \psi_{k}} \left( \rho S_{k} P \right) = - \frac{\partial}{\partial x_{i}} \left[ \rho \left< u_{i}'' | \psi \right> P \right] + \frac{\partial}{\partial \psi_{k}} \left[ \rho \left< \frac{1}{\rho} \frac{\partial J_{i,k}}{\partial x_{i}} | \psi \right> P \right]$$

The terms are, respectively, the change in the joint PDF of composition over time, the change due to species momentum transport, the change due to the reaction rate in composition space, the change in probability distribution due to the fluid velocity fluctuation, and the change due to molecular diffusive flux.  All terms on the right-hand side require modeling since they do not have closed-form solutions.

A Monte Carlo method is then used to solve this transport equation.  Notional particles are used which move randomly through physical and compositional space.  Particle convection, mixing, and reaction are treated separately.  The largest modeling error is particle mixing at the molecular level.  This is similar to the particle-in-cell method used in plasma dynamics.

-   ANSYS Fluent Theory Guide, pp. 281–288

-   [LiND](https://eng.ucmerced.edu/people/mmodest/portal/publications/proceedings-articles/pdfimece00.pdf):  Genong Li and Michael F. Modest, A hybrid finite volume/PDF Monte Carlo method to capture sharp gradients in unstructured grids.
