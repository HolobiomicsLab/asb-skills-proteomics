---
name: gan-hyperparameter-tuning-cross-validation
description: Use when training conditional GANs (SC-GAN, 2D/3D cGAN, Att cGAN) on
  256x256x256 medical images and you need to select hyperparameters (mu, lambda, lr,
  batch size) fairly via cross-validation before benchmarking.
when_to_use_negative:
- The task is not image-to-image synthesis from MRI but classification/regression
  or data already reduced to feature tables — the tuning targets voxel-level GAN training.
- No cross-validation budget or comparison networks are needed and a single off-the-shelf
  model configuration suffices.
- Synthesizing quantitative modalities where min-max rescaling to 0-1 would corrupt
  the quantitative metric values (the authors note normalization negatively affects
  DTI metrics).
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_3384
- http://edamontology.org/topic_3474
tools:
- name: SC-GAN
  role: Primary network being tuned and benchmarked; 3D conditional GAN with spectral
    normalization, feature matching, self-attention module and brain-area RMSE loss
  repo: https://github.com/Haoyulance/SC-GAN
- name: 2D conditional GAN
  role: Baseline network tuned with mu=100, nu_G=nu_D=0.01, batch size=4, lr=0.0002
    for comparison
- name: 3D conditional GAN
  role: Baseline network tuned with mu=200, nu_G=nu_D=0.001, batch size=1, lr=0.002
    for comparison
- name: Attention cGAN (Att cGAN)
  role: Baseline network tuned with mu=200, nu_G=nu_D=0.001, batch size=1, lr=0.001
    for comparison
- name: TensorFlow
  role: Deep-learning framework used to implement and train all four networks
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:bc3838c1a6eac5a94974aafc64030b7577a69e5d0d3e9125f0365b9a0dbb37e0
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
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_3384
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gan-hyperparameter-tuning-cross-validation

## Summary

Tunes and compares GAN architectures for 3D neuroimaging synthesis by sweeping regularization weights, learning rates and batch sizes under 5-fold cross-validation, then benchmarking on held-out subjects with NRMSE, peak SNR and SSIM. Grounded in the SC-GAN study on ADNI-3 PET/FA/MD synthesis from T1w and FLAIR MRI.

## When to use

Use when training conditional GANs (SC-GAN, 2D/3D cGAN, Att cGAN) on 256x256x256 medical images and you need to select hyperparameters (mu, lambda, lr, batch size) fairly via cross-validation before benchmarking.

## When NOT to use

- The task is not image-to-image synthesis from MRI but classification/regression or data already reduced to feature tables — the tuning targets voxel-level GAN training.
- No cross-validation budget or comparison networks are needed and a single off-the-shelf model configuration suffices.
- Synthesizing quantitative modalities where min-max rescaling to 0-1 would corrupt the quantitative metric values (the authors note normalization negatively affects DTI metrics).

## Inputs

- Paired multimodal neuroimaging volumes (e.g., T1w and FLAIR MRI .nii.gz upsampled to 256x256x256) with target modality (amyloid PET, FA, or MD maps), organized per-subject under data/train, data/val, and data/test directories
- ADNI-3 cohort split (265 subjects: 207 training, 58 testing)

## Outputs

- Tuned per-network hyperparameter settings (mu, gamma, lambda, nu_G, nu_D, learning rate, batch size) selected by 5-fold cross-validation
- Synthesized 3D PET volumes for the 58 held-out testing subjects per network
- Comparison table of NRMSE, peak SNR and SSIM per network (pet_synthesis_comparison_table.csv)

## How to apply

Preprocess all images with Z-score normalization followed by min-max rescaling of voxel intensities to 0-1, then split the cohort (e.g., 265 ADNI-3 subjects: 207 train / 58 test). During the tuning phase, run 5-fold cross-validation for every network being compared, sweeping the adversarial loss weight mu (e.g., 100-200), the brain-area RMSE weight gamma (200), the feature-matching weight lambda (20), generator/discriminator regularizers nu_G and nu_D (0.001-0.01), initial learning rate (0.0002-0.002) and batch size (1-4). The SC-GAN README exposes these via --lr, --g_reg, --d_reg, --l1_weight and --B_weight flags with defaults lr=0.001, g_reg=d_reg=0.001, l1_weight=200, B_weight=200 and 120 epochs. Train the winning configuration per network in TensorFlow (img_size=256), run inference on the held-out testing subjects, and compute normalized RMSE, peak SNR and SSIM against ground truth. Correct application is judged by cross-validated selection giving the lowest NRMSE / highest PSNR / highest SSIM among compared networks, with SC-GAN expected to yield ~24% lower prediction error than 2D cGAN on PET synthesis.

## Related tools

- **SC-GAN** (Primary network being tuned and benchmarked; 3D conditional GAN with spectral normalization, feature matching, self-attention module and brain-area RMSE loss) — https://github.com/Haoyulance/SC-GAN
- **2D conditional GAN** (Baseline network tuned with mu=100, nu_G=nu_D=0.01, batch size=4, lr=0.0002 for comparison)
- **3D conditional GAN** (Baseline network tuned with mu=200, nu_G=nu_D=0.001, batch size=1, lr=0.002 for comparison)
- **Attention cGAN (Att cGAN)** (Baseline network tuned with mu=200, nu_G=nu_D=0.001, batch size=1, lr=0.001 for comparison)
- **TensorFlow** (Deep-learning framework used to implement and train all four networks)

## Examples

```
python training.py --training_size=207 --gpu=0 --epoches=120 --img_size=256 --data_dir=data --modalities=flair_t1w_av45 --lr=0.001 --g_reg=0.001 --d_reg=0.001 --l1_weight=200 --B_weight=200
```

## Evaluation signals

- SC-GAN achieves the lowest NRMSE and the highest peak SNR and SSIM of all compared networks on the 58 testing subjects.
- Prediction error of SC-GAN is approximately 24% lower than 2D conditional GAN for the PET synthesis task.
- Every network in the comparison underwent 5-fold cross-validation during hyperparameter tuning, ensuring no selection bias toward SC-GAN.
- Voxel intensities are Z-score normalized then min-max rescaled to 0-1 before training, and inference is run on all 58 held-out subjects with 3D output evaluated directly for the 2D network.

## Limitations

- Computational cost is the main limitation of SC-GAN: it requires lengthy training and heavy computational resources, such as GPU memory.
- Min-max rescaling target images to 0-1 for stable training negatively affects the quantitative value of DTI metrics, so normalization should be avoided when synthesizing quantitative modalities.
- Ablation results indicate performance depends on adversarial learning and the self-attention module, followed by spectral normalization — removing these modules changes the tuning landscape.

## Evidence
