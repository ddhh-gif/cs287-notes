**Imitation Learning**

*Learning policies from expert demonstrations: behavioral cloning, DAgger, and inverse reinforcement learning for acquiring skills without hand-coded reward functions.*

**Motivation**

Directly specifying reward functions for complex tasks (folding towels, cooking, making a user happy) is extremely difficult. Demonstrations provide a natural alternative: the agent learns by observing expert behavior rather than through trial-and-error with a reward signal.

**Behavioral Cloning**

The simplest form of imitation learning: treat the problem as supervised learning. Given state-action pairs $(s, a)$ from expert demonstrations, learn a policy $\pi_\theta(s) \to a$ that minimizes prediction error:

$\theta^* = \operatorname{argmin}_\theta \sum_{(s,a) \in \text{demos}} \|\pi_\theta(s) - a\|^2$

Challenge: compounding errors. Small mistakes cause the agent to drift into states not seen in the demonstrations, where the learned policy has no training data. Errors compound over time and the agent diverges from the expert trajectory.

**DAgger (Dataset Aggregation)**

Iteratively addresses the distribution mismatch:

- Train initial policy on expert demonstrations
- Execute the learned policy, ask expert to label the visited states with correct actions
- Aggregate new labeled data into the training set
- Retrain and repeat

This ensures the policy is trained on states it actually visits, reducing compounding errors.

**Inverse Reinforcement Learning (IRL)**

Rather than directly copying actions, infer the reward function that the expert is optimizing. Given:
- Expert demonstrations $\tau_1, \ldots, \tau_n$
- Knowledge of the MDP (dynamics, state/action spaces)

Goal: find a reward function $R(s, a)$ such that the expert's policy is optimal under that reward.

The principle of maximum entropy IRL finds the reward that makes expert behavior most probable while maintaining maximum uncertainty about unobserved behaviors. The learned reward can then be used with standard RL to train a policy that generalizes beyond the demonstrated scenarios.

**Key applications / classic examples**

- Autonomous helicopter aerobatics from human pilot demonstrations
- Robotic manipulation skills learned from teleoperation
- Self-driving car lane following from human driving data
- Learning dexterous in-hand manipulation from human demonstrations
