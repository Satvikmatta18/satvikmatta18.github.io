### Model Architectures 
### [U-Net](https://arxiv.org/abs/1505.04597)

A **U-Net** maps an input image (or latent) to an output image of the same size. It has a **downsampling path** (encoder), an **upsampling path** (decoder), and **skip connections** between them.
### 1. Downsampling path (encoder)

Compresses spatial information and learns high-level structure.

**Per block:** Convolution → ReLU → Convolution → ReLU → MaxPool

- **Convolution:** Applies learned filters to detect features (edges, textures, etc.).
- **ReLU:** Nonlinearity; positive values stay, negatives become zero. Lets the model learn non-linear patterns.
- **MaxPool:** Downsamples by a factor of 2 in each spatial dimension; keeps the strongest activations in each local region.

**Effect per step:** $(h \times w, c) \to (h/2 \times w/2, 2c)$. Resolution goes down, channel depth goes up. The model builds more abstract features over a smaller grid.

### 2. Upsampling path (decoder)

Restores spatial resolution and produces the output.

**Per block:** Upsample → Convolution (and often Convolution again)

- **Upsample:** Increases resolution (e.g. by factor 2) so we get back to the original size.
- **Convolution:** Refines features after upsampling.

**Effect per step:** $(h/2 \times w/2, c) \to (h \times w, c/2)$. Resolution goes up, channels can go down as we move toward the output.

### 3. Skip connections

$$\text{skip} = \text{concat}(x_{\text{down}}, x_{\text{up}})$$

- **Concat:** Concatenate encoder features and decoder features **along the channel dimension**. So the decoder sees both its own upsampled features and the encoder’s features at the same resolution.
- **Why:** The encoder keeps fine-grained, local detail; the decoder needs that to reconstruct precise structure. Skip connections carry that information and help gradients flow.

### Summary

Downsample (encoder) to compress and learn structure → upsample (decoder) to restore resolution → combine encoder and decoder at each level via skip connections so the output keeps both high-level structure and fine detail.


### [ControlNet](https://arxiv.org/abs/2302.05543)

**ControlNet** lets diffusion models take **structural conditioning** (e.g. edges, depth maps, poses, segmentation). It adds a **trainable copy** of the original UNet encoder blocks and connects it with **zero-initialized convolutions**, so the condition can steer generation without changing the pretrained model at the start.
**1. Original block (frozen)**

$$y = f(x; \theta)$$
- $x$: input feature map  
- $f$: UNet block  
- $\theta$: pretrained weights (**frozen**)  
- $y$: output feature map  

The pretrained diffusion UNet stays fixed.

**2. Trainable copy**

Clone the same block with **trainable** weights $\theta_c$:

$$y_c = f(x_c; \theta_c)$$

- $\theta_c$: trainable copy of the block  
- $y_c$: output of the ControlNet branch  

This branch will process the conditioning.

**3. Conditioning input**

$c$: **condition map** (e.g. edges, depth, pose, segmentation). We inject $c$ into the trainable copy branch so that branch sees both the usual features and the condition.

-
**4. Zero-initialized convolutions**

Two **1×1 convolutions** $z_1$, $z_2$ connect the condition to the branch and the branch to the main path. A 1×1 conv is a linear map across channels (no spatial mixing).

**Weights of $z_1$, $z_2$ are initialized to zero.** So at the start of training:

$$y_{\text{control}} = z_2\bigl(f(x + z_1(c); \theta_c)\bigr) = 0$$

because $z_2$ is zero. The main UNet output is unchanged; the pretrained model is preserved. As we train, $z_1$, $z_2$, and $\theta_c$ learn to add a conditioned correction.

**5. Combine both branches**

$$y_{\text{final}} = y + y_{\text{control}} = f(x; \theta) + y_{\text{control}}$$

We add the **original UNet output** and the **ControlNet output**. 
So the final feature is the pretrained prediction plus a learned, condition-dependent adjustment.

### [Diffusion Transformer (DiT)](https://arxiv.org/abs/2212.09748)

A **diffusion model built with a Transformer**, run in **latent space** (e.g. after an autoencoder). The latent is turned into a sequence of tokens, then a Transformer with conditioning predicts the noise.

**1. Input: latent representation**

Input is a **noisy latent** $z_t$ at timestep $t$ (compressed image from the autoencoder).

- Shape: $(h, w, c)$
- $h \times w$: spatial size (height × width) of the latent
- $c$: number of channels (feature depth)

So we have a 3D tensor in “image” form before turning it into tokens.

**2. Patchify: latent → tokens**

Split the latent into **patches** of size $p \times p$. Each patch is one **token**.

- Number of tokens: $n = hw / p^2$
- Each token is a vector of dimension $d$ (embedding size after a linear projection)
- Latent becomes a **sequence** of token embeddings: $[x_1, x_2, \ldots, x_n]$, with $x_i \in \mathbb{R}^d$

So we go from a 2D grid $(h, w, c)$ to a 1D sequence of $n$ vectors of dimension $d$.

**3. Conditioning**

Condition on **timestep** $t$ and **class or text** $c$. Turn both into embeddings and add them:

$$e = e_t + e_c$$

- $e_t$: timestep embedding  
- $e_c$: conditioning embedding (class or text)  
- $e$: combined conditioning used later (e.g. in AdaLN)



**4. Adapter Layer Norm (AdaLN-Zero)**

In a standard Transformer, **LayerNorm** normalizes each token so activations stay in a reasonable range and training is stable.

Here we want **timestep** and **conditioning** (e.g. class or text) to affect how the model behaves. **Adaptive Layer Norm (AdaLN)** does that by making the normalization depend on the conditioning.

$$\text{AdaLN}(x) = \gamma(e)\, \text{LayerNorm}(x) + \beta(e)$$

- **LayerNorm$(x)$:** For each token, subtract the mean and divide by the standard deviation (over the feature dimension). So each token has zero mean and unit variance; this is standard in Transformers.
- **$\gamma(e)$, $\beta(e)$:** Scale and shift **computed from** the conditioning $e$ (timestep + class/text embedding). So different $t$ or different $c$ give different normalization, and the model can change behavior per timestep and per condition.
- **AdaLN-Zero:** $\gamma$ and $\beta$ are **initialized so that** $\gamma \approx 1$ and $\beta \approx 0$ at the start. Then $\text{AdaLN}(x) \approx \text{LayerNorm}(x)$ at initialization—the block is close to a standard LayerNorm and does not distort the signal.

**5. Transformer**

Standard **Transformer blocks**: self-attention (Q, K, V over the token sequence) plus feed-forward layers. The model learns relationships between patches (tokens). Blocks use the adaptive layer norm above so conditioning affects the whole block.

**6. Output: noise prediction**

The Transformer outputs a **noise prediction** $\epsilon_\theta(z_t, t, c)$ in the same shape as the latent (e.g. one vector per token, then reshaped to $(h, w, c)$). 

We use this in the usual diffusion update (e.g. DDPM or DDIM) in latent space. After denoising we get a clean latent $z_0$; a **decoder** (e.g. from the autoencoder) turns $z_0$ into the final image.


**Modern DiT:** Recent DiT-style models often use **more blocks**, **better conditioning** (e.g. cross-attention or more flexible AdaLN), and **scaling in model size and data**. 

Some also mix in **spatial attention** or **multi-resolution** tokens. 
The core idea stays: patchify latent → condition on $t$ and $c$ → Transformer with adaptive norm → predict noise → decode to image.