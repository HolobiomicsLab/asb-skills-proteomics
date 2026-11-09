---
name: diffusion-metric-map-synthesis
description: Use when the task is predicting a 3D quantitative diffusion metric map
  (MD, FA) from structural MRI modalities such as T1w and FLAIR in high-resolution
  (256 x 256 x 256) volumes and comparing GAN-based synthesis quality.
when_to_use_negative:
- Synthesizing quantitative modalities where normalization to 0-1 destroys quantitative
  value — the authors warn the normalization step 'should be avoided' for such targets
- Multidimensional diffusion MRI superresolution without the additional acquisition
  protocol data required for further experiments and analyses
- The target is a small or non-volumetric dataset that cannot support 3D training;
  SC-GAN's main limitation is computational cost (lengthy training and heavy GPU memory
  requirements)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: SC-GAN
  role: 'Primary synthesis network: 3D conditional GAN with self-attention module,
    spectral normalization, feature matching, and brain-area RMSE loss, trained on
    T1w+FLAIR to predict MD maps'
  repo: https://github.com/Haoyulance/SC-GAN
- name: 2D conditional GAN
  role: Baseline synthesis network against which SC-GAN's MD prediction error (29%
    lower NRMSE) is compared, evaluated in 3D for fairness
  repo: https://github.com/Haoyulance/SC-GAN
- name: 3D conditional GAN
  role: Baseline 3D GAN without the self-attention module, outperformed by SC-GAN
    on all MD evaluation metrics
  repo: https://github.com/Haoyulance/SC-GAN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:7a0c6fc15d71dfb9faf65b8cd5d0e558e6560f240df7c46738d1bc33231ea153
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 2D conditional GAN
  - 3D conditional GAN
  - SC-GAN
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# diffusion-metric-map-synthesis

## Summary

Synthesize 3D mean diffusivity (MD) maps from T1w and FLAIR MRI using the SC-GAN architecture (3D self-attention conditional GAN with spectral normalization), benchmarked against 2D cGAN, 3D cGAN, and Att cGAN baselines on ADNI-3 data with NRMSE, peak SNR, and structural similarity metrics.

## When to use

Use when the task is predicting a 3D quantitative diffusion metric map (MD, FA) from structural MRI modalities such as T1w and FLAIR in high-resolution (256 x 256 x 256) volumes and comparing GAN-based synthesis quality.

## When NOT to use

- Synthesizing quantitative modalities where normalization to 0-1 destroys quantitative value — the authors warn the normalization step 'should be avoided' for such targets
- Multidimensional diffusion MRI superresolution without the additional acquisition protocol data required for further experiments and analyses
- The target is a small or non-volumetric dataset that cannot support 3D training; SC-GAN's main limitation is computational cost (lengthy training and heavy GPU memory requirements)

## Inputs

- Preprocessed ADNI-3 T1w MRI volumes (.nii.gz), 256 x 256 x 256, Z-score normalized and min-max rescaled to 0-1
- Preprocessed ADNI-3 FLAIR volumes (.nii.gz), N4ITK field-corrected and co-registered to T1w
- Ground-truth mean diffusivity (MD) maps derived from diffusion MRI (DTI fit via Quantitative Imaging Toolkit after FSL TOPUP/EDDY correction)
- Train/val/test subject splits (398 train, 99 test for the MD task)

## Outputs

- Synthesized 3D MD maps (.nii.gz) for each of the 99 test subjects, produced by testing.py
- Comparison table of NRMSE, peak SNR, and structural similarity for SC-GAN, 2D cGAN, 3D cGAN, and Att cGAN (e.g., md_synthesis_comparison.csv)

## How to apply

Load preprocessed ADNI-3 data (497 subjects: 398 training, 99 testing) with T1w and FLAIR as input modalities and MD maps as targets, up-sampled to 256 x 256 x 256. Normalize all volumes with Z-score normalization followed by min-max rescaling of voxel intensities to the 0-1 range before training. Train SC-GAN with the tuned hyperparameters (mu = 200, gamma = 200, lambda = 20, learning rate 0.001 with cosine decay, batch size 1) in TensorFlow, and train the 2D cGAN (mu = 100, batch 4, lr 0.0002), 3D cGAN (mu = 200, batch 1, lr 0.002), and Att cGAN (mu = 200, batch 1, lr 0.001) baselines tuned by 5-fold cross-validation. Run testing.py on the 99-subject test split, evaluating the 3D output of the 2D network directly in 3D for a fair comparison. Compute normalized RMSE, peak SNR, and structural similarity between synthesized and ground-truth MD maps; SC-GAN's NRMSE should be about 29% lower than 2D cGAN.

## Related tools

- **SC-GAN** (Primary synthesis network: 3D conditional GAN with self-attention module, spectral normalization, feature matching, and brain-area RMSE loss, trained on T1w+FLAIR to predict MD maps) — https://github.com/Haoyulance/SC-GAN
- **2D conditional GAN** (Baseline synthesis network against which SC-GAN's MD prediction error (29% lower NRMSE) is compared, evaluated in 3D for fairness) — https://github.com/Haoyulance/SC-GAN
- **3D conditional GAN** (Baseline 3D GAN without the self-attention module, outperformed by SC-GAN on all MD evaluation metrics) — https://github.com/Haoyulance/SC-GAN

## Examples

```
python testing.py --test_size=99 --gpu=0 --img_size=256 --data_dir=./data --modalities=flair_t1w_md --logdir=./logs --output=./results
```

## Evaluation signals

- SC-GAN's NRMSE on the 99-subject MD test split is approximately 29% lower than the 2D cGAN baseline
- SC-GAN beats 3D cGAN and Att cGAN on all three metrics (NRMSE, peak SNR, structural similarity) for the MD task
- All synthesized volumes have the expected 256 x 256 x 256 dimensions and match the test-split subject count (99)
- All voxel intensities were Z-score normalized then min-max rescaled to 0-1 before training, consistent with the reported preprocessing
- The comparison table contains NRMSE, peak SNR, and structural similarity rows for all four networks (SC-GAN, 2D cGAN, 3D cGAN, Att cGAN)

## Limitations

- Computational cost is the main limitation of SC-GAN; it requires lengthy training and heavy computational resources, such as GPU memory
- Normalizing target quantitative DTI images (like MD) to 0-1 for stable training negatively affects the quantitative value of DTI metrics
- The in-house test suggesting no normalization dependency in SC-GAN is not presented, so it cannot be verified

## Evidence
