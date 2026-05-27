**Motion Planning**

*Motion planning finds a collision-free path from start to goal through configuration space. Optimization-based methods (CHOMP, TrajOpt) use sequential convex optimization with collision constraints. Sampling-based methods (PRM, RRT, RRT*) build graphs or trees via random sampling, offering probabilistic completeness.*

**Motion Planning Problem**

- Given: start state $x_S$, goal state $x_G$
- Find: a sequence of control inputs leading from start to goal
- Challenges: obstacle avoidance; underactuated dynamics that prevent moving along any coordinate at will
- Examples: helicopter path planning, cartpole swing-up, acrobot

**Configuration Space (C-Space)**

- $\mathcal{C} = \{ x \mid x \text{ is a pose of the robot} \}$
- C-space obstacles: regions where the robot would collide with workspace obstacles
- Free space: complement of C-space obstacles
- Motion planning reduces to finding a continuous path through free space

**Optimization-Based Motion Planning**

Trajectory optimization formulation:

$\min_{\theta_{1:T}} \sum_t \|\theta_{t+1} - \theta_t\|^2 + \text{other costs}$

subject to:

- $\theta_0 = \text{start state}$, $\theta_T$ in goal set
- Joint limits
- For all robot parts, for all obstacles: $\text{sd}(\theta) > 0$ (signed distance $>$ 0 means no collision)

The problem is non-convex due to collision constraints.

**Collision Constraints**

- Signed distance between convex bodies $A$ and $B$: $sd_{AB}(\theta) \approx \hat{n} \cdot (p_B - p_A(\theta))$
- Linearized approximation: $sd_{AB}(\theta) \approx sd_{AB}(\theta_0) - \hat{n}^\top J_{p_A}(\theta_0)(\theta - \theta_0)$
- $J_{p_A}$ is the Jacobian of the closest point on body $A$ w.r.t. configuration
- Gilbert-Johnson-Keerthi (GJK) algorithm and Expanding Polytope Algorithm (EPA) compute signed distance efficiently

**Collision Cost as L1 Penalty**

- Penalize when signed distance falls below $d_{\text{safe}}$:
  $\text{penalty} = \max(0, d_{\text{safe}} - sd)$
- Piecewise linear penalty: zero for $sd \ge d_{\text{safe}}$, ramps up linearly below $d_{\text{safe}}$

**Continuous-Time Safety**

- Check collision against swept-out volume (the region swept by the robot between discrete waypoints)
- Allows coarsely sampling the trajectory in time
- Overall faster computation and finds better local optima than per-timestep collision checking
- Essential for practical motion planning with limited discretization

**Reactive and Optimization-Based Methods**

- Potential-based methods [Khatib, 1986]: reactive gradient following
- Elastic bands [Quinlan and Khatib, 1993]: deform trajectory away from obstacles
- CHOMP [Ratliff et al., 2009]: covariant gradient descent with obstacle cost
- STOMP, ITOMP: stochastic trajectory optimization variants
- TrajOpt [Schulman et al., 2013]: sequential convex optimization with collision constraints
  - Code: rll.berkeley.edu/trajopt
  - Benchmark: github.com/joschu/planning_benchmark

**Sampling-Based Motion Planning**

**Probabilistic Roadmap (PRM)**

- Randomly sample configurations in C-space
- Test sampled configurations for collision; retain collision-free ones as milestones
- Link each milestone by straight paths to nearest neighbors
- Retain collision-free links to form the roadmap
- Include start and goal configurations as milestones
- Search the PRM graph for a path from start to goal

Algorithm:

- Initialize point set with $x_S$ and $x_G$
- Randomly sample points in configuration space
- Connect nearby points if reachable from each other
- Find path from $x_S$ to $x_G$ in the graph
- Alternatively: track connected components incrementally; declare success when $x_S$ and $x_G$ are in the same component

PRM properties:

