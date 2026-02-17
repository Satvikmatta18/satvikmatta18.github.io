## [Classifier-free guidance](https://arxiv.org/abs/2207.12598)

We steer generation toward a condition $y$ (e.g. a class or text) **without** a separate classifier. One diffusion model is trained to predict noise both **with** and **without** the condition.

**Training:**
- **With conditioning:** $\epsilon_\theta(x_t, t, y)$ — predicts noise given noisy $x_t$, timestep $t$, and condition $y$.
- **Without conditioning:** $\epsilon_\theta(x_t, t)$ — same model with $y$ omitted (e.g. use a null token).
- **Random drop:** During training we randomly drop $y$ some of the time. So one model learns both conditional and unconditional noise prediction.

**Guidance direction:** $\epsilon_{\text{cond}} - \epsilon_{\text{uncond}}$

- $\epsilon_{\text{cond}} = \epsilon_\theta(x_t, t, y)$, $\epsilon_{\text{uncond}} = \epsilon_\theta(x_t, t)$.
- This **difference** is the direction in noise space that pushes the sample toward the target condition $y$. Adding it to the prediction strengthens the effect of $y$.

**Combined prediction at inference:**

$$\epsilon = \epsilon_{\text{uncond}} + w\,(\epsilon_{\text{cond}} - \epsilon_{\text{uncond}})$$

or equivalently:

$$\epsilon = (1-w)\,\epsilon_{\text{uncond}} + w\,\epsilon_{\text{cond}}$$

- $w$: **guidance scale.** $w = 1$ → use only conditional
- $w = 0$ → only unconditional
- $w > 1$ → stronger push toward $y$ (over-guidance).

 So we blend the two predictions; larger $w$ moves the update more toward the conditional direction and steers the diffusion toward the target condition.
