**Partially Observable Markov Decision Processes**

*Extending MDPs to settings with sensory measurements instead of full state observation, belief-space planning, and the separation principle for linear-Gaussian systems.*

**POMDP Formalism**

A POMDP is an MDP where the agent does not observe the true state — instead, it receives sensory measurements. The agent must choose actions based on its belief (probability distribution over states) rather than the state itself. This introduces a fundamental trade-off between actions that affect the underlying state and actions that gather information.

**Belief state update**: Given belief vector $b$ (length $|S|$), transition matrices $T_a$, and observation vector $o$:

$b^a = T_a b \quad \text{(predict)}$
$p(o \mid a, b) = o^T b^a \quad \text{(observation probability)}$
$b^{a,o} = \frac{\operatorname{diag}(o) \, b^a}{o^T b^a} \quad \text{(update)}$

**Policy**: A map from belief (a probability distribution) to actions. In the tiger example with initial belief $[0.5, 0.5]$, the policy roughly amounts to "listen until sufficiently certain, then open the door."

**Exact Solution Methods**

1. **Belief MDP**: Convert to a continuous-state MDP over the space of probability distributions. Value iteration would give $V$ and optimal action for every belief. Infeasible in general because the belief space is uncountable.

2. **Search over action/observation sequences**: Branch over actions and observations with finite look-ahead. The search tree has $(|A||O|)^H$ leaf nodes.

3. **MDP planning + filtering**: Plan in the full MDP, use filtering to track the belief, and choose the MDP-optimal action for the most likely state. Computationally efficient but fails to explicitly seek out information.

**Approximate methods**: $\alpha$-vector point-based techniques (scalable to millions of states), Monte Carlo Tree Search.

**(Gaussian) Belief Space Planning**

For low-cost, less precise robots, uncertainty can be parameterized by mean and covariance. State-space planning ignores uncertainty; belief-space planning accounts for it explicitly.

**Approach**: Convert underlying state-space dynamics to belief-space dynamics using a Bayesian filter (e.g., EKF). The belief state is $(\mu_t, \Sigma_t)$ with dynamics derived from the filter equations. Solve via trajectory optimization (e.g., sequential convex programming).

**Trade-offs**:
- Augmented state methods: can constrain states, bends into solutions, but poor scalability and infeasible local optima
- Decoupled methods: better scalability, no infeasibility issues, but poorly conditioned with small step sizes; cannot jointly constrain $\mu$ and $\Sigma$

**Discontinuities in Sensing**

Sensing discontinuities (field of view boundaries, occlusions) create zero-gradient regions that trap local optimizers. Solution: model the signed distance to sensing region boundaries (computed via GJK/EPA), encode as a binary sensing variable, and use homotopy methods that successively increase the sharpness of the transition.

When no measurement is obtained, the belief update should truncate the Gaussian — without this correction, the filter's uncertainty estimate becomes unreliable.

**Collision Avoidance with Uncertainty**

**Sigma hulls**: Convex hull of a robot link transformed through sigma points on the uncertainty contour. The signed distance between the obstacle and sigma hulls enforces probabilistic collision avoidance.

**Continuous collision avoidance**: Use convex hull of sigma hulls between consecutive time steps to prevent collisions between discretized waypoints.

**Belief Space MPC**: Update belief from actual observations during execution and replan after each belief update. Provides effective feedback control.

**Separation Principle**

For linear-Gaussian systems:

$x_{t+1} = A x_t + B u_t + w_t, \quad w_t \sim \mathcal{N}(0, Q_t)$
$z_t = C x_t + v_t, \quad v_t \sim \mathcal{N}(0, R_t)$

Goal: minimize $\mathbb{E}\left[\sum_t (u_t^T U_t u_t + x_t^T X_t x_t)\right]$

The optimal control policy consists of:
1. Run LQR offline to find feedback matrices $K_1, K_2, \ldots$ for the fully observed case
2. Online: run Kalman filter to estimate state, apply $u_t = K_t \mu_{t \mid 0:t}$

Estimation and control decouple — the controller gains are computed as if the state were perfectly known.

**Key applications / classic examples**

- Tiger problem (classic POMDP illustration)
- Grasping with occlusions (belief space plan moves camera to reduce uncertainty)
- Active SLAM — exploring unknown environments while building maps
- Probabilistic collision avoidance for articulated robots
