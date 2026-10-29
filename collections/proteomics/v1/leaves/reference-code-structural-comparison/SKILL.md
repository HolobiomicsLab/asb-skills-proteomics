---
name: reference-code-structural-comparison
description: Use when re-implementing a published network (e.g., SC-GAN) whose authors
  released source code, and you must verify your reconstructed generator, discriminator,
  self-attention module, and loss function structurally match the reference before
  reuse.
when_to_use_negative:
- You only need to run the published model for inference or training rather than verify
  a reimplementation — use the repository's training.py/testing.py scripts directly
  instead.
- The article provides no released source code or architecture-level detail, leaving
  nothing to structurally compare against.
- Your goal is quantitative evaluation of synthesis quality (nRMSE, PSNR, SSIM) rather
  than architectural fidelity.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3391
tools:
- name: SC-GAN
  role: Reference implementation whose generator, discriminator, attention module,
    and losses are the comparison target for the reimplementation
  repo: https://github.com/Haoyulance/SC-GAN
- name: TensorFlow
  role: Framework used to implement and instantiate the reconstructed 3D conditional
    GAN modules for structural verification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:88cfe7d75c2e20dd2da05a5b39fd0b84c8ebaab760234108a297d97c03efd1cb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - SC-GAN
  - TensorFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# reference-code-structural-comparison

## Summary

Reconstruct a published deep-learning model (e.g., the SC-GAN 3D conditional GAN with self-attention and spectral normalization) in code and verify it by structural comparison against the authors' released reference implementation, including the composite loss with weights mu=200, gamma=200, lambda=20, nuG=0.001, nuD=0.001.

## When to use

Use when re-implementing a published network (e.g., SC-GAN) whose authors released source code, and you must verify your reconstructed generator, discriminator, self-attention module, and loss function structurally match the reference before reuse.

## When NOT to use

- You only need to run the published model for inference or training rather than verify a reimplementation — use the repository's training.py/testing.py scripts directly instead.
- The article provides no released source code or architecture-level detail, leaving nothing to structurally compare against.
- Your goal is quantitative evaluation of synthesis quality (nRMSE, PSNR, SSIM) rather than architectural fidelity.

## Inputs

- Reference repository source code (github.com/Haoyulance/SC-GAN) including requirements.txt
- Published architecture description (3D cGAN body, 3D self-attention module, spectral normalization, feature matching and B-rmse losses)
- Named hyperparameter defaults from the README training/testing configuration tables

## Outputs

- A reconstructed model module (e.g., sc_gan_model.py) containing generator, discriminator, self-attention module, and composite loss
- A structural comparison of implemented vs. reference layers, attention blocks, and loss terms
- A runnable instantiation of the full model (batch size = 1, image size 256) as a validation artifact

## How to apply

Set up the environment first using the reference repository's requirements (github.com/Haoyulance/SC-GAN/blob/master/requirements.txt, Python 3.6.9). Implement the 3D conditional GAN generator as a Python module, placing the 3D self-attention module in both the down-sampling and up-sampling paths, apply spectral normalization in both generator and discriminator, and build the composite objective combining cGAN adversarial loss, L1 loss (l1_weight=200), brain-area B-rmse loss (B_weight=200), feature matching loss (lambda=20), and L2 regularization (g_reg=0.001, d_reg=0.001). Instantiate the full model with batch size = 1 and img_size = 256 to confirm all layers and attention blocks construct without error. Finally, structurally compare the implemented generator, discriminator, attention module, and loss functions layer-by-layer against the released reference source code, checking default hyperparameters (lr=0.001, epoches=120).

## Related tools

- **SC-GAN** (Reference implementation whose generator, discriminator, attention module, and losses are the comparison target for the reimplementation) — https://github.com/Haoyulance/SC-GAN
- **TensorFlow** (Framework used to implement and instantiate the reconstructed 3D conditional GAN modules for structural verification)

## Examples

```
python training.py --trainig_size= --gpu= --epoches= --img_size=256 --data_dir= --modalities=flair_t1w_av45 --logdir= --lr=0.001 --g_reg=0.001 --d_reg=0.001 --l1_weight=200 --B_weight=200
```

## Evaluation signals

- sc_gan_model.py exists and runs without error when the model is instantiated (batch size = 1, img_size = 256).
- Generator and discriminator both contain 3D self-attention blocks placed in the down-sampling and up-sampling paths, per the paper's architecture description.
- Spectral normalization is present in both the generator and the discriminator.
- Composite loss includes adversarial, L1 (weight 200), brain-area B-rmse (weight 200), feature matching (lambda = 20), and L2 regularization terms with nuG = nuD = 0.001.
- Layer-by-layer structural diff of the implementation against github.com/Haoyulance/SC-GAN shows no missing or extra modules.

## Limitations

- SC-GAN has high computational cost: lengthy training and heavy GPU memory requirements, which constrains how far the instantiated model can be exercised during validation.
- The README's install instruction contains a typo ('pip install requirement.txt' rather than requirements.txt), so environment setup may require correction.
- Structural comparison verifies architecture, not learned weights or synthesis quality; quantitative validation needs nRMSE, PSNR, and SSIM evaluation on held-out data (e.g., ADNI-3).

## Evidence
