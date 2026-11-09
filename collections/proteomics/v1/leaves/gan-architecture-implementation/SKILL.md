---
name: gan-architecture-implementation
description: Use when a task requires reconstructing or coding a 3D conditional GAN
  (generator, discriminator, self-attention blocks, spectral normalization) and its
  composite loss from an architecture description, e.g., MRI/PET synthesis.
when_to_use_negative:
- The task is 2D slice-wise synthesis — the article shows 2D conditional GAN has 18–29%
  higher prediction error than SC-GAN on 3D volumes, but if only 2D data exist this
  3D architecture is unnecessary.
- A pretrained or off-the-shelf model suffices and no architectural reconstruction
  or modification is required.
- The target task is conventional regression (e.g., 3D U-ET-style segmentation/synthesis)
  where GAN training instability cannot be tolerated — SC-GAN requires heavy GPU memory
  and lengthy training.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_3384
tools:
- name: SC-GAN
  role: Reference implementation of the 3D self-attention conditional GAN with spectral
    normalization being reconstructed
  repo: https://github.com/Haoyulance/SC-GAN
- name: TensorFlow
  role: Deep-learning framework in which the generator, discriminator, self-attention
    module, and loss functions are implemented
- name: 3D conditional GAN
  role: Baseline 3D cGAN body that SC-GAN extends with self-attention and spectral
    normalization
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:d12b36761d89a16a47163f9fa6cf37fea059ef52ff166fc9588d51fd7a45121b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 3D conditional GAN
  - SC-GAN
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_3384
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gan-architecture-implementation

## Summary

Implements the SC-GAN architecture: a 3D conditional GAN generator and discriminator, each with a 3D self-attention module and spectral normalization, plus the composite loss (adversarial cGAN, L1 μ=200, feature matching λ, brain-area RMSE γ=200, and L2 regularization νG=νD=0.001) for multimodal neuroimaging synthesis.

## When to use

Use when a task requires reconstructing or coding a 3D conditional GAN (generator, discriminator, self-attention blocks, spectral normalization) and its composite loss from an architecture description, e.g., MRI/PET synthesis.

## When NOT to use

- The task is 2D slice-wise synthesis — the article shows 2D conditional GAN has 18–29% higher prediction error than SC-GAN on 3D volumes, but if only 2D data exist this 3D architecture is unnecessary.
- A pretrained or off-the-shelf model suffices and no architectural reconstruction or modification is required.
- The target task is conventional regression (e.g., 3D U-ET-style segmentation/synthesis) where GAN training instability cannot be tolerated — SC-GAN requires heavy GPU memory and lengthy training.

## Inputs

- 3D multimodal neuroimaging volumes (e.g., T1w and FLAIR MRI .nii.gz) organized as data/train|val/test/<subject>/modality_1.nii.gz ... target.nii.gz
- Architecture description from the SC-GAN paper (3D cGAN body, 3D self-attention module, spectral normalization)
- requirements.txt from https://github.com/Haoyulance/SC-GAN/blob/master/requirements.txt

## Outputs

- sc_gan_model.py defining generator, discriminator, self-attention module, and composite loss
- Instantiable TensorFlow model (batch size 1, image size 256) producing synthesized target modality volumes (e.g., PET av45, FA, MD maps)
- Trained checkpoint via training.py for testing.py synthesis

## How to apply

Set up a TensorFlow environment matching the reference repository's requirements.txt (Python 3.6.9) from github.com/Haoyulance/SC-GAN. Implement the 3D conditional GAN generator as a Python module, placing the 3D self-attention module in both the down-sampling and up-sampling paths, and apply spectral normalization in both generator and discriminator. Assemble the composite objective: cGAN adversarial loss plus L1 loss (l1_weight μ=200), feature matching loss (λ), and brain-area RMSE loss (B_weight γ=200), with L2 regularization on the generator (g_reg=0.001) and discriminator (d_reg=0.001). Instantiate the full model with batch size 1 and image size 256 (same for all three dimensions) to verify all layers and attention blocks construct without error, then structurally diff the generator, discriminator, attention module, and loss functions against the released SC-GAN source. The ablation study indicates adversarial learning and the self-attention module are the major performance contributors, followed by spectral normalization, so these components must not be omitted.

## Related tools

- **SC-GAN** (Reference implementation of the 3D self-attention conditional GAN with spectral normalization being reconstructed) — https://github.com/Haoyulance/SC-GAN
- **TensorFlow** (Deep-learning framework in which the generator, discriminator, self-attention module, and loss functions are implemented)
- **3D conditional GAN** (Baseline 3D cGAN body that SC-GAN extends with self-attention and spectral normalization)

## Examples

```
python training.py --training_size=100 --gpu=0 --epoches=120 --img_size=256 --data_dir=data/train --modalities=flair_t1w_av45 --logdir=logs --lr=0.001 --g_reg=0.001 --d_reg=0.001 --l1_weight=200 --B_weight=200
```

## Evaluation signals

- sc_gan_model.py exists and the model instantiates without error at batch size 1, image size 256.
- Structural comparison of the implemented generator, discriminator, self-attention module, and loss functions against the reference SC-GAN source code matches.
- Loss configuration matches the paper defaults: l1_weight=200, B_weight=200, g_reg=0.001, d_reg=0.001, epoches=120.
- Training run on T1w/FLAIR inputs produces synthesized target volumes evaluated by normalized RMS error, peak SNR, and structural similarity.
- Ablation variants (without self-attention, without adversarial learning, without brain-area RMSE loss) can be toggled, confirming each component is modular in the code.

## Limitations

- Computational cost is the main limitation of SC-GAN; it requires lengthy training and heavy GPU memory.
- Normalizing target quantitative modalities (e.g., DTI metrics) to 0–1 for stable training negatively affects the quantitative value of the metrics; normalization should be avoided when synthesizing quantitative modalities.
- The in-house test suggesting no normalization dependency in SC-GAN is not presented and cannot be verified.

## Evidence
