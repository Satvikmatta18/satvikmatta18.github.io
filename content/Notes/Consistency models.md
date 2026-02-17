## [Consistency models](https://arxiv.org/abs/2303.01469)

**Idea:** Learn a function that maps **any** noisy point $x_t$ **directly to the clean sample** $x_0$ in **one step**:

$$f_\theta(x_t, t) = x_0$$

- **Trajectory property:** Every point on a diffusion trajectory (e.g. DDIM) corresponds to the same origin $x_0$. So $x_t$, $x_{t-\Delta t}$, etc. on the same path should all map to the same $x_0$.
- **Why it works:** DDIM follows an ODE (no randomness). So the trajectory is fixed given $x_0$ and $\epsilon$: $x_t = \sqrt{\bar{\alpha}_t}\,x_0 + \sqrt{1-\bar{\alpha}_t}\,\epsilon$. All points on that trajectory share the same $x_0$ and $\epsilon$.
- The consistency model learns this **inverse mapping** $x_t \mapsto x_0$. At inference we can go from noise to data in one step (or a few steps).

### Consistency distillation (CD)

We have a pretrained diffusion model (teacher $\hat{f}_\theta$). We train a consistency model (student $f_\theta$) so that nearby points on the same trajectory map to the same $x_0$. Teacher is usually an EMA of the student.

**Consistency condition:** For two points on the same trajectory (e.g. $x_t$ and $x_{t+\Delta t}$), we want the same output:

$$f_\theta(x_{t+\Delta t}, t+\Delta t) \approx \hat{f}_\theta(x_t, t)$$

**Loss:** Minimize the distance between student at the later point and teacher at the earlier point:

$$\mathcal{L}_{\text{CD}} = \mathbb{E}\left[ d\bigl( f_\theta(x_{t+\Delta t}, t+\Delta t),\, \hat{f}_\theta(x_t, t) \bigr) \right]$$

$d$ is a distance (e.g. $\ell_2$). So the student is trained to match the teacher’s “origin” along the trajectory.

### Consistency training (CT)

We train the consistency model from scratch (no teacher). We still use the fact that each trajectory has a single starting point $x_0$. We form noisy $x_t$ from clean $x_0$ and train the model to predict $x_0$.

**Consistency condition:** For any $x_t$ on the trajectory of $x_0$, the model should output $x_0$:

$$f_\theta(x_t, t) \approx x_0$$

**Loss:** Minimize the distance between the model output and the true clean sample:

$$\mathcal{L}_{\text{CT}} = \mathbb{E}_{x_0, \epsilon, t}\left[ d\bigl( f_\theta(x_t, t),\, x_0 \bigr) \right]$$

So we supervise “noisy input → clean output” directly. In practice we often use a target network or consistency between nearby $t$ as well.

