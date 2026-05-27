**Kalman Smoother, MAP, ML, and EM**

*Smoothing for full-trajectory inference, maximum a posteriori sequence estimation, maximum likelihood parameter learning, and expectation maximization for partially observed systems.*

**Kalman Smoothing**

Filtering computes $p(x_t \mid z_0, \ldots, z_t)$; smoothing computes $p(x_t \mid z_0, \ldots, z_T)$ using all observations (past and future). Smoothing refines state estimates with hindsight.

**Forward-backward algorithm**:
- Forward pass = standard Kalman filter, computing $\alpha_t(x_t) = p(x_t, z_0, \ldots, z_t)$
- Backward pass: recursively compute $\beta_t(x_t) = p(z_{t+1}, \ldots, z_T \mid x_t)$
- Combined: $p(x_t \mid z_0, \ldots, z_T) \propto \alpha_t(x_t) \beta_t(x_t)$

One forward + backward pass yields the smoothed marginal for all time steps. Renormalize at the end.

**Pairwise posterior**: The joint $p(x_t, x_{t+1} \mid z_{0:T})$ can be expressed as:

$p(x_t, x_{t+1}, z_0, \ldots, z_T) = \alpha_t(x_t) \cdot p(x_{t+1} \mid x_t) \cdot p(z_{t+1} \mid x_{t+1}) \cdot \beta_{t+1}(x_{t+1})$

**Kalman smoother** = smoother for the linear-Gaussian case:
- Forward pass = Kalman filtering (already known)
- Backward pass and combination follow the same recursive structure
- Yields Gaussian distributions at each $t$ for the smoothed state

**Maximum A Posteriori Sequence**

MAP finds the most likely sequence $x_{0:T}^* = \operatorname{argmax}_{x_{0:T}} p(x_{0:T} \mid z_{0:T})$. Naive enumeration over all possible sequences is exponential in $T$.

- **General case**: Dynamic programming gives an $O(T n^2)$ solution
- **Linear-Gaussian case**: The joint conditional $p(x_{0:T} \mid z_{0:T})$ is a multivariate Gaussian, so its most likely instantiation equals its mean. The marginal conditional means are exactly the Kalman smoother outputs. Equivalent to solving a convex optimization problem.

**Maximum Likelihood**

Given data, ML finds parameters $\theta$ that maximize the likelihood $p(\text{data} \mid \theta)$.

**Binary variables** (thumbtack example): If $\theta = P(\text{up})$ and we observe $n_1$ ups and $n_0$ downs:

$L(\theta) = \theta^{n_1}(1-\theta)^{n_0}$
$\theta_{ML} = \frac{n_1}{n_0 + n_1}$

The log-likelihood $\log L(\theta)$ is often easier to optimize and preserves the argmax. For the thumbtack, log-likelihood is concave (easy to maximize) while the raw likelihood is non-concave.

**Multivariate cases**:
- Multinomial: ML estimates are empirical counts
- Gaussian: $\mu_{ML} = \frac{1}{m}\sum_i x^{(i)}$, $\Sigma_{ML} = \frac{1}{m}\sum_i (x^{(i)} - \mu)(x^{(i)} - \mu)^T$
- Conditional Gaussian: analogous closed-form solutions
- Fully observed linear-Gaussian HMM: two separate ML problems, one for dynamics $p(x_{t+1} \mid x_t, u_t)$ and one for observations $p(z_t \mid x_t)$

**Maximum A Posteriori Parameters**

MAP incorporates a prior $p(\theta)$:

$\theta_{MAP} = \operatorname{argmax}_\theta p(\theta \mid \text{data}) = \operatorname{argmax}_\theta p(\text{data} \mid \theta) p(\theta)$

**Beta prior**: For a binary variable with Beta($\alpha, \beta$) prior, the MAP estimate adds pseudocounts: effectively $\alpha-1$ extra ups and $\beta-1$ extra downs. The Laplace estimate corresponds to adding 1 to each count.

**Dirichlet prior**: Generalizes Beta to multinomial variables; MAP corresponds to adding pseudocounts for each outcome.

**Conditional Gaussian MAP**: With Gaussian priors on parameters, MAP estimation becomes regularized regression. The prior's influence can be tuned via cross-validation (training/validation split, typically 70/30 or 10-fold).

**Expectation Maximization (EM)**

EM solves ML problems with unobserved variables. Given observed data $z$ and latent variables $x$, with model parameters $\mu$:

- **E-step**: Compute $q^{(t)}(x) = p(x \mid z, \mu^{(t)})$ — the posterior over latents given current parameters
- **M-step**: $\mu^{(t+1)} = \operatorname{argmax}_\mu \sum_x q^{(t)}(x) \log p(z, x \mid \mu)$

The M-step objective lower-bounds the true log-likelihood and is tight at $\mu^{(t)}$ (via Jensen's inequality). The true objective increases by at least as much as the M-step objective.

**EM for mixture of Gaussians**:
- E-step: Compute soft assignments $p(x = k \mid z^{(i)})$ for each component $k$
- M-step: Update $\mu_k, \Sigma_k, \theta_k$ using soft counts from E-step

**EM for HMMs**:
- E-step: Run smoother to compute $p(x_t \mid z_{0:T})$ and $p(x_t, x_{t+1} \mid z_{0:T})$
- M-step: Update parameters from soft transition and emission counts

**EM for linear-Gaussian systems**:
- E-step: Kalman smoother (forward + backward pass)
- M-step: Closed-form updates for $A, B, C, Q, R$ from soft counts
- Track log-likelihood; it should increase monotonically

**EM for EKF setting**: Since linearization is approximate, parameter updates may decrease log-likelihood. Solution: interpolate between old and new parameters via line search to maximize log-likelihood.

**Key applications / classic examples**

- Speech recognition (HMMs with EM training)
- Time series denoising and interpolation via Kalman smoothing
- System identification from trajectory data
- Clustering via mixture of Gaussians
