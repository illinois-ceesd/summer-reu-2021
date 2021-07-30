#  Verification & Validation and Software Quality

* [Slides](./slides/6-verval.pdf)

---

- Identify verification procedures and situate them in the context of quality assurance.
- Identify validation procedures and available means.
- Distinguish model and software applications of V&V.

Verification and validation procedures form an important step in obtaining and demonstrating confidence in your code products.  V&V can be carried out over the model and over the software.  We are concerned with both _model V&V_ and _software V&V_ in working on MIRGE-Com.

**Model V&V**.  To keep things straight, think of a physical model used in engineering.  _Verification_ would mean making sure that the mathematical model is correctly implemented.  _Validation_ would mean making sure that the mathematical model actually describes reality.

> Model verification and validation are the primary processes for quantifying and building credibility in numerical models.
>
> - *[Model] verification* is the process of determining that a model implementation accurately represents the developer’s conceptual description of the model and its solution.
> - *[Model] validation* is the process of determining the degree to which a model is an accurate representation of the real world from the perspective of the intended uses of the model.  Both verification and validation are processes that accumulate evidence of a model’s correctness or accuracy for a specific scenario; thus, V&V cannot prove that a model is correct and accurate for all possible scenarios, but, rather, it can provide evidence that the model is sufficiently accurate for its intended use.
>
> *Model V&V is fundamentally different from software V&V.*  Code developers developing computer programs perform software V&V to ensure code correctness, reliability, and robustness.  In model V&V, the end product is a predictive model based on fundamental physics of the problem being solved.
>
> **The expected outcome of the model V&V process is the quantified level of agreement between experimental data and model prediction, as well as the predictive accuracy of the model.**  Model V&V is undertaken to quantify confidence and build credibility in a numerical model for the purpose of making a prediction.  [[Concepts of Model Verification and Validation, LA-14167-MS](http://www.ltas-vis.ulg.ac.be/cmsms/uploads/File/LosAlamos_VerificationValidation.pdf)]

**Software V&V**.  Software verification and validation are separate quality assurance processes.

- **Software Verification**:  Is the software product built correctly?
- **Software Validation**:  Is the software product the correct product for the customer or application?

Well-posed specifications and requirements are necessary to ensure that V&V can be carried out properly.  Independent V&V processes can be carried out by a third party auditor for mission-critical or safety-critical applications.

Software specifications vary from simple descriptions, like unit tests in test-driven development, to complex documents; these latter are more common for enterprise systems.  MIRGE-Com develops its reference documentation in tandem with the code towards overarching objectives in modeling capabilities.

- [“Complete Software Requirements Specification (SRS) Example”](https://myhistoryfeed.medium.com/complete-software-requirements-specification-srs-example-5bd08f107206)
- [Frigg, R., and J. Reiss, 2009. “The philosophy of simulation: Hot new issues or same old stew,” Synthese, 169: 593–613.](http://www.romanfrigg.org/writings/Synthese_Simulation.pdf)
- [MIRGE-Com Reference Documentation](https://mirgecom.readthedocs.io/en/latest/)
- [Stanford Encyclopedia of Philosophy, “Computer Simulations in Science,” section “Verification & Validation”](https://plato.stanford.edu/entries/simulations-science/#VerVal)
- [Verification and Validation in Computing](https://www.krellinst.org/csgf/conf/2017/video/mheroux)
- [“Y1/Y2 Center Simulation Goals” (CEESD)](https://uofi.app.box.com/s/xwszldmmqkwez7gtyzkcws7ibhcy3ifv)


##  Concepts of Model V&V

**Model verification**.  Model verification needs to establish that the nominal model and the model being solved coincide.  Examples where they would not include mis-written mathematical expressions or distributed code that misdirects messages to the wrong ranks.

> Verification assessment examines 1) if the computational models are the correct implementation of the conceptual models, and 2) if the resulting code can be properly used for an analysis.  The strategy is to identify and quantify the errors in the model implementation and the solution.  The two aspects of verification are the verification of a code and the verification of a calculation.  The objective of verifying a code is error evaluation, that is, finding and removing errors in the code.  The objective of verifying a calculation is error estimation, that is determining the accuracy of a calculation.  [[NASA CFD Verification and Validation Website](http://www.grc.nasa.gov/WWW/wind/valid/tutorial/tutorial.html)]

Model verification is relatively the easier of the two tasks if existing references for model behavior exist.  For instance, if analytical solutions or well-understood test cases exist for the equations in question, then the output of the code can be directly compared.

- For MIRGE-Com, consider how model verification could take place.

**Model validation**.  “[Model] validation examines if the conceptual models, computational models as implemented into the CFD code, and computational simulation agree with real world observations” (NASA, op. cit.)  In other words, does the nominal model actually describe the desired phenomenon?  A trivial example where validation would fail is in modeling Ohm's law in the presence of current-driven temperature changes in conductivity, up to and beyond the point of failure (melting).

> One can only validate the code for a specific range of applications for which there is experimental data. Thus one validates a model or simulation. Applying the code to flows beyond the region of validity is termed prediction.  [NASA, op. cit.]

Model validation can be quite difficult due to the need to tie it directly to real-world performance, not least because experimental data are not as tidy as computational data.

Considerations which need to be made in model validation include:

-   [Selectivity and specificity](https://mpl.loesungsfabrik.de/en/english-blog/method-validation/specificity-selectivity)
-   Accuracy and precision of results
-   Repeatability
-   Reproducibility
-   System suitability

It's never as simple as simply trawling through the literature for a roughly-matching experiment.

> The process for Validation Assessment of a CFD simulation can be summarized as:
>
> 1.  Examine Iterative Convergence.
>    Validation assessment requires that a simulation demonstrates iterative convergence. Further details can be page entitled Examining Iterative Convergence.
>
> 2.  Examine Consistency.
>    One should check for consistency in the CFD solution. For example, the flow in a duct should maintain mass conservation through the duct. Further total pressure recovery in an inlet should stay constant or decrease through the duct.
>
> 3.  Examine Spatial (Grid) Convergence.
>    The CFD simulation results should demonstrate spatial convergence. Further details and methods can be found on the page entitled Examining Spatial (Grid) Convergence.
>
>4.  Examine Temporal Convergence.
>    The CFD simulation results should demonstrate temporal convergence. Further details and methods can be found on the page entitled Examining Temporal Convergence.
>
>5.  Compare CFD Results to Experimental Data.
>    Experimental data is the observation of the "real world" in some controlled manner. By comparing the CFD results to experimental data, one hopes that there is a good agreement, which inreases confidence that the physical models and the code represents the "real world" for this class of simulations. However, the experimental data contains some level of error. This is usually related to the complexity of the experiment. Validation assessment calls for a "building block" approach of experiments which sets a hierarchy of experiment complexity.
>
>6.  Examine Model Uncertainties.
>    The physical models in the CFD code contain uncertainties due to a lack of complete understanding or knowledge of the physical processes. One of the models with the most uncertainty is the turbulence models. The uncertainty can be examined by running a number of simulations with the various turbulence models and examine the affect on the results.
>
> [[NASA CFD Verification and Validation Website](http://www.grc.nasa.gov/WWW/wind/valid/tutorial/tutorial.html)]

- For MIRGE-Com, consider how model validation could take place.


##  Concepts of Software V&V

**Software verification**.  Software verification is the process of methodically evaluating your code, using tests to determine if behavior adheres to the specification.  (You can see how this is analogous to reasoning about the model.)

Complete _verification_ is a misnomer:  a program can never be demonstrated correct, only to not have certain faults.  Thus verification is really much more like the principle of falsification in the philosophy of science.

> Program testing can be used to show the presence of bugs, but never to show their absence.  (Dijkstra)

Software verification tests are those tests which check that processes work as they should.  (_Most_ tests fall into this category; others frequently fall under the opposite heading of ensuring appropriate failure.)

Various kinds of tests are available, beyond basic unit tests:

> Five typologies of experiments can be distinguished in the process of specifying, implementing, and evaluating computing artifacts.
>
> -   Feasibility experiments are performed to evaluate whether an artifact of interest performs the functions specified by users and stakeholders;
> -   trial experiments are more specific experiments carried out to evaluate isolated capabilities of the system given some set of initial conditions;
> -   field experiment are performed in real environments and not in simulated ones;
> -   comparison experiments test similar artifacts, instantiating in different ways the same function, to evaluate which instantiation better performs the desired function both in real-like and real environments;
> -   finally, controlled experiments are used to appraise advanced hypotheses on the behaviors of the testing artifact.
>
> [[SEP, "Philosophy of Computer Science"](https://plato.stanford.edu/entries/computer-science/)]

Computer security techniques are occasionally used in software verification.  For instance, [_fuzzing_](https://en.wikipedia.org/wiki/Fuzzing) is a technique of throwing random or targeted data at a particular interface to discover its behavior and limitations.  Fuzzing can be used maliciously to obtain access to restricted data, but fuzzing can also be used to test software behavior.

**Software validation**.  Software validation is the process of making sure that the software product being developed can actually speak to the desired end.

In software, requirements are used to validate software.  Requirements are spoken of as coming from a _customer_, but not infrequently the "customer" will be the general public, a legal requirement, or the software developer himself or herself.  For MIRGE-Com, the end-use application of modeling scramjets obtains as the practical set of requirements.

> A validation action applied to an engineering element includes the following:
    - Identification of the element on which the validation action will be performed.
    - Identification of the reference that defines the expected result of the validation action.
>
> Performing the validation action includes the following:
    - Obtaining a result by performing the validation action onto the submitted element.
    - Comparing the obtained result with the expected result.
    - Deducing the degree of compliance of the element.
    - Deciding on the acceptability of this compliance, because sometimes the result of the comparison may require a value judgment to decide whether or not to accept the obtained result as compared to the relevance of the context of use.
>
> [["System Validation", _Systems Engineering Body of Knowledge_](https://sebokwiki.org/wiki/System_Validation)  (this is a treasure trove, by the by)]

---

##  Exercises

1. Would each of the following actions be better classified as a _validation_ or as a _verification_?

    | Action | Classification |
    | ------ | -------------- |
    | Unit testing for network I/O | __________ |
    | Checking the implementation of network I/O | __________ |
    | Writing documentation for a method | __________ |
    | Adhering to an API specification | __________ |
    | Testing that an object model completely enumerates possible cases | __________ |
    | Fuzzing a system to determine behavior | __________ |

The following two exercises are from a CFD class I ran a few years ago, ME 498CF.  They don't directly work in MIRGE-Com, but use them as inspiration for thinking about how to carry out V&V in MIRGE-Com.

2. The order of grid (spatial) convergence involves the behavior of the discretization error term ε with regards to the grid spacing $\Delta x$,

    $$
    \epsilon = f(\Delta x) – f_{\text{exact}} = C×(\Delta x)_{p} – \text{higher order terms}
    $$

    where $p$ is the resulting order of convergence.  Taking the logarithm, it becomes apparent how to find the value of $p$:

    $$
    \log \epsilon = \log C + p \log (\Delta x)
    $$

    We simply need to find a few points of the curve $\log \epsilon$ versus $\log (\Delta x)$ and take the slope to obtain $p$.  If the grid refinement ratio $r$ is constant, then the equation simply becomes:

    $$
    p \log r = \log \frac{f_3 - f_2}{f_2 - f_1}
    $$

    We simply need to obtain the solutions $f_i$ in order to calculate $p$.  (Note that we can distinguish a local from a global order of accuracy by our choice of $f_i$—in general the global order of accuracy is one degree less than the local.)

    - We’ll use a driven cavity flow for a square cavity of 1 m × 1 m square.  The flow is laminar and the moving wall moves at the rate 0.0001 m/s for Re = 1000.  (Set these conditions; use air.)

    1. Solve the problem to convergence with mesh size 0.01 m, then refine the mesh to 2× resolution and solve the problem again.  Now to 4× (original) resolution.  Obtain the values fi by querying the pressure at a selected point (which should be available as a cell vertex on all three grids, such as (0.25, 0.25)).  Plot these using Excel or another tool to obtain $p$.  Fill out the values below.
        1. point at ________, ________, ________
        2. $r$ = ________
        3. $f_{1×}$ = ________
        4. $f_{2×}$ = ________
        5. $f_{4×}$ = ________
        6. $p$ = ________
    2. Does the problem converge at the same rate in other variables (such as velocity)?  Elaborate.

3.  One chief challenge of validation of numerical experiments is the difference in reported quantities v. measurable quantities. Another challenge is that many CFD experiments are idealizations which cannot be physically reproduced. Sometimes this is sidestepped by validating with reference to a well-characterized benchmark problem, such as Rayleigh–Taylor mixing or 1D shock propagation.  We will examine step 5 of the validation assessment process referenced in the notes by comparing to experimental data on the driven square cavity problem.

    Prasad & Koseff experimentally simulated a lid-driven cavity flow.  Although they report a number of factors, the easiest ones for us to check are the $x$ and $y$ velocities along the respective centerlines.  Their simulation is at much higher Re, so while we anticipate qualitative agreement our calculated values will not match theirs.
        3. Plot the $x$ and $y$ velocities along the centerlines and compare the results to the plots on pp. 211–213 of Prasad & Koseff.

    - [Slater, J. W. (cur.)  (2008)  Examining grid (spatial) convergence.  NPARC Alliance CFD Verification and Validation Web Site](http://www.grc.nasa.gov/WWW/wind/valid/tutorial/spatconv.html)
    - [CFD-Wiki, “2-D laminar/turbulent driven square cavity flow”](http://www.cfd-online.com/Wiki/2-D_laminar/turbulent_driven_square_cavity_flow)
    - [Prasad, A. K., Koseff, J. R.  (1988)  Reynolds number and end-wall effects on lid-driven cavity flow.  Physics of Fluids A 1(2), pp. 208–218](https://www.researchgate.net/publication/238041375_Reynolds_number_and_end-wall_effects_on_a_lid-driven_cavity_flow)
