## [Noise Conditioned Score Function (NCSN)](https://arxiv.org/abs/1907.05600)

The model predicts the **score** instead of the noise.
Instead of predicting noise directly, the model predicts the direction to move to remove noise .
### Score function

$$s_\theta(x_t, t) \approx \nabla_{x_t} \log p(x_t)$$

 **Score** = gradient of the log probability density of $x_t$
 It points in the direction where real data is more likely
- **Direction:** where to move toward higher density
- **Magnitude:** how strongly to move

We start from pure noise: $x_T \sim \mathcal{N}(0, I)$.

Then we repeatedly move the sample in the direction of the score:

$$x_{t-1} = x_t + \eta\, s_\theta(x_t, t)$$

- $\eta$: small step size
- $s_\theta(x_t, t)$: predicted score

Each step moves toward higher-density (more realistic) regions; eventually noise becomes structured data.

### Why condition on noise level?

Real data often lies on a low-dimensional manifold. The score of clean data can be undefined or very large, which makes training unstable.

We fix this by **adding noise at different levels** so that the density is spread into the surrounding space:

$$x_t = x_0 + \sigma_t\,\epsilon, \quad \epsilon \sim \mathcal{N}(0, I)$$

- $x_0$: clean data
- $\sigma_t$: noise scale at level $t$ (we use several levels)
- This gives a well-defined score at every noise level

### Noise-conditioned score network

Instead of learning the score only at clean data, the network learns the score **at all noise levels** $t$:

$$s_\theta(x_t, t) \approx \nabla_{x_t} \log p_t(x_t)$$

where $p_t(x_t)$ is the distribution of $x_t = x_0 + \sigma_t \epsilon$ when $x_0 \sim p_{\text{data}}$ and $\epsilon \sim \mathcal{N}(0,I)$.

### Training objective (denoising score matching)

We train the network using noisy data:

$$x_t = x_0 + \sigma_t \epsilon, \quad \epsilon \sim \mathcal{N}(0, I)$$

- $x_0$: clean data
- $\sigma_t$: noise scale at level $t$
- $\epsilon$: Gaussian noise

The **target score** is the direction we want the network to predict: 
- the gradient of the log density of $x_t$ given $x_0$. It points from the noisy sample back toward the clean data. For our noise model it has a closed form:

$$\nabla_{x_t} \log p_t(x_t \mid x_0) = -\frac{\epsilon}{\sigma_t}$$

- $x_t = x_0 + \sigma_t \epsilon$: noisy sample (Gaussian around $x_0$)
- Gradient of log density with respect to $x_t$ equals $-\epsilon/\sigma_t$
- So the network is trained to output this direction, at test time it tells us which way to move toward real data
$$\mathcal{L} = \mathbb{E}_{x_0,\, \epsilon,\, t}\!\left[\left\| s_\theta(x_t, t) + \frac{\epsilon}{\sigma_t} \right\|^2\right]$$

Minimizing this makes $s_\theta(x_t, t) \approx -\epsilon/\sigma_t$, i.e. the network learns to point toward the clean data $x_0$ (opposite to the noise direction).

### Sampling process (Langevin dynamics)

We generate samples by iteratively applying the score and adding a small noise term (Langevin dynamics):

$$x_{t-1} = x_t + \eta\, s_\theta(x_t, t) + \sqrt{2\eta}\, z, \quad z \sim \mathcal{N}(0, I)$$

- $\eta$: step size
- $s_\theta(x_t, t)$: score moves the sample toward higher-density (real data) regions
- $\sqrt{2\eta}\, z$: noise term that keeps the correct stationary distribution

After enough steps, noise becomes structured data.

