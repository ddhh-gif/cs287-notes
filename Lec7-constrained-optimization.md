**Constrained Optimization**

*Penalty methods transform constrained problems into unconstrained ones. Convex programs admit efficient solutions via elimination, Newton's method with equality constraints, and barrier methods for inequalities. Dual descent provides an alternative with per-constraint step sizes.*

**Constrained Optimization Problem**

$\min_x g_0(x) \quad \text{s.t.} \quad g_i(x) \le 0 \; \forall i, \quad h_j(x) = 0 \; \forall j$

**Penalty Method**

Transform to unconstrained form:

$\min_x g_0(x) + \mu \sum_i |g_i(x)|_+ + \mu \sum_j |h_j(x)|$

where $|\cdot|_+ = \max(0, \cdot)$.

- Now unconstrained; same solution for $\mu$ large enough
- $f_\mu(x) = g_0(x) + \mu \sum_i |g_i(x)|_+ + \mu \sum_j |h_j(x)|$ is the merit function
- Inner loop: optimize $f_\mu(x)$ using gradient descent, Newton/quasi-Newton, or trust region method
- Outer loop: increase $\mu$ until constraint violations vanish

**Penalty Method with Trust Region Inner Loop**

At current point $\bar{x}$, solve:

$\min_x g_0(\bar{x}) + \nabla g_0(\bar{x})^\top (x - \bar{x}) + \mu \sum_i |g_i(\bar{x}) + \nabla g_i(\bar{x})^\top (x - \bar{x})|_+ + \mu \sum_j |h_j(\bar{x}) + \nabla h_j(\bar{x})^\top (x - \bar{x})|$
$\text{s.t.} \quad \|x - \bar{x}\|_2 \le \varepsilon$

Algorithm:

- While $\sum_i |g_i(\bar{x})|_+ + \sum_j |h_j(\bar{x})| \ge \delta$ and $\mu < \mu_{\max}$:
  - Increase $\mu := t \mu$ (with $t > 1$), reset trust region $\varepsilon := \varepsilon_0$
  - Inner loop: solve trust region subproblem (convex program)
    - If improvement sufficient: accept $x \leftarrow x_{\text{next}}$, grow trust region $\varepsilon := \varepsilon / \beta$, break
    - Else: shrink trust region $\varepsilon := \beta \varepsilon$
    - If $\varepsilon$ below threshold: break inner loop

**Tweak: Retain Convex Terms Exactly**

For problems with separable convex and non-convex parts:

$\min_x f_0(x) + g_0(x) \quad \text{s.t.} \quad f_i(x) \le 0 \; \forall i, \quad Ax - b = 0, \quad g_k(x) \le 0 \; \forall k, \quad h_l(x) = 0 \; \forall l$

where $f_i$ are convex, $g_k$ non-convex, $h_l$ nonlinear. Reformulate:

$\min_x f_0(x) + g_0(x) + \mu \sum_k |g_k(x)|_+ + \mu \sum_l |h_l(x)|$
$\text{s.t.} \quad f_i(x) \le 0 \; \forall i, \quad Ax - b = 0$

Convex parts remain as constraints (handled by solver), non-convex parts enter the penalty.

**Convex Problems: Equality Constrained Minimization**

$\min_x f(x) \quad \text{s.t.} \quad Ax = b$

Three solution methods:

**Method 1: Elimination**

- Find $F$ spanning the nullspace of $A$ (via SVD: $A = U S V^\top$, set $F = V(:, k+1{:}\text{end})$ where $k$ is rank)
- Any solution to $Ax = b$ can be written as $x = \bar{x} + Fz$ for some $z$
- Solve unconstrained problem $\min_z f(\bar{x} + Fz)$
- Cons: need to find $\bar{x}$ and $F$; may destroy problem sparsity

**Method 2: Newton's Method (Feasible)**

Optimality condition for $x^*$ with $Ax^* = b$:

$\nabla f(x^*) + A^\top \nu^* = 0, \quad Ax^* = b$

