**Solving Continuous MDPs with Discretization**

*Continuous state-space MDPs become tractable by gridding the state space, reducing it to a discrete MDP solvable with value iteration. Lookahead policies, quality guarantees, and connections to function approximation extend the practical utility of discretization.*

**Continuous State Spaces Challenge**

- $S$ is a continuous set
- Value iteration becomes impractical: requires computing Bellman backups for all states $s \in S$
- Solution: Markov chain approximation via discretization

**Discretization Procedure**

- Grid the state space: vertices become discrete states
- Reduce the action space to a finite set (sometimes unnecessary if the Bellman backup can be computed exactly over continuous actions, or if the optimal policy is known to be bang-bang)
- Original MDP $(S, A, T, R, \gamma, H)$ becomes discretized MDP $(\bar{S}, \bar{A}, \bar{T}, \bar{R}, \gamma, H)$

**Discretization Approach 1: Snap onto Nearest Vertex**

- Discrete states: $\{\xi_1, \ldots, \xi_n\}$ — the grid vertices
- Each continuous next state is snapped to the nearest grid vertex
- If a $(s, a)$ pair can result in many different next states: sample from the next-state distribution and assign to nearest vertices

**Discretization Approach 2: Stochastic Transition onto Neighboring Vertices**

- Use barycentric coordinates: assign transition weights to neighboring vertices based on the continuous next state's position
- For a 2D grid with normalized coordinates $[0,1] \times [0,1]$, a next state $s' = (x,y)$ transitions to four neighboring vertices with weights proportional to rectangular areas
- Kuhn triangulation: enables efficient computation of participating vertices and convex interpolation weights in higher dimensions [Munos and Moore, 2001]

**How to Act After Discretization**

After solving the discrete MDP, we have a policy and value function defined only at discrete states. Need strategies for any state:

**No Lookahead**

- Nearest neighbor: choose action based on the nearest discrete state
- Stochastic interpolation: for $s = p_2\xi_2 + p_3\xi_3 + p_6\xi_6$, choose $\pi(\xi_2)$, $\pi(\xi_3)$, $\pi(\xi_6)$ with probabilities $p_2, p_3, p_6$
- For continuous actions: interpolate the action itself

**1-Step Lookahead**

- Forward simulate one step, calculate $R + \gamma V(\text{next state})$ using the discrete MDP value function
- Nearest neighbor: use nearest discrete state's value
- Stochastic interpolation: interpolate values at neighboring vertices
- If dynamics are deterministic, no expectation needed; if stochastic, approximate with samples

**n-Step Lookahead**

Options for action selection:

- Enumerate sequences of discrete actions from value iteration
- Randomly sampled action sequences (random shooting)
- Run optimization over the actions (local gradient descent, cross-entropy method)

**Cross-Entropy Method (CEM)**

- Black-box method for approximately solving $\max_x f(x)$ with $x \sim \mathcal{N}(\mu, \Sigma)$
- Algorithm:
  - Sample $N$ candidates $x^{(e)} \sim \mathcal{N}(\mu, \Sigma)$
  - Evaluate $f(x^{(e)})$ for each
  - Keep top $k\%$ (e.g., 10%) of samples
  - Update $\mu$ and $\Sigma$ to the mean and covariance of the elite samples
  - Repeat
- For discrete actions: compute frequency of each action in the elite set and sample from resulting distribution
- Variations: max-ent version uses weighted mean based on $\exp(f(x))$

**Discretization Quality Guarantees**

Typical guarantees rely on smoothness of cost function and transition model:

- As grid spacing $h \to 0$, the discretized value function approaches the true value function
- One-step lookahead policy based on $V$ close to $V^*$ attains value close to $V^*$

Proof techniques:

- Chow and Tsitsiklis, 1991: show one discretized backup is close to one complete backup; then sequence of backups is also close
- Kushner and Dupuis, 2001: show sample paths in discrete stochastic MDP approach sample paths in continuous MDP
- Function approximation based proofs [Gordon, 1995; Tsitsiklis and Van Roy, 1996]

**Connection with Function Approximation**

Discretization as function approximation:

- Nearest neighbor: builds piecewise constant (0th order) approximation of value function
- Stochastic transition onto nearest neighbors: builds $n$-linear (1st order) approximation
- Kuhn triangulation: piecewise (over triangles) linear approximation of value function

Value iteration with function approximation interpretation:

- Start with $\hat{V}_0(s)$ for all $s$
- For $i = 0, 1, \ldots, H-1$, for all states in the discrete set $\bar{S}$:
  $\bar{V}_{i+1}(s) = \max_a \sum_{s'} T(s,a,s') [R(s,a,s') + \gamma \hat{V}_i(s')]$
- Then fit $\hat{V}_{i+1}(s) \approx \theta^\top \phi(s)$ via function approximation

**Continuous Time Considerations**

- Discretize time variably so that one discrete time transition roughly corresponds to a transition into neighboring grid points
- Discounting: $\delta t$ depends on state and action [Munos and Moore, 2001]
- Connection between time and space discretization: CFL (Courant-Friedrichs-Lewy) condition from numerical methods
- 1-nearest neighbor is sensitive to time-space mismatch; can cause many states to self-transition regardless of action

**Key Applications**

- Mountain Car: nearest neighbor and linear interpolation with 20–150 discrete values per state dimension
- Continuous control benchmarks with state dimensions up to ~5–6 (beyond this, discretization becomes computationally infeasible)$
$