**Direct Collocation Methods for Trajectory and Policy Optimization**

*Direct collocation treats states and controls as joint decision variables with dynamics enforced as equality constraints. Inverse dynamics models, specialized numerical solvers, and contact handling extend collocation to rigid body systems. Policy learning via collocation produces feedback controllers from trajectory data.*

**Overview**

- Previously: locally optimal control (shooting vs. collocation), forward dynamics models, LQR and DDP
- Today: direct collocation in detail (open-loop and policies), inverse dynamics models, solution methods for collocation, optimization with contacts

**Optimal Control: Shooting vs. Collocation**

Shooting formulation:

- Decision variables: $u_0, u_1, \ldots, u_H$
- States obtained by forward simulation: $x_{t+1} = f(x_t, u_t)$
- Objective: $\min_u \sum_t c_t(x_t, u_t)$

Collocation formulation:

- Decision variables: $x_t$ and $u_t$ jointly for all $t$
- Dynamics enforced as equality constraints: $x_{t+1} = f(x_t, u_t)$
- Objective: $\min_{x,u} \sum_t c_t(x_t, u_t)$ s.t. $x_{t+1} = f(x_t, u_t)$

Both can return either open-loop controls or feedback policies (linear or neural net).

**Inverse Dynamics Models**

- Forward dynamics: predict $x_{t+1}$ given $x_t, u_t$
- Inverse dynamics: predict $u_t$ given $x_t, x_{t+1}$
- With collocation and inverse dynamics, the dynamics constraint $x_{t+1} = f(x_t, u_t)$ can be replaced by: $u_t = f^{-1}(x_t, x_{t+1})$
- Eliminates the control variables from the optimization (substitute $u_t$ directly)
- For many rigid body systems, inverse dynamics is available in closed form (e.g., recursive Newton-Euler algorithm)

**Numerical Optimization for Collocation**

- The collocation problem is a large-scale nonlinear constrained optimization
- Solution approaches:
  - Sequential Quadratic Programming (SQP): iteratively solve quadratic approximations of the Lagrangian subject to linearized constraints
  - Interior point methods (e.g., IPOPT): barrier method handling inequality constraints
  - Augmented Lagrangian methods
- Exploiting problem structure (banded sparsity from the temporal chain) is critical for efficiency

Connection to natural gradient (Lec 6): the Gauss-Newton approximation to the Hessian arises naturally in trajectory optimization when the cost has a sum-of-squares structure, similar to the maximum likelihood setting where the natural gradient keeps only the outer product term.

**Optimizing Dynamics with Contact**

Rigid body systems through contact [Posa and Tedrake, 2012]:

- Contact introduces non-smooth dynamics (complementarity constraints)
- Challenges: instantaneous velocity changes upon impact, stick-slip transitions
- Direct collocation handles contacts by treating contact forces as additional decision variables with complementarity constraints
- Linear Complementarity Problem (LCP) formulation: $\phi \ge 0$, $\lambda \ge 0$, $\phi \lambda = 0$ where $\phi$ is distance to contact and $\lambda$ is contact force
- Time-stepping approaches linearize the complementarity around the current trajectory

**Collocation Methods for Policy Learning**

- Collocation produces a trajectory; can distill into a feedback policy
- Policy learning via collocation:
  - Add policy parameters $\theta$ as decision variables
  - Policy constraint: $u_t = \pi_\theta(x_t)$
  - Optimize over both trajectory and policy parameters simultaneously
- Benefits: produces a closed-loop policy directly, can generalize to nearby initial conditions
- Can parameterize policy as a neural network and use collocation gradients to train it

**Key Applications**

- Bipedal walking with foot contacts (complementarity constraints)
- Robotic manipulation with finger-object contacts
- Full-body humanoid motion planning
- Quadruped locomotion over rough terrain
- Distillation of trajectory optimization results into neural network policies$
$