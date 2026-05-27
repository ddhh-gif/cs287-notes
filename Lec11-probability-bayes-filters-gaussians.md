**Probability Review, Bayes Filters, and Gaussians**

*Probability theory provides the mathematical framework for state estimation under uncertainty. Bayes filters recursively fuse noisy sensor measurements with stochastic action models. Gaussians, as the conjugate family for linear-Gaussian systems, enable closed-form filtering via linear algebra.*

**Why Probability in Robotics?**

- Robot state and environment are often unknown; only noisy sensors are available
- Probability provides a framework to fuse sensory information, yielding a distribution over possible states
- Dynamics are often stochastic: cannot optimize for a particular outcome, only for a good distribution over outcomes
- Probability provides a framework to reason in stochastic settings and find good control policies

**Example 1: Helicopter**

- State: position, orientation, velocity, angular rate
- Sensors: GPS (noisy position, sometimes velocity), inertial sensing unit (3-axis gyro, 3-axis accelerometer, 3-axis magnetometer)
- Dynamics noise: wind, unmodeled engine/servo/blade dynamics

**Example 2: Mobile Robot Indoors**

- State: position and heading
- Sensors: odometry (wheel encoders), laser range finder (time-of-flight for distance)
- Dynamics noise: wheel slippage, unmodeled floor variation

**Axioms of Probability Theory**

- $0 \le \Pr(A) \le 1$ for any event $A \subseteq \Omega$
- $\Pr(\Omega) = 1$, $\Pr(\emptyset) = 0$
- $\Pr(A \cup B) = \Pr(A) + \Pr(B) - \Pr(A \cap B)$
- Corollary: $\Pr(\Omega \setminus A) = 1 - \Pr(A)$

**Discrete Random Variables**

- $X$ takes values in $\{x_1, x_2, \ldots, x_n\}$
- $P(X = x_i)$ is the probability mass function
- $\sum_i P(x_i) = 1$

**Continuous Random Variables**

- $X$ takes values in the continuum
- $p(x)$ is the probability density function
- $\Pr(a \le X \le b) = \int_a^b p(x) \, dx$, $\int_{-\infty}^\infty p(x) \, dx = 1$

**Joint and Conditional Probability**

- Joint: $P(x, y) = P(X = x \text{ and } Y = y)$
- Independence: $X$ and $Y$ are independent iff $P(x, y) = P(x) P(y)$
- Conditional: $P(x \mid y) = P(x, y) / P(y)$, so $P(x, y) = P(x \mid y) P(y)$
- If $X$ and $Y$ are independent: $P(x \mid y) = P(x)$

**Law of Total Probability / Marginalization**

- Discrete: $P(x) = \sum_y P(x, y) = \sum_y P(x \mid y) P(y)$
- Continuous: $p(x) = \int p(x, y) \, dy = \int p(x \mid y) p(y) \, dy$

**Bayes Rule**

$P(x \mid y) = \frac{P(y \mid x) P(x)}{P(y)} = \frac{\text{likelihood} \times \text{prior}}{\text{evidence}}$

- $P(y \mid x)$: likelihood (causal knowledge, often easier to obtain)
- $P(x)$: prior
- $P(y) = \sum_x P(y \mid x) P(x)$: normalization constant

Normalization algorithm:

- $\forall x: \text{aux}_{x \mid y} = P(y \mid x) P(x)$
- $\eta = 1 / \sum_x \text{aux}_{x \mid y}$
- $\forall x: P(x \mid y) = \eta \, \text{aux}_{x \mid y}$

**Conditioning on Additional Variables**

Law of total probability with background $z$:
$P(x \mid z) = \int P(x, y \mid z) \, dy = \int P(x \mid y, z) P(y \mid z) \, dy$

Bayes rule with background $z$:
$P(x \mid y, z) = \frac{P(y \mid x, z) P(x \mid z)}{P(y \mid z)}$

**Conditional Independence**

$X$ and $Y$ are conditionally independent given $Z$ iff:
$P(x, y \mid z) = P(x \mid z) P(y \mid z)$
Equivalently: $P(x \mid y, z) = P(x \mid z)$ and $P(y \mid x, z) = P(y \mid z)$

