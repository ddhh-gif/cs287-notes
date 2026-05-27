**Introduction to Advanced Robotics**

*Core algorithmic pillars of robotics — optimization, probabilistic reasoning, and learning — generalize across diverse hardware platforms. Robotic success stories in autonomous driving, legged locomotion, and manipulation motivate the mathematical techniques covered in the course.*

**Why Study Robotics?**

- Real-world robotics is a challenging test-bed for AI: far less forgiving than simulation or games
- Biology provides evidence intelligence can emerge in physical settings
- Progress in robotics can have direct real-world impact
- Robotic hardware is rapidly improving; expertise in algorithms, math, and programming is now the limiting factor
- Diverse robotic systems — manipulators, drones, legged robots, self-driving cars — share a few core techniques:
  - Optimization
  - Probabilistic Reasoning
  - Learning
- These techniques extend well beyond robotics

**Hardware Cost Trends**

- PR2 (Willow Garage, 2009): $400,000
- Baxter (Rethink Robotics, 2013): ~$30,000
- Projected consumer robot cost (2017): ~$3,000
- Comparable hardware by 2016: ~$100,000

**Self-Driving Cars**

- DARPA Grand Challenge 2004: CMU vehicle drove 7.36 of 150 miles
- DARPA Grand Challenge 2005: 5 teams finished; Stanford won
- DARPA Urban Challenge 2007: urban environment with other vehicles; 6 teams finished, CMU won
- Google self-driving car (2010): Mountain View to Santa Monica; >140,000 miles; Lombard Street, Golden Gate, Lake Tahoe, Pacific Coast Highway
- By July 2015: 1M autonomous miles, 14 minor accidents (never at fault)
- Ernst Dickmanns / Mercedes Benz: autonomous highway driving in Europe; 1758km Munich to Odense with lane changes up to 140km/h; longest autonomous stretch 158km (1995)
- Industry guests: Drago Anguelov (Research Director, Waymo); Jur van den Berg / Sachin Patil (Co-Founders, Ike)
- Techniques: Kalman filtering, LQR, mapping, terrain and object recognition

**Autonomous Helicopter Flight**

- [Abbeel, Coates & Ng, 2010]
- Industry guest: Adam Bry (Founder/CEO, Skydio)
- Techniques: Kalman filtering, model-predictive control, LQR, system identification, trajectory learning

**Four-Legged Locomotion**

- Without learning: value iteration, receding horizon control, motion planning, inverse reinforcement learning
- With learning: learned locomotion policies [Schulman, Moritz, Levine, Jordan, Abbeel, 2015]
- Techniques: reinforcement learning, policy gradients, value function approximation

**Mapping and SLAM**

- Baseline: raw odometry data with laser range finder scans
- FastSLAM: particle filter + occupancy grid mapping
- Essential for mobile manipulation and navigation

**Mobile Manipulation**

- SLAM, localization, motion planning for navigation and grasping
- Grasp point selection, visual category recognition, speech recognition and synthesis
- Visuomotor learning [Levine, Finn, Darrell, Abbeel, 2015]: LQR, guided policy search, deep learning

**Additional Success Stories**

- SUPERBall [iLQR, imitation learning, sim2real]
- Knot tying [Schulman, Ho, Lee, Abbeel, ISRR 2013]: imitation learning, function approximation, trajectory optimization
- Fleet learning [Levine et al., 2016]: reinforcement learning at scale
- Amazon Picking Challenge 2017: like self-driving cars, the big challenge is the long tail of real-world scenarios

**Lecture Structure**

- Problem setting
- Key idea / intuition
- Math / algorithm derivation for clean setting (on whiteboard/iPad)
- Extensions
- Strongly recommended to work through derivations in parallel — these are the main source of midterm exam questions
