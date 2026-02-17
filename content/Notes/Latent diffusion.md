## [Latent diffusion (latent variable space)](https://arxiv.org/abs/2112.10752)

**Latent Diffusion Models (LDM)** run diffusion in **latent space** instead of pixel space. That makes training cheaper, inference faster, and saves memory.

Images mix **perceptual detail** (fine pixel structure) and **semantic structure** (high-level meaning). Instead of running diffusion over millions of pixels, we compress to a smaller representation and run diffusion there.
### 1. Autoencoder: compress to latent space

The **perceptual compression** step uses an autoencoder. An **encoder** $\mathcal{E}$ compresses the image $x$ into a smaller **2D latent** $z$ (a grid of vectors, like a lower-resolution “image” in feature space). A **decoder** $\mathcal{D}$ reconstructs the image from $z$:

So we work in the space of $z$ instead of pixels. To keep the latent space stable, the paper uses **regularization** during autoencoder training:

- **KL-reg:** A small **KL penalty** toward a standard normal distribution on the latent (like in a VAE). That discourages latents from having very large variance and keeps the space bounded.
- **VQ-reg:** A **vector quantization** layer inside the decoder (as in VQVAE): latents are mapped to discrete codes, then the decoder absorbs that step. That also constrains the latent space and can give cleaner reconstructions.

### 2. Add and remove noise in latent space

The **forward pass** adds noise to the latent $z$ (e.g. $z_t = \sqrt{\bar{\alpha}_t}\,z + \sqrt{1-\bar{\alpha}_t}\,\epsilon$). We train a neural network to predict that noise in latent space. The **reverse pass** denoises in latent space step by step, same idea as DDPM/DDIM but on $z$ instead of $x$.
### 3. Conditioning (e.g. text)

We condition on an input such as text $c$. A **conditioning encoder** $\tau$ (e.g. a text encoder) gives an embedding $r = \tau(c)$. The model is trained so that the generated latent matches $c$; in architectures like DiT, $r$ is used to guide generation. So we strengthen the link between text and image, in a similar spirit to CLIP.

### 4. Decode latent to image

After denoising in latent space we get a clean latent $z_0$. We decode it to get the final image: $\hat{x} = \mathcal{D}(z_0)$. 

Full pipeline: encode $x \to z$, (optionally condition on $c$), run diffusion in latent space, then decode $z_0 \to \hat{x}$.
