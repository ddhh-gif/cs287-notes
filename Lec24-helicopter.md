**Autonomous Helicopter Flight**

*Learning dynamic aerobatic maneuvers from human pilot demonstrations: trajectory alignment, locally-weighted dynamics models, and autonomous airshow execution.*

**Challenges in Helicopter Control**

Helicopters are unstable, nonlinear, and have complicated dynamics involving airflow, mechanical coupling, and blade dynamics. State estimates (position, orientation, velocity, angular rate) are noisy. Prior work succeeded in hover and stationary flight by restricting to specific regimes, collecting extensive data (frequency sweeps), and building model-based controllers (PID, $H_\infty$, LQR).

Aggressive flight introduces additional challenges: the target trajectory is difficult to specify by hand, and accurate dynamics models are harder to obtain.

**Learning Target Trajectories**

Human pilot demonstrations are noisy and suboptimal. A clean target trajectory must be extracted from multiple demonstrations using an HMM-like generative model:
- The hidden trajectory evolves according to the helicopter dynamics model (transition model)
- Each demonstration is an observation of the hidden trajectory
- Dynamic Time Warping + Extended Kalman filter/smoother align the demonstrations in time and extract the intended trajectory

Even without prior knowledge, the inferred trajectory is much closer to an ideal maneuver (e.g., closer to a perfect loop for aerobatics).

**Learning Dynamics Models**

**Standard approach** (frequency sweeps → global model): A single linear model fitted to frequency sweep data yields 3G of prediction error on aggressive maneuvers — completely unacceptable.

**Key observation**: When the same trajectory is flown repeatedly, errors are consistent across flights after time alignment. Unmodeled variables (air currents, actuator delays) tend to repeat. This is akin to "muscle memory" for human pilots.

**Trajectory-specific local models**: Learn a locally-weighted linear model at each time $t$ into the maneuver:
- Weight data points by temporal proximity to $t$
- Since data is time-aligned from multiple demonstrations, this exploits the repeatability of unmodeled effects
- Run weighted regression separately for each time step

**Experimental Setup**

- Microstrain 3DM-GX1 IMU (3-axis magnetometer, accelerometer, gyroscope) at 333Hz
- Offboard cameras at 1280×960, 20Hz
- Sonar for altitude
- RPM sensor
- Extended Kalman filter for state estimation
- RHDDP (Receding Horizon Differential Dynamic Programming) controller at 20Hz

**Experimental Procedure**

1. Collect frequency sweeps for a baseline dynamics model
2. Expert pilot demonstrates the airshow multiple times
3. Learn a target trajectory from demonstrations (DTW + EKF smoother)
4. Learn trajectory-specific local dynamics models (temporally-weighted regression)
5. Find the optimal control policy for the learned target and dynamics model
6. Fly autonomously
7. Learn an improved dynamics model from autonomous flight data; iterate from step 4

New maneuvers learned in under 1 hour.

**Results**

Autonomous execution of a full airshow program including loops, rolls, and flips. The helicopter autonomously performs chaos maneuvers (flip/roll parameterized by yaw rate). Autonomous autorotation landings were also demonstrated — an emergency landing procedure where the engine is disengaged and the helicopter uses rotor inertia to land safely.

**Key applications / classic examples**

- Autonomous helicopter aerobatics (loops, rolls, stall-turns, chaos maneuvers)
- Aggressive flight in rapid succession (full airshow programs)
- Autonomous autorotation (emergency landing without engine power)
- Trajectory learning and model adaptation from human demonstrations more generally
