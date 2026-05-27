**Optimization for (Locally) Optimal Control**

*Shooting and collocation are the two principal formulations for trajectory optimization. Shooting optimizes over controls through forward rollouts; collocation treats states and controls as independent decision variables. MPC adds closed-loop robustness by replanning online.*

**Optimal Control Approaches**

Two axes of choice:

- **Formulation**: Shooting (optimize over controls $u_{0:H}$, roll out dynamics) vs. Collocation (optimize over both $x_t$ and $u_t$, enforce dynamics as constraints)
- **Output**: Open-loop controls $u_0, u_1, \ldots, u_H$ vs. Feedback policy (e.g., linear or neural net)

**Third Axis: Execution Strategy**

- Roll out the full open-loop sequence $u_0, u_1, \ldots, u_H$
- OR: Model-Predictive Control (MPC): take only the first action $u_0$, then re-solve the optimization from $t=1$ to $H$ after observing $x_1$
- Repeat for $t = 1, 2, \ldots, H$

MPC formulation at time $t$:

$\min_{x,u} \sum_{k=t}^T c_k(x_k, u_k) \quad \text{s.t.} \quad x_{k+1} = f(x_k, u_k) \; \forall k \in \{t, \dots, T-1\}, \quad x_t = \bar{x}_t$

Computational trick for low-latency control: warm-start the optimization with the solution from time $t-1$.

**Instability of Open-Loop Shooting**

- Rolling out $u_0, u_1, \ldots, u_H$ can be unstable: small numerical errors or noise amplify under unstable dynamics
- This makes the shooting formulation itself unstable to optimize
- Solutions:
  - During roll-out, use MPC (re-plan at each step)
  - Use the feedback version (or collocation), which avoids long unrolled trajectories

**Backpropagation Through Time (BPTT)**

In shooting formulations, computing derivatives w.r.t. $u$ (or policy parameters $\theta$) involves shared subcomputations across time steps. BPTT avoids duplicate computation.

Gradient for policy parameters $\theta$:

$\frac{\partial U}{\partial \theta_i} = \sum_{t=0}^H \frac{\partial R}{\partial s}(s_t) \frac{\partial s_t}{\partial \theta_i}$

where

$\frac{\partial s_t}{\partial \theta_i} = \frac{\partial f}{\partial s}(s_{t-1}, u_{t-1}) \frac{\partial s_{t-1}}{\partial \theta_i} + \frac{\partial f}{\partial u}(s_{t-1}, u_{t-1}) \frac{\partial u_{t-1}}{\partial \theta_i}$

and

$\frac{\partial u_t}{\partial \theta_i} = \frac{\partial \pi_\theta}{\partial \theta_i}(s_t, \theta) + \frac{\partial \pi_\theta}{\partial s}(s_t, \theta) \frac{\partial s_t}{\partial \theta_i}$

**Shooting vs. Collocation Tradeoffs**

Shooting:

- Controls $u$ (or policy parameters $\pi$) are meaningful throughout optimization
- Often poorly conditioned: effect of early $u$ is much larger than later $u$
- Not clear how to initialize to nudge toward a goal state

Collocation:

- May converge to a local optimum that is infeasible; until convergence, trajectory is often not dynamically feasible
- Decoupling between time steps via state variables $x$ makes computation stable
- Can initialize with simple linear interpolation or a guess of a good trajectory

Iterative LQR is a specific example of a shooting method with linear controllers and second-order optimization.

**MPC with Collocation: Example**

1. Use collocation to solve the open-loop problem for $u_0, \ldots, u_H$
2. Execute $u_0$
3. Observe resulting state $\bar{x}_1$
4. Re-solve the collocation problem from $t = 1$ to $T$ with $x_1 = \bar{x}_1$ as initial constraint
5. Repeat

Warm-start the collocation solver with the previous solution (shifted by one time step) for low-latency control.

**Key Applications**

- Dubin's car: collision-free path planning via collocation
- Quadrotor aggressive maneuvers with MPC
- Legged locomotion with receding horizon control
- Manipulator trajectory optimization under constraints$
$