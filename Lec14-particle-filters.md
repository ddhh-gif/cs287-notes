**Particle Filters and Monte Carlo Localization**

*Nonparametric Bayes filters using weighted sample sets to represent arbitrary distributions, with application to mobile robot localization.*

**Motivation**

Estimating the state of a dynamical system is fundamental. The recursive Bayes filter provides an effective approach, but representing the belief is challenging. Particle filters efficiently represent arbitrary (non-Gaussian) distributions through:
- A set of state hypotheses (particles)
- Survival-of-the-fittest resampling

**Bayes Filter Recap**

The recursive Bayes filter alternates prediction and correction:

$\overline{bel}(x_t) = \int p(x_t \mid u_t, x_{t-1}) \, bel(x_{t-1}) \, dx_{t-1}$

$bel(x_t) = \eta \, p(z_t \mid x_t) \, \overline{bel}(x_t)$

**Sampling Methods**

**Rejection sampling**: To draw from $f(x)$ bounded by $a$, sample $x$ uniformly and $c \sim U[0, a]$. Keep the sample if $f(x) > c$.

**Importance sampling**: Use a proposal distribution $g$ to generate samples from target $f$. Importance weight $w = f/g$ accounts for differences between $g$ and $f$. Precondition: $f(x) > 0 \implies g(x) > 0$.

- $f$ = target distribution
- $g$ = proposal distribution
- Weighted samples approximate $f$

**Particle Filter Representation**

A particle set $S_t = \{(x_t^{(i)}, w_t^{(i)}) \mid i = 1, \ldots, n\}$ represents the belief:
- $x_t^{(i)}$: state hypothesis
- $w_t^{(i)}$: importance weight

**Particle Filter Algorithm**

1. **Prediction**: For each particle, sample $x_t^{(i)} \sim p(x_t \mid x_{t-1}^{(i)}, u_t)$ using the motion model as proposal
2. **Correction**: Compute importance weight $w_t^{(i)} = p(z_t \mid x_t^{(i)})$ (target/proposal simplifies to observation likelihood)
3. **Resampling**: Draw $n$ new particles with replacement, probability proportional to weights

The proposal distribution is the motion model. The target is the posterior belief. The weights reduce to the observation likelihood because the motion model cancels:

$w_t^{(i)} = \frac{\eta \, p(z_t \mid x_t) \, p(x_t \mid x_{t-1}, u_t) \, bel(x_{t-1})}{p(x_t \mid x_{t-1}, u_t) \, bel(x_{t-1})} \propto p(z_t \mid x_t)$

**Resampling**

Given weighted samples, draw $n$ times with replacement where $p(x_i) = w_i$.

Methods:
- Roulette wheel with binary search: $O(n \log n)$
- Stochastic universal sampling (systematic resampling): $O(n)$, easy to implement, low variance

**Particle Filter Localization**

Each particle is a potential robot pose. The motion model propagates particles (prediction). The observation model weights them (correction). Resampling prevents loss of good hypotheses — without it, particles with negligible weights waste the finite particle budget.

**Handling the Kidnapped Robot Problem**

- Randomly insert particles with random poses at each iteration
- Alternatively, insert samples inversely proportional to the average observation likelihood (lower likelihood → higher probability of being wrong → more random exploration)

**Properties**

- Can model arbitrary (non-Gaussian) distributions
- Nonparametric: complexity scales with number of particles
- Also known as: Monte Carlo filter, Survival of the fittest, Condensation, Bootstrap filter

**Key applications / classic examples**

- Mobile robot localization with laser/sonar/vision sensors
- Global localization (no prior knowledge of pose)
- Ceiling-map-based localization
- Vision-based localization using camera measurements
