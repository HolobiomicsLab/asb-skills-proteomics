---
name: gan-training-hyperparameter-tuning
description: Use when synthesizing a target modality (PET, FA, or MD maps) from input
  modalities such as T1w and FLAIR, and you need to train a 3D conditional GAN with
  tuned regularization, loss weights, and learning-rate settings.
when_to_use_negative:
- Synthesizing quantitative modalities such as DTI metrics when the quantitative value
  of the metrics matters — the paper notes the 0-1 normalization step negatively affects
  quantitative DTI values
- Tasks with limited GPU memory or compute budget — SC-GAN requires lengthy training
  and heavy GPU memory on 256^3 volumes
- 2D slice-wise synthesis where 3D spatial consistency is not needed — a 2D cGAN with
  different hyperparameters (mu=100, batch size 4) may suffice
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_3474
tools:
- name: SC-GAN
  role: Primary 3D self-attention conditional GAN trained with spectral normalization
    and feature matching for multimodal synthesis
  repo: https://github.com/Haoyulance/SC-GAN
- name: 2D conditional GAN
  role: Baseline trained with its own tuned settings (mu=100, batch size 4, lr=0.0002)
    for comparison
  repo: https://github.com/Haoyulance/SC-GAN
- name: 3D conditional GAN
  role: Baseline trained with tuned settings (mu=200, batch size 1, lr=0.002) to isolate
    the effect of self-attention and spectral normalization
  repo: https://github.com/Haoyulance/SC-GAN
- name: TensorFlow
  role: Deep-learning framework in which training.py and testing.py are implemented
    (TensorFlow 1.12.2)
  repo: https://github.com/Haoyulance/SC-GAN
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:d2bac327438e7a16b656ace33d54633e9a24bb5a154639f695e2128d63ce4e08
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 2D conditional GAN
  - 3D conditional GAN
  - FreeSurfer
  - SC-GAN
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gan-training-hyperparameter-tuning

## Summary

Train the SC-GAN 3D self-attention conditional GAN (and 2D/3D cGAN baselines) on multimodal neuroimaging synthesis tasks using hyperparameters selected by 5-fold cross-validation, with spectral normalization and feature matching to stabilize 3D GAN training.

## When to use

Use when synthesizing a target modality (PET, FA, or MD maps) from input modalities such as T1w and FLAIR, and you need to train a 3D conditional GAN with tuned regularization, loss weights, and learning-rate settings.

## When NOT to use

- Synthesizing quantitative modalities such as DTI metrics when the quantitative value of the metrics matters — the paper notes the 0-1 normalization step negatively affects quantitative DTI values
- Tasks with limited GPU memory or compute budget — SC-GAN requires lengthy training and heavy GPU memory on 256^3 volumes
- 2D slice-wise synthesis where 3D spatial consistency is not needed — a 2D cGAN with different hyperparameters (mu=100, batch size 4) may suffice

## Inputs

- Per-subject NIfTI volumes (.nii.gz) of input modalities (e.g., T1w, FLAIR) and target modality (PET/av45, FA, or MD), organized as data/train, data/val, data/test folders
- ADNI-3 preprocessed volumes up-sampled to 256 x 256 x 256 with Z-score + min-max normalization applied
- requirement.txt environment specification (python 3.6.9, TensorFlow)

## Outputs

- Trained SC-GAN generator checkpoint (tensorboard logdir) for a chosen modality set
- Synthesized target volumes from testing.py on held-out subjects
- Trained baseline models (2D cGAN, 3D cGAN, Att cGAN) for comparison

## How to apply

First normalize all volumes (Z-score normalization followed by min-max rescaling of voxel intensities to 0-1) before training. Train SC-GAN in TensorFlow on the GPU cluster with the optimal hyperparameters found via 5-fold cross-validation: mu=200, gamma=200, lambda=20, learning rate 0.001 with cosine decay, batch size 1 (README defaults: lr=0.001, g_reg=0.001, d_reg=0.001, l1_weight=200, B_weight=200, epoches=120, img_size=256). Train the baselines under their own tuned settings: 2D cGAN (mu=100, batch size 4, lr=0.0002), 3D cGAN (mu=200, batch size 1, lr=0.002), and Att cGAN (mu=200, batch size 1, lr=0.001). Stabilization of 3D GAN training relies on spectral normalization and feature matching; ablations show adversarial learning and the self-attention module are the major contributors to performance, followed by the spectral normalization module. Organize data into train/val/test subject folders with per-subject modality files (e.g., flair_t1w_..._target) as required by the repository's training.py.

## Related tools

- **SC-GAN** (Primary 3D self-attention conditional GAN trained with spectral normalization and feature matching for multimodal synthesis) — https://github.com/Haoyulance/SC-GAN
- **2D conditional GAN** (Baseline trained with its own tuned settings (mu=100, batch size 4, lr=0.0002) for comparison) — https://github.com/Haoyulance/SC-GAN
- **3D conditional GAN** (Baseline trained with tuned settings (mu=200, batch size 1, lr=0.002) to isolate the effect of self-attention and spectral normalization) — https://github.com/Haoyulance/SC-GAN
- **TensorFlow** (Deep-learning framework in which training.py and testing.py are implemented (TensorFlow 1.12.2)) — https://github.com/Haoyulance/SC-GAN

## Examples

```
python training.py --training_size=398 --gpu=0 --epoches=120 --img_size=256 --data_dir=data --modalities=flair_t1w_target --logdir=logs/scgan_fa
```

## Evaluation signals

- Synthesized volumes on the 99-subject held-out test split should show lower NRMSE, higher peak SNR, and higher structural similarity than all baselines (SC-GAN NRMSE was 18%, 24%, 29% lower than 2D cGAN for FA, PET, MD respectively)
- Training should converge stably without mode collapse — spectral normalization plus feature matching are expected to ensure optimization convergence
- Ablation sanity check: removing the self-attention module or adversarial learning should degrade performance, confirming those components were active in training
- README contract check: data directory contains train/val/test subject folders, each with modality_1.nii.gz ... target.nii.gz, and --modalities encodes inputs then target (e.g., flair_t1w_av45)
- Hyperparameters match the cross-validation-selected values (mu=200, gamma=200, lambda=20, lr=0.001 with cosine decay, batch size 1)

## Limitations

- Computational cost is the main limitation of SC-GAN; it requires lengthy training and heavy computational resources, such as GPU memory
- Normalizing target DTI images to 0-1 for stable training negatively affects the quantitative value of DTI metrics
- Hyperparameters were tuned for ADNI-3 tasks at 256^3 resolution; transfer to other resolutions or datasets is unverified

## Evidence
