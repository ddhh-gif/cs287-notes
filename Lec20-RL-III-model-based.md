**Model-Based RL**

*Learning dynamics models from data and using them for policy optimization: ensemble methods for robustness, meta-learning for adaptation, asynchronous training, and vision-based approaches.*

**Model-Based RL Framework**

For iter = 1, 2, ...:
- Collect data under current policy
- Learn dynamics model from past data
- Improve policy using the dynamics model (e.g., SVG(k), TRPO/A3C in the learned simulator)

Anticipated benefit: much better sample efficiency. A learned model can enable more significant policy updates than a single policy gradient step, and the model is reusable for other tasks.

**Why Model-Based RL Is Not Always Used**

- Training instability: policy optimization exploits regions where insufficient data was collected to train the model, leading to catastrophic failures (model bias)
- Not matching asymptotic performance of model-free methods

**Ensemble Methods and ME-TRPO**

Learned dynamics models overfit to training data. A new overfitting challenge specific to model-based RL: policy optimization tends to exploit regions where insufficient data is available, causing catastrophic failures.

**Model-Ensemble TRPO (ME-TRPO)**: Train an ensemble of dynamics models. Use the ensemble during policy optimization to provide uncertainty-aware rollouts. The ensemble disagreement signals where the model is unreliable, preventing the policy from exploiting model inaccuracies.

Key findings: TRPO significantly outperforms BPTT (backprop through time) in model-based RL; ensemble size matters; using the ensemble for policy optimization achieves better sample efficiency than model-free methods on standard MuJoCo benchmarks.

**Model-Based RL via Meta Policy Optimization (MB-MPO)**

Even with ensembles, the learned model remains imperfect. The policy trained in simulation may not be optimal in the real world. Rather than trying to learn a perfect model (attempted fix 1, insufficient so far), MB-MPO learns an **adaptive policy** (attempted fix 2).

**Key idea**: Learn an ensemble of models representative of how the real world generally works. Learn a meta-policy that can quickly adapt to any individual learned model. At test time, this adaptive policy quickly adapts to the real-world dynamics.

For iter = 1, 2, ...:
- Collect data under current adaptive policies
- Learn ensemble of $K$ simulators from all past data
- Meta-policy optimization over the ensemble → new meta-policy → new adaptive policies

MB-MPO achieves asymptotic performance comparable to state-of-the-art model-free methods while maintaining the sample efficiency of model-based approaches.

**Asynchronous Model-Based RL**

In synchronous model-based RL, data collection, model learning, and policy improvement operate sequentially. The asynchronous framework runs them in parallel:
- Data collection worker gathers experience
- Model learning worker updates the dynamics model from the data buffer
- Policy improvement worker optimizes the policy using the current model

Benefits beyond wall-clock speedup:
- Policy learning regularization: the policy sees slightly outdated models, preventing overfitting
- Improved exploration: data is collected with policies derived from older models, increasing diversity

The asynchronous framework is robust to data collection frequency and effective on real robot tasks (reaching, shape matching, stacking Lego).

**Vision-Based Model-Based RL**

High-dimensional observations (images) make dynamics modeling more challenging. Approaches include:

**World Models**: Learn a compact latent representation (VAE), learn dynamics in the latent space (MDN-RNN), and train a controller in the compact space.

**Embed to Control**: Learn a latent embedding and latent dynamics jointly, then plan in the latent space.

**SOLAR**: Learn a deep structured representation with latent dynamics. Iterate: infer latent dynamics given observed data, update policy given latent dynamics, collect new data.

**Deep Spatial Autoencoders**: Train a spatial autoencoder for visual feature extraction, run iLQR in the latent space for model-based control.

**PlaNet**: Learn latent space dynamics model for multi-step prediction; plan directly in the latent space using model-predictive control.

**Visual Foresight**: Video prediction + Cross-Entropy Method for MPC — predict future frames and use them for planning.

**Forward + Inverse Dynamics Models**: Learn latent features such that $s_t, s_{t+1}$ can predict $a_t$ (inverse model constraint prevents the latent features from collapsing to zero).

**Key representations literature**: PVEs (Position-Velocity Encoders), DARLA (disentangled representations for zero-shot transfer), Causal InfoGAN (plannable representations), Successor Features, Predictron, and the separation principle for control with deep learning.

**Key applications / classic examples**

- MuJoCo locomotion tasks (ME-TRPO, MB-MPO comparing with TRPO, PPO)
- Real robot reaching, insertion, and stacking (asynchronous model-based RL)
- Car racing from pixels (World Models)
- Visual robotic control (visual foresight, deep spatial autoencoders)
- Robotic pushing and poking (forward + inverse dynamics models)
