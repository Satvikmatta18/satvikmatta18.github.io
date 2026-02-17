## [DDPM (Denoising Diffusion Probabilistic Models)](https://arxiv.org/abs/2006.11239)

DDPM is a generative model that learns to create data by **gradually removing Gaussian noise**.

It defines two stochastic processes:

• **Forward process**: adds noise to real data over many timesteps until it becomes pure Gaussian noise  
• **Reverse process**: a neural network learns to remove this noise step-by-step to recover structured data

The process is **stochastic** because each step samples from a Gaussian distribution.

DDPM uses 100s–1000s of steps. 
## SDE (stochastic differential equation)

Continuous diffusion is modeled by a Stochastic Differential Equation (SDE):

$$dx = f(x,t)dt + g(t)dW$$

- *dx*: State transition
- *f(x, t)*: Deterministic drift
- *g(t)*: Noise scaling
- *dW*: Gaussian noise increment

DDPM's forward process adds noise via the SDE, while the reverse process is trained to undo that.
### Forward process

**Image → randomly add noise k times → Gaussian noise**

$$
X_0  \sim q(X_0)
$$
- $X_0$: input image/video/embedding
- $q(x)$: real data distribution

#### 1. Add noise at each timestep

At each timestep $t = 1, 2, \ldots, T$, we add small Gaussian noise. 
The transition is defined as:

$$
q(x_t \mid x_{t-1}) = \mathcal{N}\!\left(x_t;\,\sqrt{1-\beta_t}\,x_{t-1},\,\beta_t I\right)
$$
This means $x_t = \sqrt{1-\beta_t}\,x_{t-1} + \sqrt{\beta_t}\,\epsilon_t$

- $x_t$: noisy sample at timestep $t$
- $x_{t-1}$: previous timestep sample
- $\beta_t \in (0,1)$: noise schedule (controls noise strength)
- $\epsilon_t \sim \mathcal{N}(0, I)$: Gaussian noise 
- $I$: identity matrix 

**At each step:** New sample = scaled signal + scaled noise

- $\sqrt{1-\beta_t}$: how much signal is preserved from older data
- $\sqrt{\beta_t}$: how much new noise is added 

The noise increases over timesteps according to noise schedule, gradually destroying signal. The schedule is typically chosen so that early steps add little noise and later steps add more.

#### 2. Signal retention factor

Fraction of how much signal is preserved from the previous image.

$$
\alpha_t = 1 - \beta_t
$$

$$
x_t = \sqrt{\alpha_t}\,x_{t-1} + \sqrt{1-\alpha_t}\,\epsilon_t
$$
- $\alpha_t$: fraction of signal preserved
- $1-\alpha_t$: fraction replaced by noise

#### 3. Cumulative signal retention

After $K$ many steps, the total signal retained:

$$
\bar{\alpha}_t = \prod_{s=1}^{t} \alpha_s
$$
- Early timesteps -> $\bar{\alpha}_t$ = 1 
- Late timesteps -> $\bar{\alpha}_T$ = 0

This product matters because we often want to jump directly from $x_0$ to $x_t$ without computing every intermediate step, which is what the direct sampling form uses.

#### 4. Direct sampling form

The noisy sample at timestep $t$ is a weighted combination of the original clean data and pure Gaussian noise. This lets us sample $x_t$ from $x_0$ in one step (needed for efficient training—we randomly pick a timestep $t$ and apply this formula).

$$
x_t = \sqrt{\bar{\alpha}_t}\,x_0 + \sqrt{1-\bar{\alpha}_t}\,\epsilon
$$
**noisy sample = scaled signal + scaled noise**

#### 5. Forward process definition

$$
q(x_{1:T} \mid x_0) = \prod_{t=1}^{T} q(x_t \mid x_{t-1})
$$

The forward process is a Markov chain, meaning each step depends only on the previous step. This makes the math tractable.

### Reverse diffusion process (removing noise)

The reverse process is a learned stochastic process that gradually removes noise to recover structured data.

**Gaussian noise → iterative denoising → real data**

We begin with pure noise and run the process backwards. The key insight is that if we learn the reverse transitions $p_\theta(x_{t-1} \mid x_t)$ , we can sample from the data distribution.

$$
x_T \sim \mathcal{N}(0,I)
$$

$x_T$: starting pure Gaussian noise
$\mathcal{N}(0,I)$: standard Gaussian distribution  

**Goal:** recover clean sample $x_0$

#### 1. Remove noise at each timestep

We run backwards from $t = T$ down to $t = 1$, and at each step the model removes a small amount of noise.

**The reverse transition**

The distribution of the cleaner sample $x_{t-1}$ given the noisy sample $x_t$ is Gaussian:

$$
p_\theta(x_{t-1} \mid x_t) = \mathcal{N}\!\left(x_{t-1} \mid \mu_\theta(x_t, t), \sigma_t^2 I\right)
$$

