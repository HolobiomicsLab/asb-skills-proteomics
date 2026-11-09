---
name: tensorflow-model-instantiation-verification
description: 'Use when a paper''s neural architecture (e.g. SC-GAN: 3D cGAN + self-attention
  + spectral normalization) has been reimplemented and you must verify the model graph
  constructs correctly and matches a reference implementation.'
when_to_use_negative:
- The goal is to train or evaluate synthesis quality (normalized RMS error, PSNR,
  SSIM) — this skill only verifies graph construction, not prediction accuracy
- The model being verified is 2D or a different architecture (e.g. 3D Unet, plain
  3D conditional GAN) whose layer topology differs from SC-GAN's self-attention 3D
  cGAN
- The reference implementation is unavailable, since structural comparison against
  the released SC-GAN code is a required validation step
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3304
tools:
- name: TensorFlow
  role: Deep-learning framework in which the SC-GAN model is implemented and instantiated
    for verification
  repo: https://github.com/Haoyulance/SC-GAN
- name: SC-GAN
  role: The 3D self-attention conditional GAN with spectral normalization whose architecture
    is being verified; reference code released by the authors
  repo: https://github.com/Haoyulance/SC-GAN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:b6a55e4906f7939d85f724cb7b48fb487ed96dc1e06e4acbd1df7a6586564876
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - SC-GAN
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3304
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# tensorflow-model-instantiation-verification

## Summary

A structural verification skill for a reimplemented TensorFlow model: instantiate the full SC-GAN generator/discriminator (3D conditional GAN with self-attention and spectral normalization) at the paper's configuration (batch size 1, 256×256×256) to confirm all layers, attention blocks, and loss terms build without error before any training is attempted.

## When to use

Use when a paper's neural architecture (e.g. SC-GAN: 3D cGAN + self-attention + spectral normalization) has been reimplemented and you must verify the model graph constructs correctly and matches a reference implementation.

## When NOT to use

- The goal is to train or evaluate synthesis quality (normalized RMS error, PSNR, SSIM) — this skill only verifies graph construction, not prediction accuracy
- The model being verified is 2D or a different architecture (e.g. 3D Unet, plain 3D conditional GAN) whose layer topology differs from SC-GAN's self-attention 3D cGAN
- The reference implementation is unavailable, since structural comparison against the released SC-GAN code is a required validation step

## Inputs

- Reimplemented model module (e.g. sc_gan_model.py) defining the 3D cGAN generator, discriminator, and self-attention module in TensorFlow
- Reference repository requirements.txt from https://github.com/Haoyulance/SC-GAN/blob/master/requirements.txt
- Reference SC-GAN source code for structural comparison

## Outputs

- A successfully instantiated SC-GAN model object (batch size = 1, image size 256) whose forward pass runs without error
- A structural comparison confirming generator, discriminator, attention module, and composite loss match the reference implementation

## How to apply

First install the exact environment from the reference repository's requirements.txt (github.com/Haoyulance/SC-GAN; Python 3.6.9 was used in the study) via pip. Implement the 3D conditional GAN generator with the self-attention module placed in both down-sampling and up-sampling paths, implement the discriminator with self-attention, and apply spectral normalization in both networks. Then instantiate the full model at the paper's operational point — batch size = 1, image size 256 — so every 3D convolution, attention block, and the composite loss (adversarial + L1 weight μ=200, brain-area B-rmse weight γ=200, feature matching weight λ=20, L2 regularizers νG=νD=0.001) is constructed and executed on a dummy forward pass. Finally, structurally compare the implemented generator, discriminator, attention module, and loss functions against the released SC-GAN source code. Verification passes if the instantiation runs without error and the architecture matches the reference.

## Related tools

- **TensorFlow** (Deep-learning framework in which the SC-GAN model is implemented and instantiated for verification) — https://github.com/Haoyulance/SC-GAN
- **SC-GAN** (The 3D self-attention conditional GAN with spectral normalization whose architecture is being verified; reference code released by the authors) — https://github.com/Haoyulance/SC-GAN

## Examples

```
python training.py --training_size=1 --gpu=0 --epoches=1 --img_size=256 --data_dir=data/train --modalities=flair_t1w_av45 --logdir=logs
```

## Evaluation signals

- sc_gan_model.py exists and runs without error when the model is instantiated
- Instantiation uses the paper's configuration: batch size = 1 and image size 256 (same for all three dimensions)
- The self-attention module is present in both the down-sampling and up-sampling paths of the generator
- Spectral normalization is applied in both the generator and the discriminator
- Composite loss is constructed with the paper's weights: mu = 200 (L1), gamma = 200 (B-rmse), lambda = 20 (feature matching), nuG = nuD = 0.001 (L2 regularization)
- Architecture structurally matches the released reference SC-GAN implementation

## Limitations

- Instantiation-only verification confirms graph construction but does not validate training stability or synthesis quality; the paper notes computational cost is the main limitation of SC-GAN, requiring lengthy training and heavy GPU memory
- Structural comparison can miss numerical/weight-level differences not visible in layer topology
- Verification is environment-sensitive: the study used Python 3.6.9, so mismatches with other TensorFlow/Python versions may cause construction errors unrelated to architecture correctness

## Evidence

- [methods] Set up a TensorFlow environment matching the reference repository's requirements.txt from github.com/Haoyulance/SC-GAN.: [evidence span withheld — anchor status: not_found]
- [methods] Instantiate the full model (batch size = 1, image size 256) to verify that all layers and attention blocks are constructed correctly.: [evidence span withheld — anchor status: not_found]