**Recursive Bayesian Updating**

Markov assumption: measurement $z_n$ is independent of $z_1, \ldots, z_{n-1}$ given the state $x$:
$P(x \mid z_1, \ldots, z_n) = \frac{P(z_n \mid x) P(x \mid z_1, \ldots, z_{n-1})}{P(z_n \mid z_1, \ldots, z_{n-1})} = \eta \, P(z_n \mid x) P(x \mid z_1, \ldots, z_{n-1})$

Unrolling:
$P(x \mid z_1, \ldots, z_n) = \eta_{1\ldots n} \prod_{i=1}^n P(z_i \mid x) \, P(x)$

**Example: Door State Estimation**

- $P(z \mid \text{open}) = 0.6$, $P(z \mid \neg\text{open}) = 0.3$
- Prior: $P(\text{open}) = P(\neg\text{open}) = 0.5$
- After $z$: $P(\text{open} \mid z) = \frac{0.6 \times 0.5}{0.6 \times 0.5 + 0.3 \times 0.5} = \frac{2}{3} \approx 0.67$
- Second measurement $z_2$: $P(z_2 \mid \text{open}) = 0.5$, $P(z_2 \mid \neg\text{open}) = 0.6$
- After $z_1, z_2$: $P(\text{open} \mid z_1, z_2) = \frac{0.5 \times (2/3)}{0.5 \times (2/3) + 0.6 \times (1/3)} = 0.625$

**Caution: Independence Assumption**

If measurements are treated as independent when they are not, repeated Bayesian updates can lead to overconfidence. Convergence to the wrong answer can be rapid.

**Actions and State Transitions**