- Using 2nd order approximation at feasible $x$:
  $\min_{\Delta x} f(x) + \nabla f(x)^\top \Delta x + \frac{1}{2} \Delta x^\top \nabla^2 f(x) \Delta x \quad \text{s.t.} \quad A \Delta x = 0$
- Newton step $\Delta x_{\text{nt}}$ obtained by solving:
  $\begin{bmatrix} \nabla^2 f(x) & A^\top \\ A & 0 \end{bmatrix} \begin{bmatrix} \Delta x_{\text{nt}} \\ \nu_{\text{nt}} \end{bmatrix} = \begin{bmatrix} -\nabla f(x) \\ 0 \end{bmatrix}$

**Method 3: Infeasible Start Newton Method**

- Does not require initial $x$ to satisfy $Ax = b$
- Linearize optimality conditions at current $(x, \nu)$:
  $\begin{bmatrix} \nabla^2 f(x) & A^\top \\ A & 0 \end{bmatrix} \begin{bmatrix} \Delta x_{\text{nt}} \\ \Delta \nu_{\text{nt}} \end{bmatrix} = -\begin{bmatrix} \nabla f(x) + A^\top \nu \\ Ax - b \end{bmatrix}$

**Convex Problems: Inequality Constrained — Barrier Method**

Problem: $\min_x f(x)$ s.t. $f_i(x) \le 0 \; \forall i$, $Ax = b$

Reformulation via indicator function:

$\min_x f(x) + \sum_i I_-(f_i(x)) \quad \text{s.t.} \quad Ax = b$

where $I_-(u) = 0$ if $u \le 0$, $I_-(u) = \infty$ if $u > 0$. No inequality constraints remain, but objective is poorly conditioned.

Logarithmic barrier approximation: $I_-(u) \approx -(1/t) \log(-u)$, $t > 0$

- For $t > 0$, $-(1/t)\log(-u)$ is a smooth approximation of $I_-(u)$
- Approximation improves as $t \to \infty$
- Better conditioned for smaller $t$

Barrier method algorithm:

- Given: strictly feasible $x$, $t = t^{(0)} > 0$, $\mu > 1$, tolerance $\varepsilon > 0$
- Repeat:
  1. **Centering step**: compute $x^*(t)$ by solving $\min_x t f(x) - \sum_i \log(-f_i(x))$ s.t. $Ax = b$, starting from $x$
  2. Update $x := x^*(t)$
  3. Stop if $m / t < \varepsilon$ (where $m$ is number of inequalities)
  4. Increase $t := \mu t$

**Initialization (Phase I)**

Basic phase I method:

- Solve $\min_{x,s} s$ s.t. $f_i(x) \le s \; \forall i$, $Ax = b$ with initial $s = \max_i f_i(x)$
- Stop early when $s < 0$ (feasible point found)

Sum of infeasibilities phase I:

- Solve $\min_{x,s} \sum_i s_i$ s.t. $f_i(x) \le s_i$, $s_i \ge 0 \; \forall i$, $Ax = b$ with $s_i = \max(0, f_i(x))$
- For infeasible problems, produces a solution satisfying many more inequalities than basic phase I

**Dual Descent**

Original problem: $\min_x g_0(x)$ s.t. $g_i(x) \le 0$, $h_j(x) = 0$

Penalty method iterates: optimize over $x$, increase $\mu$ as needed

Dual descent formulation introduces Lagrange multipliers $\lambda_i \ge 0$, $\nu_j$:

- Form Lagrangian: $L(x, \lambda, \nu) = g_0(x) + \sum_i \lambda_i g_i(x) + \sum_j \nu_j h_j(x)$
- Dual descent iterates:
  - Optimize over $x$: $x \leftarrow \operatorname{argmin}_x L(x, \lambda, \nu)$
  - Gradient ascent on multipliers: $\lambda_i \leftarrow \lambda_i + \alpha \, g_i(x)$, $\nu_j \leftarrow \nu_j + \alpha \, h_j(x)$
- Individual additive updates to $\lambda$ and $\nu$, rather than scaling a single $\mu$

**Key Applications**

- Trajectory optimization with collision constraints
- Motion planning with joint limits and obstacle avoidance
- Control input saturation handled via bounded constraints$
$