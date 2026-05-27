**Policy Gradients**

*Stochastic gradient ascent on expected return: from the likelihood ratio trick through baseline subtraction and temporal decomposition to modern algorithms like A3C, GAE, TRPO, and PPO.*

**Policy Optimization Setup**

Consider a stochastic policy $\pi_\theta(u \mid s)$ parameterized by $\theta$. The goal is:

$\max_\theta \mathbb{E}\left[\sum_{t=0}^H R(s_t) \;\middle|\; \pi_\theta\right]$

Policy optimization directly optimizes the objective of interest, is compatible with rich architectures (including recurrence), but historically has been less sample-efficient than value-based methods.

**Why Policy Optimization**

- $\pi$ can be simpler than $Q$ or $V$: e.g., for robotic grasping, $V$ doesn't prescribe actions (would need a dynamics model) and $Q$ requires solving $\operatorname{argmax}_u Q(s, u)$ in continuous action spaces
- Optimizes what you care about directly, rather than indirectly through self-consistency

**Black-Box Methods**

**Finite differences**: Perturb parameters, evaluate return, estimate gradient. Simple but noisy; works best in low dimensions.

**Cross-Entropy Method (CEM)**: Sample $n$ parameter vectors $\theta_i \sim \mathcal{N}(\mu, \operatorname{diag}(\sigma^2))$, run one rollout each to get return $R(\tau_i)$, select top $k\%$, refit Gaussian. Simple, scalable, but ignores temporal structure.

**Likelihood Ratio Policy Gradient**

The gradient of expected return decomposes via the likelihood ratio trick:

$\nabla_\theta U(\theta) = \nabla_\theta \mathbb{E}_{\tau \sim \pi_\theta}[R(\tau)] = \mathbb{E}_{\tau \sim \pi_\theta}\left[R(\tau) \nabla_\theta \log P(\tau; \theta)\right]$

With Monte Carlo estimation from $m$ rollouts:

$\hat{g} = \frac{1}{m}\sum_{i=1}^m \nabla_\theta \log P(\tau^{(i)}; \theta) \, R(\tau^{(i)})$

Valid even when $R$ is discontinuous or derived from an unknown function. Intuition: increase probability of good paths, decrease probability of bad ones.

**Decomposing over time**: Since $P(\tau; \theta) = P(s_0) \prod_t \pi_\theta(u_t \mid s_t) P(s_{t+1} \mid s_t, u_t)$, the dynamics terms cancel in the gradient:

$\hat{g} = \frac{1}{m}\sum_{i=1}^m \sum_{t=0}^{H-1} \nabla_\theta \log \pi_\theta(u_t^{(i)} \mid s_t^{(i)}) \left(\sum_{k=0}^{H-1} R(s_k^{(i)}, u_k^{(i)}) \right)$

**Baseline Subtraction**

Adding a baseline $b$ maintains unbiasedness (since $\mathbb{E}[\nabla_\theta \log \pi_\theta \cdot b] = 0$) while reducing variance:

$\hat{g} = \frac{1}{m}\sum_{i=1}^m \sum_{t=0}^{H-1} \nabla_\theta \log \pi_\theta(u_t^{(i)} \mid s_t^{(i)}) \left(\sum_{k=0}^{H-1} R(s_k^{(i)}, u_k^{(i)}) - b\right)$

**Temporal decomposition**: Rewards before time $t$ don't depend on $u_t^{(i)}$, so they can be dropped. The baseline can be state-dependent:

$\hat{g} = \frac{1}{m}\sum_{i=1}^m \sum_{t=0}^{H-1} \nabla_\theta \log \pi_\theta(u_t^{(i)} \mid s_t^{(i)}) \left(\sum_{k=t}^{H-1} R(s_k^{(i)}, u_k^{(i)}) - b(s_t^{(i)}) \right)$

The best choice for $b(s_t)$ is the value function $V^\pi(s_t) = \mathbb{E}[\sum_{k=t}^{H-1} R_k \mid s_t]$, giving the advantage $A^\pi(s_t, u_t) = Q^\pi(s_t, u_t) - V^\pi(s_t)$.

**Value Function Estimation**

