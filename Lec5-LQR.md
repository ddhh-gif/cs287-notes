**Optimal Control for Linear Dynamical Systems and Quadratic Cost (LQR)**

*LQR provides an exact closed-form solution to continuous state-space optimal control when dynamics are linear and cost is quadratic. Extensions to nonlinear systems, affine dynamics, stochasticity, and trajectory following make it a versatile foundation for robotics control.*

**Bellman's Curse of Dimensionality**

- $n$-dimensional state space: number of states grows exponentially in $n$ for fixed discretization levels
- Discretization computationally feasible only up to ~5–6 dimensions even with variable resolution and optimized implementations
- Function approximation may or may not work; in practice often somewhat local

**The LQR Setting**

- Dynamics: $x_{t+1} = A x_t + B u_t$ (linear)
- Cost: $\sum_t \left( x_t^\top Q x_t + u_t^\top R u_t \right)$ with $Q \succeq 0$, $R \succ 0$ (quadratic)
- Can solve continuous state-space optimal control exactly with only linear algebra operations
- Running time: $O(H n^3)$

**LQR Value Iteration**

- Back-up step for $i+1$ steps to go:
  $J_{i+1}(x) = \min_u \left[ x^\top Q x + u^\top R u + J_i(Ax + Bu) \right]$
- Assume $J_i(x) = x^\top P_i x$ (quadratic). Then:
  $J_{i+1}(x) = x^\top P_{i+1} x$
  where
  $P_{i+1} = Q + A^\top P_i A - A^\top P_i B (R + B^\top P_i B)^{-1} B^\top P_i A$
- Optimal policy: $u^*_t = -K_t x_t$ with $K_t = (R + B^\top P_t B)^{-1} B^\top P_t A$

$J_1(x)$ is quadratic, just like $J_0(x)$, so the value iteration update is the same for all times and can be done in closed form.

**Convergence to Infinite Horizon**

- Fact: guaranteed to converge to the infinite horizon optimal policy if and only if the dynamics $(A, B)$ is such that there exists a policy that can drive the state to zero (stabilizability)
- Often most convenient to use the steady-state $K$ for all times

**LQR Extensions**

**Ext 0: Affine Systems** — $x_{t+1} = A x_t + B u_t + c$

- Optimal control policy remains linear, optimal cost-to-go remains quadratic
- Redefine state as $z_t = [x_t; 1]$ to convert to standard LQR

**Ext 1: Stochastic Systems** — $x_{t+1} = A x_t + B u_t + w_t$ with $\operatorname{E}[w_t] = 0$

- Same optimal control policy as deterministic case
- Cost-to-go has one additional term depending on noise variance (not influenced by control inputs)

**Ext 2: Nonlinear Systems**

- For nonlinear system $x_{t+1} = f(x_t, u_t)$, linearize around equilibrium $(x^*, u^*)$:
  $x_{t+1} - x^* \approx A (x_t - x^*) + B (u_t - u^*)$
  where $A = \nabla_x f(x^*, u^*)$, $B = \nabla_u f(x^*, u^*)$
- Let $z_t = x_t - x^*$, $v_t = u_t - u^*$ to obtain standard LQR

**Ext 3: Penalize Change in Control Inputs**

- Standard LQR can generate high-frequency control inputs on real systems
- Augment state with past control input: $x'_t = [x_t; u_{t-1}]$, $u'_t = \Delta u_t = u_t - u_{t-1}$
- Cost penalizes both $x_t^\top Q x_t$ and $\Delta u_t^\top R \Delta u_t$

**Ext 4: Linear Time Varying (LTV) Systems** — $x_{t+1} = A_t x_t + B_t u_t$

- Same Riccati recursion applies with time-dependent $A_t, B_t, Q_t, R_t$

**Ext 5: Trajectory Following for Nonlinear Systems**

- Target trajectory: $x_0^*, x_1^*, \ldots, x_H^*$ is feasible if $\exists u_t^*$ s.t. $x_{t+1}^* = f(x_t^*, u_t^*)$
- Linearize around trajectory:
  $x_{t+1} - x_{t+1}^* \approx A_t (x_t - x_t^*) + B_t (u_t - u_t^*)$
  where $A_t = \nabla_x f(x_t^*, u_t^*)$, $B_t = \nabla_u f(x_t^*, u_t^*)$
- Transforms to LTV LQR; run standard back-up iterations
- Works even for infeasible target trajectories (offset term appears in dynamics)

**Iterative LQR (iLQR)**

For the general optimal control problem with nonlinear $f$ and non-quadratic cost $g$:

