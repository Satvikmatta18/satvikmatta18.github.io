
## [DDPM (Denoising Diffusion Probabilistic Models)](DDPM.md)
Learns to generate by gradually removing noise.
- **Training:** add noise to real data step by step until it's pure Gaussian; model learns to reverse it.
- **Generation:** start from random noise, denoise iteratively.
- Each reverse step samples from a Gaussian → **stochastic**; needs many steps (slow).
## [DDIM (Denoising Diffusion Implicit Models)](DDIM.md)
Same model as DDPM; reverse is an ODE (no randomness).
- Fixed direction each step → **deterministic**; same noise → same output.
- Can skip timesteps (e.g. 1000 → 800 → 600) → much faster.
## [Noise Conditioned Score Function (NCSN)](NCSN.md)
Predicts the **score** (direction toward more realistic data), not the noise.
- Generation: repeatedly move the sample in that direction (Langevin dynamics).
- Conditioned on noise level for stable training. Same idea as diffusion, different framing.
## [Variance schedule improvements](Variance%20schedule%20improvements.md)
Controls how much noise is added at each timestep.
- Options: linear, cosine, log-SNR, Karras, etc.
- Affects: training stability, sample quality, number of steps.
- Modern schedules keep signal longer, often better quality.
## [Reverse variance parametrization](Reverse%20variance%20parametrization.md)
Reverse step = Gaussian with mean (denoising direction) + variance.
- **Fixed** (from forward schedule): stable, less flexible.
- **Learned** (e.g. interpolated): often better samples, fewer steps.
## [Classifier-guided diffusion](Classifier-guided%20diffusion.md)
Separate classifier on noisy data; its gradient is added to the denoising direction.
- Pushes samples toward a target class.
- **Downside:** need to train and run a classifier on noisy inputs.
## [Classifier-free guidance](Classifier-free%20guidance.md)
One model does conditional and unconditional prediction (condition dropped at random in training).
- At inference: combine both predictions to steer generation (no classifier).
- Used in Stable Diffusion, SDXL, Imagen.
## [Consistency models](Consistency%20models.md)
Map any point on a trajectory (e.g. $x_t$) straight to clean $x_0$.
- One or a few steps instead of many → much faster.
- Training: distill a diffusion model (CD) or train from scratch with consistency loss (CT).
## [Latent diffusion (latent variable space)](Latent%20diffusion.md)
Diffusion in autoencoder latent space, not pixels.
- **Pipeline:** compress → diffuse (and condition, e.g. text) in latent space → decode.
- Cheaper, less memory, scales better. Used in Stable Diffusion.
## [Scaling up and down](Scaling%20up%20and%20down.md)
- **Cascaded diffusion:** stages (e.g. 64→256→1024), each adds resolution.
- **Noise conditioning augmentation:** perturb conditioning in training → robust to noisy inputs.
- **UnCLIP:** text → image embedding (prior) → image (decoder).
- **Imagen-style:** same but LLM as text encoder → stronger text–image alignment.
## [Model Architectures](Model%20Architectures.md)
- **[U-Net](https://arxiv.org/abs/1505.04597):** encoder–decoder + skip connections; down → structure, up → resolution, skips → detail. Standard diffusion backbone.
- **[ControlNet](Model%20Architectures.md#controlnet):** structural conditioning (edges, depth, pose, segmentation); trainable copy of encoder + zero-init convs; pretrained model frozen.
- **[Diffusion Transformer (DiT)](Model%20Architectures.md#diffusion-transformer-dit):** transformer on latent tokens; better scaling, global coherence; e.g. SD3, Flux.

