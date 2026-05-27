**Markov Decision Processes and Exact Solution Methods**

*Value iteration, policy iteration, linear programming, and maximum entropy formulations provide exact solutions for discrete MDPs. These methods form the foundation for continuous and approximate extensions covered in later lectures.*

**Markov Decision Process $(S, A, T, R, \gamma, H)$**

- $S$: set of states
- $A$: set of actions
- $T_t(s,a,s') = P(s_{t+1} = s' \mid s_t = s, a_t = a)$ — transition function, $T: S \times A \times S \times \{0, 1, \ldots, H\} \to [0,1]$
- $R_t(s,a,s')$: reward for transition $(s, a, s')$
- $\gamma \in (0,1]$: discount factor
- $H$: horizon over which the agent acts
- Goal: find $\pi^*: S \times \{0, 1, \ldots, H\} \to A$ that maximizes expected sum of discounted rewards

**Examples of MDPs**

- Cleaning robot, walking robot, pole balancing
- Games: Tetris, backgammon
- Server management, shortest path problems
- Models for animal and human behavior

**Canonical Example: Grid World**

- Agent lives in a grid; walls block the agent's path
- Actions are stochastic: 80% intended direction, 10% each orthogonal direction
- If blocked by a wall, agent stays put
- Big terminal rewards at the end (e.g., +1 and -1)

**Solving MDPs**

- An optimal policy $\pi^*$ gives an action for each state at each time
- An optimal policy maximizes expected sum of discounted rewards
- Contrast with deterministic planning: in MDPs, policies give actions for all states, not just a sequence from start to goal
- Focus for now: discrete state-action spaces; continuous spaces covered next lecture

**Value Iteration**

Algorithm:

- Initialize $V_0(s) = 0$ for all $s$
- For $i = 1, \ldots, H$, for all states $s \in S$:
  $V_{i}(s) = \max_a \sum_{s'} T(s, a, s') \left[ R(s, a, s') + \gamma V_{i-1}(s') \right]$
- $V_i(s)$: expected sum of rewards starting from state $s$, acting optimally for $i$ steps
- $\pi_i(s) = \operatorname{argmax}_a \sum_{s'} T(s, a, s') \left[ R(s, a, s') + \gamma V_{i-1}(s') \right]$: optimal action for $i$ steps to go

**Value Iteration Convergence**

- Theorem: value iteration converges; at convergence, $V^*$ satisfies the Bellman equations
- Convergence intuition: additional reward collected beyond horizon $H$ goes to zero as $H \to \infty$ (bounded by $\frac{\gamma^{H+1}}{1-\gamma} R_{\max}$)
- Definition: max-norm $\|U\|_\infty = \max_s |U(s)|$
- Definition: an update operation is a $\gamma$-contraction in max-norm if $\|F(U) - F(V)\|_\infty \le \gamma \|U - V\|_\infty$ for all $U, V$
- Theorem: a contraction converges to a unique fixed point regardless of initialization
- Fact: the value iteration update is a $\gamma$-contraction in max-norm
- Corollary: value iteration converges to a unique fixed point
- Additionally: $\|V_i - V^*\|_\infty \le \frac{\|V_{i+1} - V_i\|_\infty}{1-\gamma}$ — once updates are small, the value function is close to optimal
- Infinite horizon optimal policy is stationary: same action at state $s$ at all times

**Policy Evaluation**

- Value iteration iterates: $V_{i+1}(s) = \max_a \sum_{s'} T(s, a, s') [R(s, a, s') + \gamma V_i(s')]$
- Policy evaluation for a fixed policy $\pi$: $V^{\pi}(s) = \sum_{s'} T(s, \pi(s), s') [R(s, \pi(s), s') + \gamma V^{\pi}(s')]$
- At convergence this gives $V^{\pi}$, the value of following policy $\pi$
- Can be solved as a linear system in variables $V^{\pi}(s)$ with constants $T$ and $R$

