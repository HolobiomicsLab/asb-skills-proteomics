---
name: tensorflow-gpu-distributed-training
description: Use when training a 3D conditional GAN (e.g., SC-GAN) in TensorFlow on
  256x256x256 multimodal MRI volumes to synthesize PET, FA, or MD maps, or when GPU
  memory constraints force batch size 1 and stabilized training (spectral normalization,
  feature matching) is needed.
when_to_use_negative:
- Synthesizing quantitative modalities (e.g., DTI metrics) where the 0-1 normalization
  step would negatively affect the quantitative value of the resulting metrics
- Multidimensional diffusion MRI superresolution tasks lacking the additional acquisition
  protocol data required for further experiments and analyses
- 'When GPU memory is insufficient: SC-GAN on 256^3 volumes requires heavy GPU resources,
  so without adequate hardware training is impractical'
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3384
tools:
- name: SC-GAN
  role: Primary 3D self-attention conditional GAN trained in TensorFlow for PET synthesis;
    uses spectral normalization, feature matching, and brain-area RMSE loss for stable
    training
  repo: https://github.com/Haoyulance/SC-GAN
- name: 2D conditional GAN
  role: Baseline network trained with tuned hyperparameters (mu=100, nu_G=nu_D=0.01,
    lr=0.0002, batch 4) for NRMSE/PSNR/SSIM comparison; 3D output evaluated directly
- name: 3D conditional GAN
  role: Baseline network trained with mu=200, lr=0.002, batch 1 to benchmark SC-GAN's
    stabilized training
- name: Attention cGAN (Att cGAN)
  role: Baseline network trained with mu=200, lr=0.001, batch 1 for comparison across
    all three image-quality metrics
- name: TensorFlow
  role: Deep-learning framework used to implement and run training.py/testing.py for
    SC-GAN and baselines; requirements pinned in requirement.txt (Python 3.6.9)
  repo: https://github.com/Haoyulance/SC-GAN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:0562871f0b9f59c155b245b7d6966d75c91d6aa7e192abcb6e6f2c53afb9e3c3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 2D conditional GAN
  - 3D conditional GAN
  - Attention cGAN (Att cGAN)
  - SC-GAN
  - TensorFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3384
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 20
---

# tensorflow-gpu-distributed-training

## Summary

GPU-based TensorFlow training of the SC-GAN 3D self-attention conditional GAN (with spectral normalization and feature matching) for end-to-end synthesis of 256x256x256 neuroimaging volumes such as PET from multimodal MRI, plus tuned baselines (2D cGAN, 3D cGAN, Att cGAN) for benchmarking.

## When to use

Use when training a 3D conditional GAN (e.g., SC-GAN) in TensorFlow on 256x256x256 multimodal MRI volumes to synthesize PET, FA, or MD maps, or when GPU memory constraints force batch size 1 and stabilized training (spectral normalization, feature matching) is needed.

## When NOT to use

- Synthesizing quantitative modalities (e.g., DTI metrics) where the 0-1 normalization step would negatively affect the quantitative value of the resulting metrics
- Multidimensional diffusion MRI superresolution tasks lacking the additional acquisition protocol data required for further experiments and analyses
- When GPU memory is insufficient: SC-GAN on 256^3 volumes requires heavy GPU resources, so without adequate hardware training is impractical

## Inputs

- ADNI-3 multimodal MRI inputs (T1w, FLAIR) and target PET images as .nii.gz volumes upsampled to 256x256x256
- Z-score normalized and min-max rescaled (0-1) voxel intensities, organized as data/train, data/val, data/test per-subject folders (207 training / 58 testing subjects)

## Outputs

- Trained SC-GAN model checkpoints with TensorBoard logs (--logdir)
- Synthesized 256x256x256 PET volumes for the 58 test subjects (--output directory of testing.py)
- Per-network NRMSE, peak SNR, and SSIM comparison table (pet_synthesis_comparison_table.csv)

## How to apply

Organize preprocessed data under data/train, data/val, data/test with per-subject modality files (e.g., T1w and FLAIR .nii.gz plus a target.nii.gz), after Z-score normalization followed by min-max rescaling of voxel intensities to 0-1. Launch the training script with the SC-GAN hyperparameters (l1_weight/mu=200, B_weight/gamma=200, generator and discriminator regularizers nu_G=nu_D=0.001, lr=0.001, epochs=120, img_size=256, batch size 1) on an assigned GPU, and monitor convergence via TensorBoard logs. Train the three baselines with their own tuned settings for comparison: 2D cGAN (mu=100, lr=0.0002, batch 4), 3D cGAN (mu=200, lr=0.002, batch 1), and Att cGAN (mu=200, lr=0.001, batch 1). Run testing.py on the 58 held-out ADNI-3 subjects to generate synthesized volumes, then evaluate NRMSE, peak SNR, and SSIM against ground truth. Correct application is confirmed when SC-GAN achieves the lowest NRMSE (~24% lower error than 2D cGAN) and ranks best on all three metrics.

## Related tools

- **SC-GAN** (Primary 3D self-attention conditional GAN trained in TensorFlow for PET synthesis; uses spectral normalization, feature matching, and brain-area RMSE loss for stable training) — https://github.com/Haoyulance/SC-GAN
- **2D conditional GAN** (Baseline network trained with tuned hyperparameters (mu=100, nu_G=nu_D=0.01, lr=0.0002, batch 4) for NRMSE/PSNR/SSIM comparison; 3D output evaluated directly)
- **3D conditional GAN** (Baseline network trained with mu=200, lr=0.002, batch 1 to benchmark SC-GAN's stabilized training)
- **Attention cGAN (Att cGAN)** (Baseline network trained with mu=200, lr=0.001, batch 1 for comparison across all three image-quality metrics)
- **TensorFlow** (Deep-learning framework used to implement and run training.py/testing.py for SC-GAN and baselines; requirements pinned in requirement.txt (Python 3.6.9)) — https://github.com/Haoyulance/SC-GAN

## Examples

```
python training.py --trainig_size=207 --gpu=0 --epoches=120 --img_size=256 --data_dir=data --modalities=flair_t1w_av45 --logdir=logs
```

## Evaluation signals

- SC-GAN achieves the lowest NRMSE among the four networks, with prediction error approximately 24% lower than the 2D conditional GAN for the PET task
- SC-GAN ranks best on all three evaluation metrics (NRMSE, peak SNR, structural similarity) against 2D cGAN, 3D cGAN, and Att cGAN
- TensorBoard logs (--logdir) show stable convergence of generator/discriminator losses over the 120 training epochs with lr=0.001 and cosine decay
- Output volumes have the expected 256x256x256 dimensions and voxel intensities consistent with the min-max rescaled 0-1 target space
- Comparative metrics assemble cleanly into pet_synthesis_comparison_table.csv with one row per network and one column per metric

## Limitations

- Computational cost is the main limitation of SC-GAN; it requires lengthy training and heavy computational resources, such as GPU memory
- Min-max rescaling of target intensities to 0-1 for stable training negatively affects the quantitative value of synthesized metrics for quantitative modalities
- The in-house test suggesting no normalization dependency in SC-GAN is not presented in the paper, so it cannot be verified
- Training config only supports cubic images (img_size applies identically to all three dimensions, default 256)

## Evidence