1. Initialize with a nominal trajectory $\{x_t^{(0)}, u_t^{(0)}\}$
2. Linearize $f$ and quadratize $g$ around current trajectory
3. Solve the resulting LQ problem
4. Update trajectory; repeat

**iLQR Convergence Considerations**

- May not converge as formulated: optimal LQ policy might diverge far from the linearization point
- Solution: add trust region term to cost:
  $\sum_t g(x_t, u_t) + \alpha \sum_t \left( \|x_t - x_t^{(i)}\|^2 + \|u_t - u_t^{(i)}\|^2 \right)$
- For $\alpha$ close to 1, the trust region term dominates, ensuring the new trajectory stays near the linearization point
- Practicalities: $f$ is nonlinear $\to$ non-convex problem, can get stuck in local optima; good initialization matters
- If $Q_t$ or $R_t$ are not positive definite, increase penalty for deviating from current trajectory until they become positive definite

**Differential Dynamic Programming (DDP)**

- Directly performs 2nd order Taylor expansion of the Bellman backup equation (rather than linearizing dynamics and 2nd order approximating cost separately)
- Retains a quadratic dynamics term discarded in iLQR
- DDP: $J_i(x_{t+1}) \approx J_i(\bar{x}_{t+1}) + J_i'(\bar{x}_{t+1})(x_{t+1} - \bar{x}_{t+1}) + \frac{1}{2} J_i''(\bar{x}_{t+1})(x_{t+1} - \bar{x}_{t+1})^2$
  with $x_{t+1} = f(x_t, u_t)$, expanding fully in $(u - \bar{u})$
- iLQR: expands $J_i(f(x_t, u_t))$ only through first-order dynamics approximation around $\bar{u}$
- Reference: Jacobson and Mayne, "Differential Dynamic Programming," 1970

**Model Predictive Control (MPC) / Receding Horizon Control**

- At convergence of iLQR/DDP, we have linearizations around the converged trajectory
- In practice, the system may be off-trajectory due to perturbations, initial state error, or model error
- Solution: at time $t$, re-solve the control problem using iLQR/DDP over $t$ through $H$
- Full replanning is often impractical; instead re-plan over a shorter horizon $h$ (receding horizon)
- Requires a cost-to-go $J(t+h)$ accounting for future costs; can be taken from the offline iLQR/DDP run

**Bounded Controls**

- Often $u_t \in [-1, +1]$
- Reparameterize: $u_t = \tanh(v_t)$, optimize over $v_t$ instead
- Penalize $v$ for being far from zero to keep optimization well-conditioned

**Lyapunov's Linearization Method**

- Once a controller is designed: $x_{t+1} = f(x_t)$ is the autonomous closed-loop system
- $x^*$ is an asymptotically stable equilibrium if $\exists \varepsilon > 0$ s.t. for all $\|x - x^*\| \le \varepsilon$, $\lim_{t \to \infty} x_t = x^*$
- If $x^*$ is asymptotically stable for the linearized system, it is asymptotically stable for the nonlinear system
- If $x^*$ is unstable for the linear system, it is unstable for the nonlinear system
- If marginally stable for the linear system, no conclusion can be drawn

**Controllability**

- A linear system is $t$-time-steps controllable if from any $x_0$ we can reach any $x^*$ at time $t$
- For LTI: $x_t = A^t x_0 + \sum_{i=0}^{t-1} A^{t-1-i} B u_i$
- System is $t$-time-steps controllable iff the controllability matrix has full rank
- Cayley-Hamilton: for $t \ge n$, controllability is determined by $\operatorname{rank}([B, AB, A^2B, \ldots, A^{n-1}B]) = n$

**Feedback Linearization**

- For nonlinear systems $\dot{x} = f(x) + g(x)u$
- Find a diffeomorphism (smooth, invertible coordinate transformation) that converts the system to a linear one
- Condition can be checked by applying the chain rule and examining ranks of certain matrices
- Reference: Slotine and Li, Chapter 6; Isidori, *Nonlinear Control Systems*, 1989

**Learning Linear Dynamics Latent Spaces**

- Embed to Control: locally linear latent dynamics model for control from raw images [Watter et al.]
- Deep Spatial Autoencoders for Visuomotor Learning [Finn et al.]
- SOLAR: Deep Structured Representations for Model-Based RL [Zhang et al.]

**Key Applications / Classic Examples**

- Cart-pole stabilization: LQR around the unstable upright equilibrium; varying $Q$ and $R$ trades off stabilization region vs. control effort
- Helicopter flight, quadrotor control
- Robotic manipulation with visuomotor policies using learned latent linear dynamics$
$