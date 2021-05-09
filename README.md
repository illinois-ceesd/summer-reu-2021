# CEESD Summer REU Tutorials

These tutorials are designed to acquaint computer science and engineering students with the physical and computational principles underlying [MIRGE-Com](https://github.com/illinois-ceesd/mirgecom), a software library developed by the [Center for Exascale-Enabled Scramjet Design](https://ceesd.illinois.edu/) for simulating scramjets on distributed systems.  MIRGE-Com is written in Python; although I make oblique reference to other languages like C, only familiarity with Python is expected to complete these lessons.

0. [The Physics of Scramjets](./0-physics.md)
  - Qualitative Behavior
  - Governing Equations
  - Physical Models
  - Assumptions & Simplifications
1. [Discretization & Mapping](./1-discrete.md)
  - Geometry
  - Structure
  - Special Requirements (Wall functions, etc.)
2. [Grid Data Structures](./2-griddata.md)
  - Representations
  - Distributed Computing
3. [Transport on Grids](./3-transport.md)
4. [Matrix Solvers & Conditioning](./4-solvers.md)
5. [Shock Modeling](./5-shocks.md)
6. [Software Quality/V&V](./6-verval.md)
7. [Chemistry Modeling in Flows](./7-chemrxn.md)
  - Combustion Chemistry
  - Reaction Coordinate Method

These tutorials were prepared by N E Davis for the [Center for Exascale-Enabled Scramjet Design](https://ceesd.illinois.edu/) in 2021.  The program's design was inspired by the [Los Alamos National Laboratory Computational Physics Student Summer Workshop](https://www.lanl.gov/org/padwp/adx/computational-physics/summer-workshop/index.php).  Some material draws from my 2016 course ME 498CF “Computational Fluid Dynamics with Fluent”.
