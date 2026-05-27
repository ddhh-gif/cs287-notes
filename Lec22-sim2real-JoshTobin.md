**Sim-to-Real Transfer and Domain Randomization**

*Guest lecture by Josh Tobin: bridging the reality gap through simulation design, domain adaptation, and domain randomization for transferring policies from simulation to real robots.*

**The Real World Is Not an MDP**

Real-world states are complex and ambiguous. Observations are high-dimensional, multimodal, and noisy. Reward is hard to specify (how do you write a reward function for folding a towel or making a user happy?). Controller design requires deep understanding of the system and does not scale well to high-dimensional problems. Robots break and degrade constantly.

**The Data Problem**

Deep learning is data hungry (ImageNet: 1.2M images; machine translation: 36M sentence pairs; DeepRL: 38M+ timesteps), but robotic data is expensive — limited by robot cost, safety constraints, and labeling effort.

Approaches to mitigate data scarcity:
- Large-scale robotic data collection (3,000 hours for hand-eye coordination, 50K grasps for self-supervised grasping)
- Efficient RL (model-based, meta-learning, learning from demonstrations)
- Unsupervised robotic learning (learning models, feature spaces, or via self-supervised objectives)

**Advantages of Simulated Data**

Cheap, fast, scalable, safe, labeled, and not beholden to real-world probability distributions. Simulators can also expose the agent to rare edge cases that are underrepresented in real data.

**Why Sim-to-Real Is Hard**

1. Accurate sensor and physics modeling is difficult and computationally expensive. Physics simulators make big assumptions: convex objects, discrete time, rigid bodies, Coulomb friction.
2. Accurate models require measuring unobservable parameters (damping, inertia, friction).
3. Photorealistic sensor simulation is expensive and unsolved (e.g., LIDAR simulation involves modeling scattering, material properties, weather effects).
4. Neural networks overfit to tiny differences in data distributions.
5. Errors compound over time during closed-loop execution.

**Using Simulation Without Solving Sim-to-Real**

- Prototyping algorithms (OpenAI Gym)
- Debugging software stacks (Gazebo + ROS with realistic latency)
- Prototyping systems (robot selection, cell design, ROI estimation)
- Reliability testing / continuous integration (1 billion miles in simulation for autonomous vehicles)
- "Simulation-first" development: make simulation harder than reality, be rigorous about randomness, diagnose errors to find bugs

**Building a Good Simulator**

**Design process**: Design simulation model → create scenarios → collect data & improve simulation.

**Simulator options**: Bullet/PyBullet, MuJoCo (most common); also Drake, Gazebo.

**World design**: 3D models from ShapeNet (60K objects, varying quality), YCB/BigBird (high quality, limited objects), Dex-Net (10K objects), Unity asset store. Object placement: random, physics-based, or procedural.

**System identification**: Find simulation parameters $\eta$ that minimize the distance between simulated and real trajectories:

$\min_\eta \sum_a D(\tau_\eta(a), \tau_r(a))$

where $\tau_\eta$ is the simulated trajectory and $\tau_r$ is the real trajectory for action sequence $a$. Use iterative coordinate descent; exclude changes that improve performance < 0.1%.

**Domain Adaptation**

Using real data to bridge the sim-to-real gap:
- **Supervised**: Progressive nets (lateral connections from sim-trained to real network), learning inverse dynamics models, using simulation as a Bayesian prior, dimensionality reduction from sim to guide real exploration
- **Weakly supervised**: Pairwise constraints between sim and real representations
- **Self-supervised / unsupervised**: CyCADA (cycle-consistent adversarial domain adaptation), sim-to-real grasping via domain adaptation

**Domain Randomization**

Core idea: if the model sees enough simulated variation, the real world may look like just the next simulator.

**History**: Radical Envelope of Noise Hypothesis (Jakobi, 1997) — create a "minimal simulation" with a base set (aspects sufficient for the behavior, randomized slightly) and implementation aspects (everything else, randomized enough so controllers learn to ignore them).

**Applications**:
- Live repetition counting (trained on random noise patterns, tested on real videos)
- CAD$^2$RL: quadcopter collision avoidance from ~500 semi-realistic textures, 40-50% collision-free on 1000m real trajectories
- Pose estimation: unrealistic randomized scenes with procedural textures → 1.5cm real-world accuracy, enabling grasping
- Object/scene randomization: random procedurally-generated objects work as well as realistic 3D models for grasping
- Dynamics randomization: train recurrent policies with different physics parameters per rollout → sim-to-real transfer of locomotion and in-hand manipulation (OpenAI hand)

**Why DR works**:
1. Training data comes from a covering distribution (though in high dimensions, truly covering the real distribution requires massive data)
2. DR specifies to the model what to ignore — randomized variations across training force the model to focus on invariant features
3. DR is meta-learning: the recurrent policy's hidden state adapts to each randomized environment during a rollout, learning fast adaptation across environments

**Tools**: Gazebo Awesome Plugins, NDDS (Unreal), ORRB (Unity), DeepDrive.

**Practical workflow**: Build a simulated world → calibrate it to the environment → design randomizations to cover real-world variability → train a model and evaluate in real → examine failure modes and add randomization.

**Challenges**: Building simulations is manual and time-consuming; deciding what to randomize requires judgment; randomizing parameters maximally may not be optimal.

**Extensions**:
- Randomized-to-Canonical Adaptation Networks (specialized architectures for transfer)
- SimOpt: close the sim-to-real loop by adapting randomization distributions from real-world experience
- Meta-Sim: use real data to generate realistic randomization distributions via scene graphs
- Learning to Simulate: parameterize the space of simulator distributions as an RL policy, trained to maximize real-world validation accuracy
- Active Domain Randomization: curriculum learning over randomization ranges — sample harder cases where policy performs poorly
- Network-Driven Domain Randomization (DeceptionNet): restrict randomizations to differentiable deception modules (distortion, background/foreground, noise)
- Policy Transfer with Strategy Optimization: train a conditional policy $\pi_\eta$ over physics parameters $\eta$, then optimize $\eta^*$ on the real robot
- Automatic Domain Randomization (ADR): automatically expand randomization ranges when performance is good at the boundary, narrow when poor — used in OpenAI Rubik's Cube hand

**Key applications / classic examples**

- Dextrous in-hand manipulation (OpenAI, trained with PPO on massively randomized MuJoCo simulations)
- Object pose estimation from synthetic data for robotic grasping (1.5cm accuracy)
- Quadcopter collision avoidance without any real images
- Autonomous vehicle testing (Waymo: 1B miles in simulation)
- Rubik's Cube solving with a robot hand (ADR + progressive randomization curriculum)
