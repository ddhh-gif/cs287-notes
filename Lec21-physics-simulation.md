**Physics Simulation**

*A lightning tour of rigid body dynamics, Lagrangian mechanics, numerical integration, and collision/contact handling for robotics simulation.*

**Newton's Laws for Rigid Bodies**

For a point mass: $\sum F = m \ddot{x}$. For a rigid body, forces cause both linear and angular acceleration. Newton's laws are generally applicable but become cumbersome in multi-body systems with constraints and internal forces.

**Lagrangian Dynamics**

The Lagrangian formulation eliminates internal forces and expresses dynamics in terms of generalized coordinates $q_i$ (the degrees of freedom of the system).

Define $L = T - U$ where $T$ is total kinetic energy and $U$ is total potential energy. The Lagrangian dynamic equations:

$\frac{d}{dt}\frac{\partial L}{\partial \dot{q}_i} - \frac{\partial L}{\partial q_i} = Q_i$

where $Q_i$ are generalized forces (e.g., actuator torques).

**Standard robot dynamics form**: $H(q)\ddot{q} + C(q, \dot{q}) + G(q) = B(q)u$, where $H$ is the inertia matrix, $C$ contains Coriolis and centripetal terms, $G$ is gravity, and $B$ maps controls to generalized forces.

**Examples**: Double pendulum (2-DOF), cart-pole (underactuated, 2-DOF), acrobot (underactuated, 2-DOF), car models (tricycle, simple car, Reeds-Shepp, Dubins).

**Friction and drag**: Static friction coefficient $\mu_s$ > dynamic friction $\mu_d$. Drag forces are often modeled proportional to velocity.

**Continuous Time to Discrete Time**

Numerical integration schemes for $\dot{x} = f(x, u)$:

- **Forward Euler (explicit)**: $x_{t+1} = x_t + \Delta t \cdot f(x_t, u_t)$ — simple but can be unstable for stiff systems
- **Backward Euler (implicit)**: $x_{t+1} = x_t + \Delta t \cdot f(x_{t+1}, u_{t+1})$ — requires solving an equation, more stable
- **Symplectic (semi-implicit) Euler**: Updates velocities implicitly and positions explicitly — energy-conserving properties, good for long-term simulation
- **Runge-Kutta methods**: Higher-order methods (most commonly RK4) with multiple intermediate evaluations per step for improved accuracy

**Collision Checking**

**Broad phase**: Quickly eliminate pairs of objects that cannot possibly collide. Methods include:
- Spatial partitioning (quadtrees, octrees, grids)
- Conservative bounding volume checks (AABB, spheres)

**Narrow phase**: Precise collision detection for pairs that pass broad phase:
- Convex-convex: Separating Axis Theorem
- GJK (Gilbert-Johnson-Keerthi) algorithm: computes the distance/closest points between convex shapes
- EPA (Expanding Polytopes Algorithm): computes penetration depth from GJK result

**Contact**

Contact resolution typically uses an impulse formulation. Upon collision, instantaneous velocity changes are computed to prevent interpenetration and enforce friction constraints (typically Coulomb friction — cone of friction).

**Simulation Engines**

- **MuJoCo**: Optimized for model-based control and RL, soft contact model, fast and differentiable
- **Bullet** (PyBullet): Open-source, widely used, supports rigid and soft bodies, constraint solving via sequential impulse
- **Drake**: From MIT/TRI, focused on model-based design and control
- **Gazebo**: Full robotics simulator with ROS integration, sensor simulation

**Robot Specification**: Typically parameterized via Denavit-Hartenberg parameters. In practice: URDF (Universal Robot Description Format) files specify links, joints, inertias, and visual/collision geometry.

**Key applications / classic examples**

- Mujoco for reinforcement learning (Gym environments)
- Bullet for game physics and robotics research
- Gazebo + ROS for integrated robot system development
- Drake for control design and trajectory optimization
