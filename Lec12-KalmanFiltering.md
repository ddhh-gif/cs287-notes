**Kalman Filtering**

*Gaussian representations, recursive state estimation for linear-Gaussian systems, and nonlinear extensions via linearization and unscented transforms.*

**Multivariate Gaussians**

A multivariate Gaussian over $\mathbb{R}^n$ is fully specified by its mean vector $\mu$ and covariance matrix $\Sigma$. For a partitioned random vector $(X, Y)$:

$\mu = \begin{bmatrix} \mu_x \\ \mu_y \end{bmatrix}, \quad \Sigma = \begin{bmatrix} \Sigma_{xx} & \Sigma_{xy} \\ \Sigma_{yx} & \Sigma_{yy} \end{bmatrix}$

The precision matrix $\Gamma = \Sigma^{-1}$ gives an equivalent dual representation.

- **Marginalization**: If $p(x, y)$ is jointly Gaussian, then $p(x) = \int p(x, y) dy$ is Gaussian with mean $\mu_x$ and covariance $\Sigma_{xx}$
- **Conditioning**: $p(x \mid y = y_0)$ is Gaussian with mean $\mu_x + \Sigma_{xy}\Sigma_{yy}^{-1}(y_0 - \mu_y)$ and covariance $\Sigma_{xx} - \Sigma_{xy}\Sigma_{yy}^{-1}\Sigma_{yx}$. The conditional mean shifts according to correlation and measurement variance; the conditional covariance does not depend on $y_0$

**Kalman Filter**

The Kalman filter is a special case of the Bayes filter with linear-Gaussian dynamics and observation models:

$x_{t+1} = A_t x_t + B_t u_t + w_t, \quad w_t \sim \mathcal{N}(0, Q_t)$

$z_t = C_t x_t + v_t, \quad v_t \sim \mathcal{N}(0, R_t)$

**Time update**: Given belief $p(x_t) = \mathcal{N}(\mu_t, \Sigma_t)$, the joint distribution $p(x_t, x_{t+1})$ is Gaussian. Marginalizing over $x_t$:

$\bar{\mu}_{t+1} = A_t \mu_t + B_t u_t$

$\bar{\Sigma}_{t+1} = A_t \Sigma_t A_t^T + Q_t$

**Observation update**: After observing $z_{t+1}$, the posterior is obtained by conditioning the joint $p(x_{t+1}, z_{t+1})$:

$K_{t+1} = \bar{\Sigma}_{t+1} C_t^T (C_t \bar{\Sigma}_{t+1} C_t^T + R_t)^{-1}$

$\mu_{t+1} = \bar{\mu}_{t+1} + K_{t+1}(z_{t+1} - C_t \bar{\mu}_{t+1})$

$\Sigma_{t+1} = (I - K_{t+1} C_t) \bar{\Sigma}_{t+1}$

$K_{t+1}$ is the Kalman gain and $(z_{t+1} - C_t \bar{\mu}_{t+1})$ is the innovation.

**Complete algorithm**:
- Initialize: $p(x_0) = \mathcal{N}(\mu_0, \Sigma_0)$
- For $t = 1, 2, \ldots$: dynamics update then measurement update

**Properties**:
- Time complexity: $O(k^{2.376} + n^2)$ for measurement dimensionality $k$ and state dimensionality $n$
- Optimal for linear Gaussian systems
- If $A_t = A$, $C_t = C$, $Q_t = Q$, $R_t = R$ and the system is observable, covariances and Kalman gain converge to steady-state values. The system is observable iff $\text{rank}([C; CA; CA^2; \ldots; CA^{n-1}]) = n$

**Extended Kalman Filter (EKF)**

Most realistic robotic problems involve nonlinear dynamics and observation models:

$x_{t+1} = f(x_t, u_t) + w_t, \quad z_t = h(x_t) + v_t$

The EKF linearizes $f$ and $h$ via first-order Taylor expansion around the current mean $\mu_t$:

$A_t = \frac{\partial f}{\partial x}(\mu_t, u_t), \quad C_t = \frac{\partial h}{\partial x}(\bar{\mu}_t)$

Then applies the standard Kalman filter updates with these Jacobians.

- Same complexity as Kalman filter: $O(k^{2.376} + n^2)$
- Not optimal; can diverge if nonlinearities are large
- Works well in practice even when theoretical assumptions are violated
- Linearization accuracy depends on the variance of $p(x)$ relative to the region where the linearization is valid

**Unscented Kalman Filter (UKF)**

The UKF uses the exact nonlinear functions but approximates the distribution $p(x)$ with a minimal set of sigma points. EKF approximates $f$ to first order; UKF uses $f$ exactly with a sampled approximation of $p(x)$.

**Unscented transform**: Picks $2n+1$ sigma points that match the first, second, and third moments of a Gaussian with mean $\bar{x}$ and covariance $P_{xx}$:

$\mathcal{X}_0 = \bar{x}$
$\mathcal{X}_i = \bar{x} + \left(\sqrt{(n+\kappa)P_{xx}}\right)_i, \quad i = 1, \ldots, n$
$\mathcal{X}_{i+n} = \bar{x} - \left(\sqrt{(n+\kappa)P_{xx}}\right)_i, \quad i = 1, \ldots, n$

$\kappa$ is an extra degree of freedom; for Gaussian $x$, $n + \kappa = 3$ is a suggested heuristic. $L = \sqrt{P_{xx}}$ is any matrix satisfying $L L^T = P_{xx}$.

**Dynamics/observation updates**: Propagate sigma points through $f$, recompute mean and covariance from the transformed points. For the observation update, use sigma points to compute the cross-covariance between $x_t$ and $z_t$, then apply the standard Kalman update.

**Properties**:
- Same complexity as EKF, with a constant factor slowdown in practice
- Accurate in first two terms of Taylor expansion (EKF only gets first term), plus captures aspects of higher-order terms
- Derivative-free: no Jacobians required
- Still not optimal

**Additional topics (not covered in depth)**

- Square-root Kalman filter: tracks square root of covariance, equally fast, numerically more stable
- Sparse Information Filter: for very large systems with sparsity structure
- Ensemble Kalman Filter: for very large systems with low-rank structure
- Kalman filtering over SE(3)
- EM algorithm for estimating $A_t, B_t, C_t, Q_t, R_t$ from data

**Key applications / classic examples**

- GPS + IMU sensor fusion for robot localization
- Tracking moving objects from noisy measurements
- Spacecraft attitude estimation
- Financial time series filtering
