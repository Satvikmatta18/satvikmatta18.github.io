## Variance schedule improvements

The **noise schedule** ($\beta_t$ or $\bar{\alpha}_t$) controls how much noise is added at each timestep. Original DDPM used linear. With later schedules often give better quality and sampling.

### [Linear schedule (original DDPM)](https://arxiv.org/abs/2006.11239)

$$\beta_t = \beta_1 + \frac{t-1}{T-1}(\beta_T - \beta_1)$$

- $\beta_t$: increases linearly from $\beta_1$ at $t=1$ to $\beta_T$ at $t=T$
- **Pro:** Simple
- **Con:** A lot of noise early; can destroy signal too fast

### [Cosine schedule](https://arxiv.org/abs/2102.09672)

$$\bar{\alpha}_t = \cos^2\!\left(\frac{t/T + s}{1 + s} \cdot \frac{\pi}{2}\right)$$

- $s$: small offset (e.g. 0.008) so $\bar{\alpha}_T$ is not exactly 0
- **Pro:** Slow at very start and end, faster in middle; more information-efficient
- **Con:** Slightly more involved than linear
- **Used in:** Improved DDPM and many later works

### Quadratic schedule

$$\beta_t = \beta_1 + (\beta_T - \beta_1) \cdot \left(\frac{t}{T}\right)^2$$

- $\beta_t$: grows quadratically in $t$ — small early, larger toward $t = T$
- **Pro:** Less early noise than linear
- **Con:** Less common than cosine or log-SNR

### Sigmoid schedule

$$\beta_t = \sigma\bigl(\gamma \cdot (2t/T - 1)\bigr), \quad \sigma(z) = \frac{1}{1 + e^{-z}}$$

- $\sigma$: sigmoid; $\gamma$: steepness. $\beta_t$ is an S-curve: flat at start/end, steep in middle
- **Pro:** Very slow start and end, fast in middle
- **Con:** Extra hyperparameter $\gamma$; less standard

### [Log-SNR schedule (modern)](https://arxiv.org/abs/2206.00364)

$$\text{log-SNR}(t) = \log \frac{\bar{\alpha}_t}{1 - \bar{\alpha}_t}$$

- Schedule the log signal-to-noise ratio instead of hand-picking $\beta_t$
- **Pro:** Direct control over signal vs noise at each $t$; good for few-step sampling
- **Con:** Need to convert desired log-SNR curve back to $\bar{\alpha}_t$
- **Used in:** EDM, Consistency Models, and other modern diffusion-style models

### [Karras noise schedule (EDM-style)](https://arxiv.org/abs/2206.00364)

Noise levels chosen in log-space so steps contribute more evenly.

- **Pro:** Simple, stable, good for few-step sampling; widely used
- **Con:** Different parameterization than classic $\beta_t$ DDPM
- **Used in:** EDM, Stable Diffusion 2, many recent codebases

### What is used now

- **Cosine** and **log-SNR / Karras-style** are most common in recent models
- Linear is mainly historical; quadratic and sigmoid are rarer
- For new work, cosine or a log-SNR / Karras schedule is the usual choice

