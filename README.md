# Auralis Nova: AI Product Banner Generation with ComfyUI (Flux)

## Project Overview

This project generates a cinematic advertising image for a fictional premium wireless headphone, **Auralis Nova**, using text-to-image generation in ComfyUI. The image is composed with the product on the right and a large clean dark area on the left, so headline text or a logo can be added later. The base image is then upscaled to a final **1920 x 1080** (Full HD) output.

**Goals**
- Generate a high-quality, photorealistic product visual from a text prompt.
- Leave clean negative space for marketing copy.
- Deliver a sharp 1920 x 1080 final image.

## Repository Structure

```
.
├── README.md
├── workflow/
│   └── assessment2_auralis_nova_workflow.json
├── output/
│   ├── auralis_nova_base_00001_.png
│   └── auralis_nova_final_1920x1080_00001_.png
└── screenshots/
    └── workflow_graph.png
```

## Dataset Structure

This is a text-to-image task, so no input dataset is used. The only input is the text prompt below.

| Item | Details |
|---|---|
| Input | Text prompt (no reference image) |
| Base output | 1536 x 864 px |
| Final output | 1920 x 1080 px |

## Model Used

| Component | File |
|---|---|
| Diffusion model | `flux1-dev.safetensors` (FLUX.1 [dev], weight dtype fp8_e4m3fn) |
| Text encoders | `clip_l.safetensors`, `t5xxl_fp8_e4m3fn.safetensors` |
| VAE | `ae.safetensors` |
| Upscaler | `4x-UltraSharp.pth` |

## Installation Steps

1. Install [ComfyUI](https://github.com/comfyanonymous/ComfyUI) and run it once to confirm it starts.
2. Download the models and place them in these folders:

   | File | Folder |
   |---|---|
   | `flux1-dev.safetensors` | `ComfyUI/models/diffusion_models/` |
   | `clip_l.safetensors` | `ComfyUI/models/clip/` |
   | `t5xxl_fp8_e4m3fn.safetensors` | `ComfyUI/models/clip/` |
   | `ae.safetensors` | `ComfyUI/models/vae/` |
   | `4x-UltraSharp.pth` | `ComfyUI/models/upscale_models/` |

   `flux1-dev` is a gated model on Hugging Face. Log in and accept the licence on the model page before downloading.
3. Restart ComfyUI (or press `R` to refresh the model lists).

No custom nodes are required. All nodes used are built into ComfyUI.

## Training Instructions

Not applicable. No model was trained or fine-tuned. All models are used as released.

## Prediction Instructions

1. Open ComfyUI in the browser.
2. Drag `workflow/assessment2_auralis_nova_workflow.json` onto the canvas.
3. Confirm the model names in the loader nodes match your files.
4. Click **Run**.
5. Results are saved in `ComfyUI/output/`:
   - `auralis_nova_base_*.png` is the 1536 x 864 base image.
   - `auralis_nova_final_1920x1080_*.png` is the upscaled final image.

To get a different composition, change the `seed` in the KSampler node.

## Prompt Used

```
Cinematic advertising photograph of a single pair of premium matte-black wireless over-ear headphones with brushed champagne-gold metal accents and soft leather ear cushions, floating at the right side of the frame, slightly angled to show the sculpted ear cup and slim headband, dramatic rim lighting in teal and warm amber, subtle flowing ribbons of light curving around the headphones suggesting immersive sound, dark charcoal gradient studio background with soft volumetric haze, shallow depth of field, ultra-detailed materials, sleek modern industrial design, 85mm lens, high-end commercial product photography, large clean empty dark space on the left half of the frame
```

Flux does not use a negative prompt, so the negative input is a zeroed-out copy of the positive conditioning.

## Image Preprocessing and Post-processing

There is no input image to preprocess. The pipeline is:

1. **Empty latent** at 1536 x 864 (16:9, both sides divisible by 16 as Flux requires).
2. **Generation:** FluxGuidance 3.5, KSampler with Euler, simple scheduler, 28 steps, cfg 1.0, denoise 1.0, seed 20260101.
3. **VAE decode** to a 1536 x 864 base image (saved).
4. **Upscale 4x** with 4x-UltraSharp, giving 6144 x 3456.
5. **Downscale by 0.3125** with Lanczos, giving exactly 1920 x 1080. Upscaling first and then downscaling gives a cleaner, sharper result than a direct 1.25x resize.
6. **Save** the final image.

## Evaluation Results

Evaluation is visual, as there is no ground truth image.

| Check | Result |
|---|---|
| Product matches the description (matte black, champagne-gold accents) | Yes |
| Product placed on the right, clean space on the left | Yes |
| Lighting (teal and amber rim light) | Yes |
| Final resolution | 1920 x 1080 |
| Artefacts or distortions | None observed |

Sample outputs are in the `output/` folder.

## Known Limitations

- Generated headphones are not a real product, and fine details such as the headband join or logos can look inconsistent.
- Flux can place the product off-position or fill the empty left area with light effects. Changing the seed usually fixes this.
- Flux cannot render brand text or logos reliably. Add the "Auralis Nova" name in a design tool afterwards.
- The 4x-UltraSharp upscaler adds sharpening, which can look slightly artificial on soft gradients.
- Results depend on the seed, so outputs are only reproducible with the same seed, models and ComfyUI version.

