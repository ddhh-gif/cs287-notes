**Off-Policy Model-Free RL**

*Q-learning, DQN, and continuous-action extensions: leveraging dynamic programming structure for sample-efficient off-policy learning from DQN through DDPG to Soft Actor-Critic.*

**Motivation**

TRPO and PPO use importance sampling to go beyond a single gradient step, but updates remain local. Off-policy methods leverage the dynamic programming structure, breaking the problem into 1-step pieces via Bellman backups. This enables more sample reuse, making learning more data-efficient.

The trade-off: Q-learning can be less stable than policy gradients, and the data may not always support learning about the optimal policy (rather than improving the current policy).

**(Tabular) Q-Learning**

Q-value iteration update:

$Q_{k+1}(s, a) = \mathbb{E}_{s' \sim P(\cdot \mid s, a)}\left[R(s, a, s') + \gamma \max_{a'} Q_k(s', a')\right]$

Replace the expectation with samples: for an observed transition $(s, a, r, s')$, compute $\text{target} = r + \gamma \max_{a'} Q_k(s', a')$ and blend into a running average:

$Q_{k+1}(s, a) \leftarrow (1 - \alpha) Q_k(s, a) + \alpha \cdot \text{target}$

**Action selection** ($\epsilon$-greedy): With probability $\epsilon$, choose random action; otherwise choose $\operatorname{argmax}_a Q_k(s, a)$. Balances exploration with exploitation.

**Convergence**: Q-learning converges to the optimal policy even when acting suboptimally (off-policy learning), provided all state-action pairs are visited infinitely often and the learning rate schedule satisfies $\sum_t \alpha_t(s,a) = \infty$, $\sum_t \alpha_t^2(s,a) < \infty$.

**Deep Q-Networks (DQN)**

Tabular methods don't scale: Atari has $\sim 10^{308}$ RAM states, humanoid has $\sim 10^{100}$ states. Solution: parameterize $Q_\theta(s, a)$ as a neural network.

**Approximate Q-learning update**: $\theta_{k+1} \leftarrow \theta_k - \alpha \nabla_\theta \frac{1}{2}(Q_\theta(s, a) - \text{target})^2$ where $\text{target} = r + \gamma \max_{a'} Q_{\theta_k}(s', a')$.

**DQN innovations**:
- Experience replay: store transitions in a replay buffer, sample random mini-batches for training (breaks correlation, enables reuse)
- Target network: use a lagged (frozen) copy of $Q$ for computing targets, increasing stability
- Variants: Double DQN (decouples action selection from evaluation), Prioritized Replay, Dueling DQN, Distributional DQN, Noisy DQN ("Rainbow" combines all)

**Soft Q-Learning (Continuous Actions)**

Q-learning with discrete actions uses $\max_{a'}$. For continuous actions, soft Q-learning replaces the hard max with a soft maximum that is differentiable:

$\text{target} = r + \gamma \cdot \text{softmax}_{a'} Q(s', a')$

The soft value is approximated by sampling actions from an implicit policy network trained via Stein variational gradient descent to match the Boltzmann distribution $\propto \exp(Q(s, a))$.

**Deep Deterministic Policy Gradient (DDPG)**

For continuous action spaces, DDPG maintains both a Q-function $Q_\phi(s, a)$ and a deterministic policy $\pi_\theta(s)$:

- Q-function update: $\phi \leftarrow \phi - \alpha_Q \nabla_\phi (Q_\phi(s_t, u_t) - \hat{Q}_t)^2$ with $\hat{Q}_t = r_t + \gamma Q_{\phi'}(s_{t+1}, \pi_{\theta'}(s_{t+1}))$
- Policy update: $\theta \leftarrow \theta + \alpha_\pi \nabla_\theta Q_\phi(s_t, \pi_\theta(s_t))$ — backprop through Q
- Exploration: add noise to policy outputs
- Replay buffer for off-policy learning
- Lagged (Polyak-averaged) target networks $\phi', \theta'$ for stability

Very sample-efficient due to off-policy updates, but often unstable.

**Soft Actor-Critic (SAC)**

Adds an entropy bonus to the objective to encourage exploration and prevent the policy from overfitting to Q-function quirks:

$\max_\pi \mathbb{E}\left[\sum_t r_t + \alpha \mathcal{H}(\pi(\cdot \mid s_t))\right]$

**Soft policy iteration**:
1. Soft policy evaluation: $Q^{\pi}(s, a) \leftarrow r + \gamma \mathbb{E}_{s'}[V^{\pi}(s')]$ with $V^{\pi}(s) = \mathbb{E}_{a \sim \pi}[Q^{\pi}(s, a) - \alpha \log \pi(a \mid s)]$
2. Soft policy improvement: $\pi_{\text{new}} = \operatorname{argmin}_\pi \text{KL}(\pi(\cdot \mid s) \| \exp(Q^{\pi_{\text{old}}}(s, \cdot) / \alpha) / Z)$

In practice, SAC takes single stochastic gradient steps for both updates, alternating with environment interaction. Maintains a replay buffer, learns $V, Q$, and a stochastic policy $\pi$.

**Key applications / classic examples**

- Crawler bot (tabular Q-learning demo: arm angle + hand angle states, discrete actions, speed reward)
- Atari games (DQN achieves superhuman performance on many titles)
- Simulated 2D/3D robotics tasks (DDPG, with pixel inputs for driving)
- Real robot manipulation tasks (SAC learns directly on hardware with off-policy data)
