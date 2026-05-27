**Function Approximation for MDPs**

*Value iteration, policy iteration, and linear programming generalize to large state spaces via function approximation. Supervised learning fits a parametric representation after each Bellman backup. Non-expansion approximators provide convergence guarantees; approximate policy evaluation under weighted regression is a contraction.*

**Function Approximation**

- Main idea: use $\hat{V}_\theta(s) \approx V^*(s)$ with parameter $\theta \in \Theta$
- Representation size: $|\Theta| \ll |S|$ parameters instead of one value per state
- Advantage: fewer parameters to estimate
- Disadvantage: less expressiveness — typically there exist many $V^*$ for which no $\theta$ satisfies $\hat{V}_\theta = V^*$

**Supervised Learning Approach**

- Given: set of examples $(s^{(1)}, V(s^{(1)})), \ldots, (s^{(m)}, V(s^{(m)}))$
- Find best $\theta$ through least squares: $\min_{\theta \in \Theta} \sum_{i=1}^m (\hat{V}_\theta(s^{(i)}) - V(s^{(i)}))^2$

**Function Approximation Examples**

Tetris (~$2^{200}$ states):

- 22 basis functions $\phi_i(s)$ mapping state to features
- Ten basis functions: height $h[k]$ of each of the 10 columns
- Nine basis functions: $|h[k+1] - h[k]|$, absolute differences between successive columns
- One basis function: $\max_k h[k]$, maximum column height
- One basis function: number of holes in the board
- One constant basis function equal to 1
- $\hat{V}_\theta(s) = \sum_{i=0}^{21} \theta_i \phi_i(s) = \theta^\top \phi(s)$

Pacman:

- $V(s) = \theta_0 + \theta_1 \cdot (\text{distance to closest ghost}) + \theta_2 \cdot (\text{distance to closest power pellet}) + \theta_3 \cdot (\text{in dead-end}) + \theta_4 \cdot (\text{closer to power pellet than ghost}) + \ldots$
- Linear in features: $\hat{V}_\theta(s) = \theta^\top \phi(s)$

Nearest neighbor (0th order):

- Store values $\theta_1, \ldots, \theta_k$ at anchor points $x_1, \ldots, x_k$
- $\hat{V}(s) = \hat{V}(x_{\text{nearest}}) = \theta_{\text{nearest}}$
- $\hat{V}(s) = \theta^\top \phi(s)$ with $\phi(s)$ one-hot at the nearest anchor point

$k$-nearest neighbor interpolation (1st order):

- Assign interpolated value from nearest $k$ anchor points
- $\hat{V}(s) = \phi_1(s)\theta_1 + \phi_2(s)\theta_2 + \phi_5(s)\theta_5 + \phi_6(s)\theta_6$ (weights sum to 1)

Other examples: polynomials ($S = \mathbb{R}$, $\hat{V}_\theta(s) = \sum_{i=0}^n \theta_i s^i$), neural nets

**Neural Network Background**

- Single artificial neuron: $y = g(w^\top x + b)$ where $g$ is an activation function (sigmoid, ReLU, tanh, etc.)
- Neural network: composition of layers; choice of weights $w$ determines the function from $x \to y$
- Universal function approximation theorem: given any continuous function $f(x)$, a 2-layer neural network with enough hidden units can closely approximate $f(x)$ [Cybenko, 1989; Hornik, 1991]
- Overfitting: degree-15 polynomial fits training data but generalizes poorly
- Mitigation: reduce number of features, regularize, early stopping on hold-out data

**Value Iteration with Function Approximation**

Algorithm:

- Initialize $\theta^{(0)}$
- For $i = 0, 1, \ldots, H$:
  - Step 0: pick subset $S' \subseteq S$ with $|S'| \ll |S|$ (typically sampled from a trajectory or random)
  - Step 1 (Bellman backups): for all $s \in S'$, compute
    $\bar{V}_{i+1}(s) = \max_a \sum_{s'} T(s,a,s') \left[ R(s,a,s') + \gamma \hat{V}_{\theta^{(i)}}(s') \right]$
  - Step 2 (supervised learning): find $\theta^{(i+1)}$ as the solution to
    $\min_\theta \sum_{s \in S'} \left( \hat{V}_\theta(s) - \bar{V}_{i+1}(s) \right)^2$

With neural nets: only a small number of gradient updates or early stopping based on a hold-out set to avoid overfitting

**Value Iteration w/Function Approximation — Mini-Tetris Example**

- Two types of blocks, translation only (no rotation); $S'$ has 4 representative states
- 10 basis functions for a 4-column board (4 heights, 3 differences, max height, holes, constant)
- Initial $\theta^{(0)} = (-1, -1, -1, -1, -2, -2, -2, -3, -2, 10)$
- Bellman backups compute $\bar{V}$ values (e.g., 6.4, 19, 19, -29.6) for the 4 states
- Least squares fit yields updated $\theta^{(1)} = (0.195, 6.24, -2.11, 0, -6.05, 0.13, -2.11, 2.13, 0, 1.59)$

**Theoretical Analysis: Contraction Guarantees**

- Definition: operator $G$ is a non-expansion w.r.t. a norm $\|\cdot\|$ if $\|G(U) - G(V)\| \le \|U - V\|$
- Fact: if $F$ is a $\gamma$-contraction and $G$ is a non-expansion w.r.t. the same norm, then $G \circ F$ is a $\gamma$-contraction
- Corollary: if the supervised learning step is a non-expansion, value iteration with function approximation converges to a unique fixed point
- Averager function approximators are non-expansions: nearest neighbor (state aggregation), linear interpolation over triangles/tetrahedrons
- If we use a non-expansion approximator that can approximate $V^*$ well, we obtain a good value function estimate

**Policy Iteration with Function Approximation**

- Insert function approximation into policy evaluation step
- IF weighted linear regression is used, weighted by state visitation frequencies under the current policy
- THEN the resulting projection is a contraction w.r.t. the weighted 2-norm
- Policy evaluation Bellman update is also a contraction w.r.t. the same norm
- Result: guaranteed convergence

**Linear Programming with Function Approximation**

- Approximate LP using $\hat{V}_\theta(s) = \theta^\top \phi(s)$ with subset $S'$ rather than full $S$:
  $\min_\theta \sum_{s \in S'} \mu_0(s) \phi^\top(s) \theta \quad \text{s.t.} \quad \phi^\top(s) \theta \ge \sum_{s'} T(s,a,s') \left[ R(s,a,s') + \gamma \phi^\top(s') \theta \right], \quad \forall s \in S', a \in A$
- LP solver will converge
- Solution quality [de Farias and Van Roy, 2002]: assuming one feature is equal to 1 for all states and $S' = S$:
  $\|V^* - \Phi \theta\|_{1,\mu_0} \le \frac{2}{1-\gamma} \min_\theta \|V^* - \Phi \theta\|_\infty$
- Slightly weaker probabilistic guarantees hold for $S' \neq S$ with $|S'|$ growing with the number of features

**Key Applications**

- Tetris with linear function approximation over 22 hand-crafted features
- Pacman with distance-based features
- Large-scale RL with neural network value function approximators (DQN and variants)$
$