Learn $V^\pi_\phi(s)$ alongside the policy:
- **Monte Carlo**: Regress $V^\pi_\phi(s_t^{(i)})$ against empirical returns $\sum_{k=t}^{H-1} R_k^{(i)}$
- **Bootstrap (Fitted V)**: Regress against $r + \gamma V^\pi_{\phi_i}(s')$ with a regularization term $\lambda\|\phi - \phi_i\|^2$

**Advantage Estimation**

For the advantage term $Q^\pi(s_t, u_t) - V^\pi(s_t)$:
- Single roll-out estimate: high variance
- Discounted returns: $Q^{\pi,\gamma}(s, u) = \mathbb{E}[r_0 + \gamma r_1 + \gamma^2 r_2 + \ldots]$
- Function approximation: $r_0 + \gamma V^\pi(s_1)$ (1-step), $r_0 + \gamma r_1 + \gamma^2 V^\pi(s_2)$ (2-step), etc.

**GAE (Generalized Advantage Estimation)**: Exponentially weighted average ($\sim$ TD($\lambda$)) of $k$-step advantage estimates, with weighting $(1-\lambda), (1-\lambda)\lambda, (1-\lambda)\lambda^2, \ldots$

**A3C (Asynchronous Advantage Actor-Critic)**: Uses $n$-step advantage estimates with parallel actors. Actor updates policy via policy gradient; critic updates $V^\pi_\phi$ via bootstrap regression.

**Trust Region Policy Optimization (TRPO)**

Step size is critical in RL: a step too far in supervised learning is corrected by the next update, but in RL the next batch is collected under that bad policy, making recovery difficult.

TRPO constrains the policy update using KL divergence between path distributions:

$\max_{\delta\theta} \hat{g}^T \delta\theta \quad \text{s.t.} \quad \text{KL}(P(\tau; \theta) \| P(\tau; \theta + \delta\theta)) \le \epsilon$

The KL simplifies dramatically because the dynamics cancel: $\text{KL}(P(\tau; \theta) \| P(\tau; \theta + \delta\theta)) \approx \frac{1}{M}\sum_{(s,u) \sim \theta} \text{KL}(\pi_\theta(u \mid s) \| \pi_{\theta+\delta\theta}(u \mid s))$.

Second-order Taylor expansion gives $\text{KL} \approx \delta\theta^T F_\theta \delta\theta$, where $F_\theta$ is the Fisher information matrix, easily computed from policy gradients.

TRPO uses conjugate gradient to efficiently approximate the natural gradient direction without building/inverting $F_\theta$, plus a surrogate loss for higher-order approximation.

**TRPO surrogate loss**: $\max_\pi \mathbb{E}_{\pi_{\text{old}}}\left[\frac{\pi(a \mid s)}{\pi_{\text{old}}(a \mid s)} A^{\pi_{\text{old}}}(s, a)\right]$ subject to $\mathbb{E}_{\pi_{\text{old}}}[\text{KL}(\pi \| \pi_{\text{old}})] \le \epsilon$.

**Proximal Policy Optimization (PPO)**

TRPO's KL constraint is hard to enforce for complex architectures (dropout, shared parameters, etc.). PPO simplifies:

**PPO v1 (Dual Descent)**: Replace the hard KL constraint with a penalty: $\max_\theta \text{surrogate} - \beta \cdot \text{KL}$, then do dual descent on $\beta$.

**PPO v2 (Clipped Surrogate)**: With $r_t(\theta) = \frac{\pi_\theta(a_t \mid s_t)}{\pi_{\theta_{\text{old}}}(a_t \mid s_t)}$, optimize:

$\max_\theta \mathbb{E}_t\left[\min(r_t(\theta) A_t, \operatorname{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon) A_t)\right]$

The clipping removes the incentive to move $r_t$ too far from 1, achieving trust-region-like behavior with first-order optimizers (Adam, RMSProp).

**Key applications / classic examples**

- Atari games (TRPO competitive with DQN)
- Simulated locomotion (TRPO + GAE learns walking, running)
- Autonomous helicopter hovering via policy search
- AIBO robot walking (finite-difference policy search)
- Toddler robot learning to stand
- OpenAI Five (Dota 2, trained with PPO)
- OpenAI Rubik's Cube hand manipulation
