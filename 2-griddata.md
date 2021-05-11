#   Grid Data Structures

* [Slides](./slides/2-griddata.pdf)

---

- Describe schemes for representing grids efficiently on a computer.
- Analyze MIRGE-Com's grid representation scheme.

##  Grid Data

A discretized differential equation over both conserved state quantities and emergent unconserved quantities (such as vorticity) can be represented in many formats of varying computational efficiency and ease-of-use.

At the most naïve end of the spectrum, a battery of raw arrays containing only `float`s or `double`s may be employed:

```c
#define N 100
double X[N];     /* x-coord of grid */
double Y[N];     /* y-coord of grid */
double T[N] = {[0 ... N]0.0};     /* temperature */
double V[3][N];  /* 3D vector velocity */
```

In modern C, variable-length arrays defined on the stack rather than the heap (`malloc`) may be employed.  (Depending on the size of the array, this may or may not be desirable on a cluster machine with relatively small memory available.)

```c
int n;           /* n = 100 from some dynamic source such as a config file */
double X[n];     /* x-coord of grid */
double Y[n];     /* y-coord of grid */
double T[n];     /* temperature */
double V[3][n];  /* 3D vector velocity */
```

In Python, which elides much of the complexity of direct memory management, this simply becomes

```py
import numpy as np

n = 100
X = np.linspace(x0,xf,n+1)
Y = np.linspace(x0,xf,n+1)
T = np.zeros(size=(n,))
V = np.zeros(size=(n,3))
```

where NumPy handles the backend memory allocation and access.  (`np.empty` may be preferred for speed since it doesn't overwrite memory with values.)

The raw-array approach has the benefit of being very transparent, since each value is directly accessible in its place, but rather scattered in its quantities.  _C'est la vie_:  most simulations are built like this.

Perhaps a more rigorous approach would be to have a `MeshPoint` structure or class which tabulates the values of all state variables at a single mesh location.  (A class requires an object-oriented language, so you wouldn't see this in raw C.)  

In C:

```c
struct MeshPoint {
  double X;
  double Y;
  double T;
  double V[3];
};
```

In Python, it's a bit more work:

```py
class MeshPoint(object):
    def __init__(self,
                 x : float,
                 y : float,
                 t : float,
                 v : np.ndarray
                 ) -> None:
        '''Defines x and y variables'''
        self._X = x
        self._Y = y
        self._T = t
        assert (v.shape[0] == 3) and (v.shape[1] <= 1)
        self._V = v.copy()

    @property
    def X(self):
        return self._X

    @X.setter
    def X(self, x):
        self._X = x

    # and so forth
```

The downside of this method is that data transfer and access tend to be relatively slow compared to bare arrays.

MIRGE-Com opts for a hybrid version of these, using a single aggregate NumPy array of all array values.  The MIRGE-Com mesh and elements are produced as discussed in “Discretization”:

```py
from meshmode.mesh.generation import generate_regular_rect_mesh
mesh = generate_regular_rect_mesh(
    a=(-0.5,-0.5),
    b=( 0.5, 0.5),
    n=(nel_1d, nel_1d))

from grudge.eager import EagerDGDiscretization
discr = EagerDGDiscretization(actx, mesh, order=order)
```

These elements are integrated forward by a stepper function `advance_state`.  Typically, the workhorse Runge-Kutta 4th-order method `rk4_step` plays this role.  This assumes the `rhs` function deals with mesh connectivity, such as `wave_operator`.

```py
def my_rhs(t, state):
    return euler_operator(discr,
                          q=state,
                          t=t,
                          boundaries=boundaries,
                          eos=eos)

rhs = (current_step, current_t, current_state) = \
            advance_state(rhs=my_rhs,
                          timestepper=rk4_step,
                          checkpoint=my_checkpoint,
                          get_timestep=get_timestep,
                          state=current_state,
                          t=current_t,
                          t_final=t_final)
```

- Read and run the examples `sod-mpi.py` and `heat-source-mpi.py` provided with MIRGE-Com.
- Examine the output using ParaView.
- Modify the domain partition used in one of these codes to utilize more effective ranks (and thus subdivide in more pieces).


##  Distributed Computing

When resolving a large, long-time-spanning, or detailed model, it is necessary to bring more computing power to bear than a single workstation can provide.  Various forms of distributed or parallel computing, including the use of graphics processing units (GPUs) and computing clusters, are frequently employed in scientific modeling to allow up to tens of thousands of CPUs to focus on solving a particularly thorny problem in a single battery.  Distributed computing reequires the developer to adopt a scheme of carving up a problem into parts; one of the most common methods, and the one that MIRGE-Com uses, is to divide the physical domain being simulated into different chunks, each one evaluated in parallel on a different processor.

Some terminology is in order:  we call a single CPU a _processor_ and refer to it by its _rank_, a unique cardinal number.  (Typically all activity is coordinated by the rank-zero processor, the _manager rank_.)  A collection of processors on a single board is a _node_; nodes can have from one to 32 processors on them, depending on the system architecture.  Sometimes nodes also have a number of _GPUs_ available, as on Lassen or Campus Cluster.  GPUs are used for specialized calculations since they have strict categories of efficiency and memory operations to and from them are relatively expensive.

You don't get anything for free in parallel computing:  although some problems can be readily computed without reference to other points, as soon as dependencies along the grid are required it becomes critical to coordinate activity and information.  If you are interested in learning more about the fundamental benefits and limits of parallel computing, check out CS 240 “Parallel Programming for Science and Engineering” or CS 484 “Parallel Programming”.

Given a discretized domain chopped up across many processors, each processor must, at each time step, marshall all values needed for calculations on other processors and share them while listening for the data it needs.  Such grid elements as are necessary to communicate are commonly called _boundary elements_.  When gathering information in order to later send it, it is common to refer to the conceptually adjacent processors as _ghost ranks_.

One way of producing a mesh is illustrated in this simplified 2D example from `wave-eager-mpi.py`, which produces a unit square mesh centered at the origin:

```py
if mesh_dist.is_manager_rank():
    from meshmode.mesh.generation import generate_regular_rect_mesh
    mesh = generate_regular_rect_mesh(a=(-0.5,-0.5),
                                      b=(+0.5,+0.5),
                                      n=(16,16))

    print(f"{mesh.nelements} elements")
    part_per_element = get_partition_by_pymetis(mesh, num_parts)
    local_mesh = mesh_dist.send_mesh_parts(mesh, part_per_element, num_parts)
    del mesh

else:
    local_mesh = mesh_dist.receive_mesh_part()
```

At the beginning, the mesh only consists of a collection of spatial points, which do not yet carry any state information.  These mesh points are assigned to distributed ranks by PyMetis.  To be clear, _only_ the first rank (`0`) generates the mesh; the other ranks merely listen for their local assignments.

After that has taken place, the mesh discretization along these points takes place by actually constructing the discrete elements:

```py
discr = EagerDGDiscretization(actx, local_mesh, order=3, mpi_communicator=comm)
```

Grudge seamlessly handles distributed-memory operations through the `_RankBoundaryCommunication` class in `grudge/eager.py`.  This uses non-blocking `Isend`/`Irecv` MPI operations in [MPI4Py](https://mpi4py.readthedocs.io/en/stable/).  Later `Wait` ensures completion before the results are handled.

- Why are non-blocking processes used here?
- Examine the distributed-memory functionality block in `grudge/eager.py`.