**Policy Iteration**

- Repeat until policy converges:
  1. Policy evaluation: compute $V^{\pi_k}$ (solve linear system)
  2. Policy improvement: $\pi_{k+1}(s) = \operatorname{argmax}_a \sum_{s'} T(s, a, s') [R(s, a, s') + \gamma V^{\pi_k}(s')]$
- Theorem: Policy iteration is guaranteed to converge. At convergence, the current policy and its value function are optimal.
- Guarantee: in every step the policy improves; a given policy can appear at most once. After at most $|A|^{|S|}$ iterations the algorithm must converge.
- At convergence: $V^{\pi_k}$ satisfies the Bellman equation, so $V^{\pi_k} = V^*$

**Linear Programming Formulation**

- At convergence of value iteration: $V^*(s) \ge \sum_{s'} T(s, a, s') [R(s, a, s') + \gamma V^*(s')]$ for all $s, a$
- LP formulation to find $V^*$:
  $\min_V \sum_{s \in S} \mu_0(s) V(s) \quad \text{s.t.} \quad V(s) \ge \sum_{s'} T(s,a,s') [R(s,a,s') + \gamma V(s')], \quad \forall s \in S, a \in A$
  where $\mu_0$ is a probability distribution over $S$ with $\mu_0(s) > 0$ for all $s$
- Theorem: $V^*$ is the solution to the above LP

**Dual Linear Program**

- Interpretation: dual variables $\lambda(s,a)$ represent state-action visitation frequencies
- Maximize expected discounted sum of rewards subject to flow conservation constraints
- Optimal policy: $\pi^*(s) = \operatorname{argmax}_a \lambda(s,a)$

**Maximum Entropy MDP**

Motivation: what if the optimal path becomes blocked? A distribution over solutions is more robust than a single deterministic policy.

**Entropy**

- Entropy = measure of uncertainty over random variable $X$
- Number of bits required to encode $X$ on average: $H(X) = -\sum_x P(x) \log P(x)$
- For binary random variable: max entropy at $P = 0.5$

**Max-ent Formulation**

- Regular: $\max_\pi \operatorname{E}[\sum_t \gamma^t R(s_t, a_t) \mid \pi]$
- Max-ent: $\max_\pi \operatorname{E}[\sum_t \gamma^t (R(s_t, a_t) + \alpha H(\pi(\cdot \mid s_t))) \mid \pi]$
- Trades off reward maximization with policy entropy; temperature parameter $T = 1/\alpha$ controls the trade-off

**Constrained Optimization Intermezzo**

- Original problem: $\min_x f(x)$ subject to $g(x) = c$
- Lagrangian: $L(x, \lambda) = f(x) + \lambda(g(x) - c)$
- At optimum: $\nabla_x L(x, \lambda) = 0$, $\nabla_\lambda L(x, \lambda) = 0$

**Max-ent for 1-step Problem**

- For a single time step, max-ent solution: $\pi(a \mid s) = \frac{\exp(Q(s,a)/T)}{\sum_{a'} \exp(Q(s,a')/T)}$ — softmax over Q-values

**Max-ent Value Iteration**

- $Q(s, a) = \sum_{s'} T(s, a, s') [R(s, a, s') + \gamma V(s')]$
- $V(s) = T \log \sum_a \exp(Q(s, a)/T)$ — softmax Bellman backup
- As $T \to 0$, recovers standard value iteration
- As $T \to \infty$, all actions become equally likely

**Course Trajectory: Optimal Control Approaches**

- Dynamic programming / Value iteration
- Discrete state spaces: exact methods
- Continuous state spaces: approximate solutions through discretization
- Large state spaces: approximate solutions through function approximation
- Linear systems: closed-form exact solution with LQR
- Nonlinear systems: extensions via local linearization, iLQR, differential dynamic programming
- Optimal control through nonlinear optimization: shooting vs. collocation formulations
- Model Predictive Control (MPC)$
$