- Pro: probabilistically complete (with probability 1, if run long enough, will find a solution if one exists)
- Cons: requires solving 2-point boundary value problems; builds graph over entire state space (unnecessarily expensive for specific start-goal queries)

**Sampling Techniques**

- Uniform sampling from $[0,1]^n$: sample each coordinate uniformly
- Uniform sampling from unit $n$-sphere surface: sample from $n$-D Gaussian, then normalize
- Uniform sampling for 3D orientations: sample quaternions uniformly

**Rapidly-Exploring Random Tree (RRT)** [LaValle, 1998]

- Build a tree by generating next states through random controls
- Basic RRT procedure:
  1. Sample a random configuration $x_{\text{rand}}$ (99% random, 1% goal to bias toward solution)
  2. Find the nearest node in the tree: $x_{\text{near}}$
  3. Select a control input that drives $x_{\text{near}}$ toward $x_{\text{rand}}$, generating $x_{\text{new}}$
  4. Add $x_{\text{new}}$ and the edge to the tree
- Biases exploration toward largest Voronoi regions (unexplored areas)

**RRT Practicalities**

- Nearest neighbor search: KD-trees (works well up to ~20D), Locality Sensitive Hashing
- Control selection (SELECT_INPUT):
  - Sample a few inputs, retain the one producing $x_{\text{new}}$ closest to $x_{\text{rand}}$
  - Can run optimization to find the best input
  - Essentially solves a 2-point boundary value problem
- Extension: holonomic with no obstacles is trivial; non-holonomic requires approximate boundary value problem (often: pick best of random control sequences)

**RRT Variants**

Bi-directional RRT: grow trees from both $x_S$ and $x_G$ simultaneously

- Volume swept out by bi-directional RRT is far larger than unidirectional, making connection faster
- Difference more pronounced as dimensionality increases

Multi-directional RRT: planning around obstacles or narrow passages can be easier in one direction

Resolution-Complete RRT (RC-RRT):

- Problem: nodes stuck behind obstacles are chosen too often for expansion
- Solution: each node has a maximum number of expansion attempts $m$
- Track Constraint Violation Frequency (CVF):
  - Initialize CVF to zero when node is added
  - On unsuccessful expansion: increase CVF of node by 1, parent by $1/m$, grandparent by $1/m^2$, etc.
  - When node is selected for expansion, skip with probability $\text{CVF}/m$

**RRT\*** [Karaman and Frazzoli]

- Asymptotically optimal: converges to the optimal path with infinite samples
- Main idea: when adding a new node, rewire nearby nodes to use the new node as parent if it provides a shorter path
- RRT* for kinodynamic systems [Li, Littlefield, Bekris, 2014]: asymptotic optimality from random control sampling without solving a 2-point boundary value problem
- SST*: uses pruning for efficiency

**PRM\* Probabilistic Bounds** [Dobson, Moustakides, Bekris, 2014]

- Finite-time bounds on the current best path cost relative to the optimal cost

**LQR-Trees** [Tedrake, IJRR 2010]

- Grow a randomized tree of stabilizing controllers to the goal
- Like RRT, but each node has an LQR controller that stabilizes a region around it
- Can discard sample points already in stabilized regions
- Builds a feedback policy over the entire reachable state space

**Smoothing**

Randomized motion planners produce jagged paths, often longer than necessary. Smoothing methods:

- Shortcutting: along the found path, pick two vertices $x_{t_1}, x_{t_2}$ and try to connect them directly, skipping intermediate vertices
- Nonlinear trajectory optimization (trajopt): optimize with an objective including smoothness in state, small control inputs, etc.

**Key Applications / Classic Examples**

- Industrial box picking (TrajOpt benchmark)
- DRC (DARPA Robotics Challenge) humanoid manipulation
- PR2 mobile manipulation planning
- Steerable needle path planning for medical procedures
- Brachytherapy implant channel layout optimization
- Dubin's car collision-free trajectory generation$
$