- $p_\theta(x_{t-1} \mid x_t)$: probability of moving from noisy $x_t$ to cleaner $x_{t-1}$
- $\mu_\theta(x_t, t)$: mean predicted by the neural network (the "denoised" estimate)
- $\sigma_t^2 I$: variance, usually fixed from the forward process rather than learned

We train the network to predict $\mu_\theta$ or $\epsilon_\theta$, then add a fixed noise scale $\sigma_t$ at each step.


**Computing the cleaner sample**

$$
x_{t-1} = \mu_\theta(x_t, t) + \sigma_t z
$$

- $x_t$: current noisy sample
- $x_{t-1}$: cleaner sample we are solving for
- $\mu_\theta(x_t, t)$: neural network prediction of cleaner signal
- $\sigma_t$: controls how much randomness is added
- $z \sim \mathcal{N}(0, I)$: Gaussian noise

**Cleaner sample:** neural network estimate of the denoised sample + small stochastic noise.

#### 2. Neural network

The network takes the current noisy sample $x_t$, identifies the noise in it, and outputs a cleaner estimate $x_{t-1}$.

$$
x_t = \sqrt{\bar{\alpha}_t}\,x_0 + \sqrt{1-\bar{\alpha}_t}\,\epsilon
$$

- $x_0$: clean data
- $\epsilon$: Gaussian noise we added

We train the network to predict the noise $\epsilon$ (not $x_0$ or $\mu_{t-1}$ directly).

The network outputs $\epsilon_\theta(x_t, t)$; we remove that from $x_t$ to recover the cleaner signal.

#### 3. Reverse process definition

Defines the probability of generating a full sample trajectory from noise to data.

$$
p_\theta(x_{0:T}) = p(x_T)\prod_{t=1}^{T} p_\theta(x_{t-1} \mid x_t)
$$

This is a Markov chain, meaning each step depends only on the current noisy sample. 

#### 4. Training objective

The network is trained on the *forward* process: we take real data $x_0$, add noise to get $x_t$, and train the model to predict the noise we added. The loss is the Mean Squared Error (MSE) between the true noise and the predicted noise.

$$
\mathcal{L} = \mathbb{E}\left[\|\epsilon - \epsilon_\theta(x_t, t)\|^2\right]
$$

### Recover clean data from predicted noise

**Noise prediction → data**

Recall the forward equation: the noisy sample $x_t$ (data at step $t$) equals scaled clean signal plus scaled noise: $x_t = \sqrt{\bar{\alpha}_t}\,x_0 + \sqrt{1-\bar{\alpha}_t}\,\epsilon$, where $x_0$ is the original clean data and $\epsilon$ is the added Gaussian noise.

**Goal:** recover the clean sample $x_0$ from the noisy $x_t$.

#### 1. Rearrange equation to isolate $x_0$

$$
x_0 = \frac{x_t - \sqrt{1-\bar{\alpha}_t}\,\epsilon}{\sqrt{\bar{\alpha}_t}}
$$

#### 2. Replace true noise with predicted noise from the neural network

We do not know the true noise $\epsilon$, but the network predicts it: $\epsilon_\theta(x_t, t)$. Plugging that in gives our estimate of the clean data:

$$
\hat{x}_0 = \frac{x_t - \sqrt{1-\bar{\alpha}_t}\,\epsilon_\theta(x_t, t)}{\sqrt{\bar{\alpha}_t}}
$$

So the estimated clean sample $\hat{x}_0$ is the noisy sample minus the predicted noise, scaled appropriately.

#### 3. Training loss: MSE

The loss trains the network to identify the noise in noisy samples. 

Once it can predict the true noise $\epsilon$ accurately with $\epsilon_\theta(x_t, t)$, it can remove it and recover clean data:

$$
\mathcal{L} = \mathbb{E}_{x_0, \epsilon, t}\left[\|\epsilon - \epsilon_\theta(x_t, t)\|^2\right]
$$ 

### Reverse sampling using predicted noise

At inference we start from pure noise $x_T$ and run backwards. At each step we plug the network's predicted noise $\epsilon_\theta(x_t, t)$ into the reverse-step formula:

$$
x_{t-1} = \frac{1}{\sqrt{\alpha_t}}\left(x_t - \frac{1-\alpha_t}{\sqrt{1-\bar{\alpha}_t}}\epsilon_\theta(x_t, t)\right) + \sigma_t z
$$

This turns the current noisy sample $x_t$ into a cleaner sample $x_{t-1}$. 

The formula has two parts: 
- subtract the predicted noise from $x_t$ to denoise 
- Add a small random draw $\sigma_t z$ for diversity. 

After enough steps, pure noise becomes structured data.
### Summary

- **Forward pass** adds noise to destroy structure: $x_0$ → $x_t$
- **Neural network** learns to predict noise: $\epsilon_\theta(x_t, t)$
- **Reverse process** removes noise to generate data: $x_T$ → $x_0$ 