- The world changes due to robot actions, other agents, or time passing
- Actions are never carried out with absolute certainty
- Actions generally increase uncertainty (in contrast to measurements)
- Action model: $P(x' \mid u, x)$ — probability that executing $u$ in state $x$ results in new state $x'$

**Integrating the Outcome of Actions**

Discrete case: $P(x' \mid u) = \sum_x P(x' \mid u, x) P(x)$

Continuous case: $p(x' \mid u) = \int p(x' \mid u, x) \, p(x) \, dx$

**Bayes Filter Framework**

- Given: stream of observations $z_t$ and actions $u_t$, sensor model $P(z \mid x)$, action model $P(x' \mid u, x)$, prior $P(x)$
- Wanted: Belief $\operatorname{Bel}(x_t) = P(x_t \mid u_1, z_1, \ldots, u_t, z_t)$
- Markov assumptions:
  - $p(x_t \mid x_{1:t-1}, z_{1:t-1}, u_{1:t}) = p(x_t \mid x_{t-1}, u_t)$ (state is sufficient statistic)
  - $p(z_t \mid x_{0:t}, z_{1:t-1}, u_{1:t}) = p(z_t \mid x_t)$ (measurement depends only on current state)

**Bayes Filter Algorithm**

Given $\operatorname{Bel}(x_{t-1})$, action $u_t$, observation $z_t$:

1. **Prediction step** (motion update):
   $\overline{\operatorname{Bel}}(x_t) = \int P(x_t \mid u_t, x_{t-1}) \, \operatorname{Bel}(x_{t-1}) \, dx_{t-1}$

2. **Correction step** (measurement update):
   $\operatorname{Bel}(x_t) = \eta \, P(z_t \mid x_t) \, \overline{\operatorname{Bel}}(x_t)$

Return $\operatorname{Bel}(x_t)$.

**Bayes Filter Algorithm (Pseudocode)**

For each data item $d$:
- If $d$ is a perceptual data item $z$:
  1. For all $x$: $\operatorname{Bel}'(x) = P(z \mid x) \operatorname{Bel}(x)$
  2. $\eta = 1 / \sum_x \operatorname{Bel}'(x)$
  3. For all $x$: $\operatorname{Bel}(x) = \eta \operatorname{Bel}'(x)$
- Else if $d$ is an action data item $u$:
  1. For all $x'$: $\operatorname{Bel}'(x') = \int P(x' \mid u, x) \operatorname{Bel}(x) \, dx$

Return $\operatorname{Bel}(x)$

**Gaussians**

**Univariate Gaussian**

$p(x) = \frac{1}{\sqrt{2\pi}\sigma} \exp\left(-\frac{(x-\mu)^2}{2\sigma^2}\right)$

- Mean $\mu$, standard deviation $\sigma$, variance $\sigma^2$
- Integrates to 1
- Central Limit Theorem: sum of many independent random variables with finite variance is approximately Gaussian

**Multivariate Gaussian**

$p(x) = \frac{1}{(2\pi)^{d/2} |\Sigma|^{1/2}} \exp\left(-\frac{1}{2} (x - \mu)^\top \Sigma^{-1} (x - \mu)\right)$

- Mean vector $\mu \in \mathbb{R}^d$, covariance matrix $\Sigma \in \mathbb{R}^{d \times d}$, $\Sigma \succ 0$
- $\operatorname{E}[x] = \mu$
- $\operatorname{Cov}[x] = \operatorname{E}[(x-\mu)(x-\mu)^\top] = \Sigma$

Examples of 2D Gaussians:

- Spherical: $\Sigma = I$, contours are circles
- Diagonal but scaled: $\Sigma = \operatorname{diag}(\sigma_1^2, \sigma_2^2)$, contours are axis-aligned ellipses
- Correlated: $\Sigma = \begin{bmatrix} 1 & \rho \\ \rho & 1 \end{bmatrix}$, contours are rotated ellipses
- Negative correlation: $\Sigma = \begin{bmatrix} 1 & -\rho \\ -\rho & 1 \end{bmatrix}$
- Mixed scaling and correlation: $\Sigma = \begin{bmatrix} 3 & 0.8 \\ 0.8 & 1 \end{bmatrix}$

**Partitioned Multivariate Gaussian**

$\begin{bmatrix} X \\ Y \end{bmatrix} \sim \mathcal{N}\left( \begin{bmatrix} \mu_x \\ \mu_y \end{bmatrix}, \begin{bmatrix} \Sigma_{xx} & \Sigma_{xy} \\ \Sigma_{yx} & \Sigma_{yy} \end{bmatrix} \right)$

Precision matrix $\Gamma = \Sigma^{-1}$:
$\begin{bmatrix} \Gamma_{xx} & \Gamma_{xy} \\ \Gamma_{yx} & \Gamma_{yy} \end{bmatrix} = \begin{bmatrix} \Sigma_{xx} & \Sigma_{xy} \\ \Sigma_{yx} & \Sigma_{yy} \end{bmatrix}^{-1}$

Relationship between representations:
$\Sigma_{xx} = (\Gamma_{xx} - \Gamma_{xy} \Gamma_{yy}^{-1} \Gamma_{yx})^{-1}, \quad \Sigma_{xy} = -\Sigma_{xx} \Gamma_{xy} \Gamma_{yy}^{-1}$

**Marginalization**

$p(x) = \int p(x, y) \, dy \quad \Longrightarrow \quad X \sim \mathcal{N}(\mu_x, \Sigma_{xx})$

The marginal of a joint Gaussian is Gaussian with the corresponding block of the mean and covariance matrix.

**Conditioning**

Given $Y = y_0$:
$p(x \mid Y = y_0) = \mathcal{N}(\mu_{x \mid y}, \Sigma_{x \mid y})$

where

$\mu_{x \mid y} = \mu_x + \Sigma_{xy} \Sigma_{yy}^{-1} (y_0 - \mu_y)$
$\Sigma_{x \mid y} = \Sigma_{xx} - \Sigma_{xy} \Sigma_{yy}^{-1} \Sigma_{yx}$

Key observations:

- Conditional mean shifts according to the correlation and the measurement residual
- Conditional covariance is reduced (uncertainty decreases) and does NOT depend on the actual measurement $y_0$
- This is the mathematical foundation for the Kalman filter

**Key Applications**

- Robot localization: tracking position and heading using odometry and laser scans
- Helicopter state estimation: fusing GPS, IMU, and magnetometer data
- Kalman filtering for linear-Gaussian systems
- Extended/Unscented Kalman filters for nonlinear systems
- Particle filters for non-Gaussian, nonlinear state estimation$
$