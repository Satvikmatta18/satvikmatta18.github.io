## Reverse variance parametrization

Reverse variance parametrization is how we define the **reverse distribution**: the distribution of the cleaner sample $x_{t-1}$ given the noisier sample $x_t$. 

We write it as a Gaussian with a **mean** (denoising direction) and a **variance** (how much randomness we add).

When we generate data, we start at pure noise $x_T$ and run backward step by step ($t = T \to T-1 \to \cdots \to 1$).  At each step we sample $x_{t-1}$ from this distribution. After $T$ steps we get $x_0$ (real data).

### Reverse distribution (one step: $x_t \to x_{t-1}$)

$$p_\theta(x_{t-1} \mid x_t) = \mathcal{N}\!\left(x_{t-1};\, \mu_\theta(x_t, t),\, \sigma_t^2 I\right)$$

- This is the distribution we use at **step $t$** of the reverse process: we have noisy $x_t$, and we sample the next, cleaner $x_{t-1}$ from this Gaussian.

**Mean** $\mu_\theta(x_t, t)$:
- Denoising direction: where to move from $x_t$ toward real data.
- Usually given by the network (e.g. via predicted noise $\epsilon_\theta(x_t,t)$).

**Variance** $\sigma_t^2$:
- Amount of randomness added at this step.
- Can be **fixed** (from the forward schedule) or **learned** (e.g. via interpolation).

**Sampling at step $t$:**

$$x_{t-1} = \mu_\theta(x_t, t) + \sigma_t z, \quad z \sim \mathcal{N}(0, I)$$

- So at each reverse step we take the mean (denoising direction) and add scaled noise $\sigma_t z$. Repeating this from $t = T$ down to $t = 1$ takes us from noise $x_T$ to data $x_0$

### Original DDPM: fixed variance

$$\sigma_t^2 = \beta_t \quad \text{(from the forward noise schedule)}$$

- Variance is **not learned**; it’s fixed from the forward process.
- **Pro:** Stable training.
- **Con:** Less flexible; can’t adapt randomness per step.

### Learned variance via interpolation

$$\sigma_t^2 = \exp\!\bigl(v \log \beta_t + (1-v) \log \tilde{\beta}_t\bigr)$$

- $\beta_t$ and $\tilde{\beta}_t$ are fixed bounds from the forward process.
- The network outputs $v$ (between 0 and 1) so we can pick a variance between those bounds. Log-space keeps things positive and stable.
- **Pro:** Variance can change per step; often allows fewer steps and better samples.
- **Con:** A bit more involved than fixed variance.


