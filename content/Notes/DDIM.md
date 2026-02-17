## [DDIM (Denoising Diffusion Implicit Models)](https://arxiv.org/abs/2010.02502)

From the DDPM equations, we know that there is random noise added. However it's possible to follow the path of the noise without having random noise. This can be done **deterministically**.

DDIM is a **deterministic** diffusion sampling method that generates data by removing noise **without** adding noise at each step.

It uses the same trained neural net as DDPM but changes the reverse step to remove the stochastic part.

### ODE (Ordinary Differential Equation)

In the SDE we had $dx = f(x,t)\,dt + g(t)\,dW$ (drift + random noise). If we drop the $dW$ term we get an ODE:

- $dx = f(x,t)\,dt$
- $dx$: change in sample
- $f(x,t)$: neural net–defined denoising direction (no randomness)

Each step follows a fixed path instead of randomly sampling. This speeds up sampling and we need fewer steps because we can skip steps in the diffusion process.

We can remove the stochastic term ($\sigma_t z$) from the DDPM sampling equation so each step is deterministic.

### Skip steps

**1. Solve for clean data**

From the forward process we had $x_t = \sqrt{\bar{\alpha}_t}\,x_0 + \sqrt{1-\bar{\alpha}_t}\,\epsilon$. Solve for $x_0$ and use the NN’s predicted noise $\epsilon_\theta(x_t, t)$ instead of the true $\epsilon$:

$$\hat{x}_0 = \frac{x_t - \sqrt{1-\bar{\alpha}_t}\,\epsilon_\theta(x_t, t)}{\sqrt{\bar{\alpha}_t}}$$

So we get an estimate of the clean sample $\hat{x}_0$ from the current noisy $x_t$ and the predicted noise. $\bar{\alpha}_t$ is the cumulative signal retention at step $t$.

**2. Skip to a later timestep**

Treat $\hat{x}_0$ as the “clean” data and use the **forward** formula to jump to any other timestep $s$ (e.g. $s < t$):

$$x_s = \sqrt{\bar{\alpha}_s}\,\hat{x}_0 + \sqrt{1-\bar{\alpha}_s}\,\epsilon_\theta(x_t, t)$$

We reuse the same predicted noise $\epsilon_\theta(x_t, t)$ (from the current $x_t$) for the new $x_s$. So we don’t need to evaluate the network at every intermediate step. This allows skipping steps (e.g. $t = 1000 \to 800 \to 600$). It works because we’re following the same trajectory deterministically—no new randomness is added.

### Summary

- **Forward:** add noise
- **Neural net** predicts noise (same as DDPM)
- **DDIM** removes noise deterministically (no $\sigma_t z$)
- No randomness → faster sampling, fewer steps, and we can skip timesteps


