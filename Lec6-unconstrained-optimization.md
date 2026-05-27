**Unconstrained Optimization**

*Gradient descent, Newton's method, and natural gradient provide increasingly sophisticated approaches to unconstrained minimization. Momentum, RMSprop, and Adam extend these for scalable deep learning. Convexity guarantees unique global minima.*

**Optimal Control as Optimization**

- Goal: find sequence of control inputs (and states) solving $\min_{x_t, u_t} \sum_t g_t(x_t, u_t)$ subject to $x_{t+1} = f(x_t, u_t)$
- Generally hard. Exception: convex problems where $g$ is convex, constraint sets are convex, and $f$ is linear
- Iterative LQR is one approach but becomes tricky with control and state constraints
- $u$ could be parameters of a policy rather than raw control inputs

**Convex Optimization**

- A function $f$ is convex if $\forall x_1, x_2 \in \operatorname{Domain}(f), \forall \lambda \in [0, 1]$:
  $f(\lambda x_1 + (1-\lambda) x_2) \le \lambda f(x_1) + (1-\lambda) f(x_2)$
- A convex optimization problem has the form:
  $\min_{x \in \mathbb{R}^n} f_0(x) \quad \text{s.t.} \quad f_i(x) \le 0, \; i = 1, \ldots, n, \quad Ax = b$
  where all $f_i$ are convex
- Convex functions have a unique minimum; the sublevel set $\{x \mid f(x) \le a\}$ is convex

**Unconstrained Minimization**

- $x^*$ is a local minimum of differentiable $f$ if:
  1. $\nabla f(x^*) = 0$ (gradient vanishes)
  2. $\nabla^2 f(x^*) \succeq 0$ (Hessian positive semidefinite)
- In simple cases, solve $\nabla f(x) = 0$ directly for candidate minima and check condition (2)
- In general, numerical methods are needed

**Steepest Descent = Gradient Descent**

- Start at some point, repeat: take a step in the steepest descent direction
- The steepest descent direction is $-\nabla f(x)$ (unit ball constraint); hence gradient descent
- Algorithm:
  1. Initialize $x$
  2. Repeat:
     - Determine descent direction: $\Delta x = -\nabla f(x)$
     - Line search: choose step size $t > 0$
     - Update: $x := x + t \Delta x$
  3. Until stopping criterion satisfied

**Stepsize Selection**

- Exact line search: $t = \operatorname{argmin}_{s > 0} f(x + s \Delta x)$
  - Used when 1D minimization is cheap relative to computing the search direction
- Backtracking line search (inexact):
  - Start with $t = 1$, reduce $t := \beta t$ (with $\beta \in (0,1)$) until sufficient decrease: $f(x + t\Delta x) \le f(x) + \alpha t \nabla f(x)^\top \Delta x$ (Armijo condition, $\alpha \in (0, 0.5)$)

**Gradient Descent Convergence**

- For quadratic functions, convergence speed depends on condition number = $\lambda_{\max} / \lambda_{\min}$ of the Hessian
- In high dimensions, almost guaranteed to have a high (bad) condition number
- Rescaling coordinates changes the condition number

**Newton's Method**

- 2nd order Taylor approximation: $f(x + \Delta x) \approx f(x) + \nabla f(x)^\top \Delta x + \frac{1}{2} \Delta x^\top \nabla^2 f(x) \Delta x$
- Assuming $\nabla^2 f(x) \succ 0$ (true for convex $f$), the minimizer of the approximation is:
  $\Delta x_{\text{nt}} = -(\nabla^2 f(x))^{-1} \nabla f(x)$
- Newton step: $x := x + t \Delta x_{\text{nt}}$, where $t$ chosen by backtracking line search

**Affine Invariance**

- Under coordinate transformation $y = A^{-1}x$ ($x = Ay$):
  - Newton's method on $f(x)$ starting from $x^{(0)}$ gives $x^{(0)}, x^{(1)}, x^{(2)}, \ldots$
  - Newton's method on $g(y) = f(Ay)$ starting from $y^{(0)} = A^{-1} x^{(0)}$ gives $y^{(0)} = A^{-1} x^{(0)}, y^{(1)} = A^{-1} x^{(1)}, \ldots$
- Newton's method is invariant to affine reparameterizations

**Newton vs. Gradient Descent Performance**

- Gradient descent: zig-zags, many iterations required for ill-conditioned problems
- Newton's method: converges in one step if $f$ is convex quadratic; generally much fewer iterations
- Cost per iteration is higher ($O(n^3)$ for Hessian inverse vs. $O(n)$ for gradient descent)

**Quasi-Newton Methods**

- Approximate the Hessian to reduce computational cost
- Example 1: diagonal Hessian approximation (ignore off-diagonal entries)
- Example 2: Natural gradient / Gauss-Newton (next section)

**Natural Gradient**

- Consider maximum likelihood: $\max_\theta \sum_i \log p(x^{(i)}; \theta)$
- Gradient: $\nabla f(\theta) = \sum_i \nabla \log p(x^{(i)}; \theta)$
- Hessian: $\nabla^2 f(\theta) = \sum_i \frac{\nabla^2 p(x^{(i)}; \theta)}{p(x^{(i)}; \theta)} - (\nabla \log p(x^{(i)}; \theta))(\nabla \log p(x^{(i)}; \theta))^\top$
- Natural gradient: keeps only the 2nd (outer product) term in the Hessian
  $G = \sum_i (\nabla \log p(x^{(i)}; \theta))(\nabla \log p(x^{(i)}; \theta))^\top$
  $\Delta \theta_{\text{natural}} = -G^{-1} \nabla f(\theta)$
- Benefits: faster to compute (only gradients needed), guaranteed negative definite, invariant to any smooth reparameterization (stronger than Newton's affine invariance)

**Momentum, RMSprop, Adam**

Gradient descent with momentum:

- $v := \beta v + (1-\beta) \nabla f(x)$ (exponentially weighted average of gradients)
- $x := x - \alpha v$
- Typically $\beta = 0.9$

RMSprop (Root Mean Square propagation):

- $s := \beta s + (1-\beta) (\nabla f(x))^2$ (exponentially weighted avg of squared gradients)
- $x := x - \alpha \frac{\nabla f(x)}{\sqrt{s} + \varepsilon}$ (elementwise division)
- Typically $\beta = 0.999$

Adam (Adaptive momentum estimation):

- $v := \beta_1 v + (1-\beta_1) \nabla f(x)$ (momentum)
- $s := \beta_2 s + (1-\beta_2) (\nabla f(x))^2$ (RMSprop-like scaling)
- $x := x - \alpha \frac{v}{\sqrt{s} + \varepsilon}$
- Typically $\beta_1 = 0.9$, $\beta_2 = 0.999$, $\varepsilon = 10^{-8}$

**Key Applications**

- LQR cost function optimization (convex quadratic)
- Neural network training for visuomotor policies
- Trajectory optimization objective minimization$
$