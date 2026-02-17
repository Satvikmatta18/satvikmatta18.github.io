## Scaling up and down

Generating high-quality images with diffusion is expensive. Several ideas make it more efficient or better conditioned.

### Cascaded diffusion

Instead of generating a large image in one shot, we use **multiple stages**: each stage generates a higher-resolution image conditioned on a lower-resolution one.

Example: $64 \times 64 \to 256 \times 256 \to 1024 \times 1024$. Each stage runs a diffusion model that upsamples and adds detail. The model learns a conditional distribution:

$$p_\theta(x_{\text{high}} \mid x_{\text{low}})$$

So we go from small to large in steps; each stage conditions on the previous output.

### [Noise conditioning augmentation](https://arxiv.org/abs/2106.15282)

Super-resolution models often **overfit to clean inputs** seen during training. At test time, inputs may be slightly noisy or imperfect.

**Noise conditioning augmentation:** During training we perturb the conditioning input so the model sees a range of qualities.

- **Low-res side:** Add Gaussian noise: $x_{\text{low}} + \epsilon$, $\epsilon \sim \mathcal{N}(0, \sigma^2 I)$.
- **High-res side:** Apply **Gaussian blur** to the target.

So the model learns to handle noisy or blurred conditioning and is more robust at test time.

**Two conditioning strategies at inference:**

1. **Truncated conditioning:** Stop the low-resolution diffusion **early** at some step $s$, so the conditioning image passed to the next stage remains **slightly noisy**. That mimics imperfection and can match training conditions better.
2. **Non-truncated:** Run low-res diffusion until fully denoised, then **add noise again** to the result before feeding it as condition. That simulates realistic imperfection (e.g. compression, sensor noise) so the next stage sees something other than “perfect” low-res.

### [UnCLIP](https://arxiv.org/abs/2204.06125)

**CLIP** maps text and images into a **shared embedding space**. **UnCLIP** uses that idea for generation: first go from text to an embedding, then from embedding to image.

UnCLIP trains two models:

- **Prior model** $p_\theta(z_i \mid z_t)$: predicts **image embedding** $z_i$ from **text embedding** $z_t$ (e.g. from CLIP). So we get a “target” image embedding for the text.
- **Decoder** $p_\theta(x \mid z_i)$: generates **images** from the image embedding $z_i$ (e.g. a diffusion model in pixel or latent space).

**Flow:** Text $\to z_t \to z_i$ (prior) $\to$ image (decoder). So text is turned into an image embedding, then the decoder produces the image.

### [Imagen](https://arxiv.org/abs/2205.11487)

**Imagen** follows the same high-level logic as UnCLIP (text $\to$ embedding $\to$ image) but uses a **large language model (LLM)** as the **text encoder** instead of (or in addition to) a CLIP-style encoder.

The LLM gives a strong text embedding, then a prior and a decoder map that to an image embedding and finally to the image. 
