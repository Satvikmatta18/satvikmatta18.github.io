## [Classifier-guided diffusion](https://arxiv.org/abs/2105.05233)

Train a **classifier** on noisy images and use it to **guide** generation toward a chosen class.
- **Classifier:** 
	- Input = noisy image $x_t$; 
	- output = probability of class $y$, i.e. $p(y \mid x_t)$.
- **Example:** 
	- Noisy image of a dog; 
	- we want the process to move toward "dog." 
	- The classifier says how confident it is that $x_t$ is class $y$.

**Classifier gradient:** $\nabla_{x_t} \log p(y \mid x_t)$
- Tells us how to change the image so the classifier is more confident it is class $y$.
- It is a direction in image space (which way to nudge $x_t$).

**Original score:** $s_\theta(x_t, t) = \nabla_{x_t} \log p_t(x_t)$ (points toward real data).

**Guided score:** add the classifier gradient, scaled by $w$:

$$s_\theta(x_t, t, y) = s_\theta(x_t, t) + w\,\nabla_{x_t} \log p(y \mid x_t)$$

- $w$: **guidance scale.** 
- $w = 0$ = no guidance
- $w = 1$ = standard
- Larger $w$ = stronger push toward class $y$ (can over-guidance).

**Reverse step:** use the guided score in the usual reverse sampling:

$$x_{t-1} = x_t + \sigma_t^2\, s_\theta(x_t, t, y) + \sigma_t z, \quad z \sim \mathcal{N}(0, I)$$

So at each step we move in the direction of the score plus the classifier gradient, so samples tend toward class $y$.

**Cons:** Need to train a classifier on noisy images; extra compute; classifier gradients can be unstable.
