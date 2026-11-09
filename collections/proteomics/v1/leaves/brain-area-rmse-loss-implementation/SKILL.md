---
name: brain-area-rmse-loss-implementation
description: Use when implementing or reproducing the SC-GAN generator objective (adversarial
  + L1 + feature matching + brain-area RMSE terms) and the B-rmse term must be computed
  on the brain mask only.
when_to_use_negative:
- When synthesizing quantitative modalities where the article notes normalization
  negatively affects DTI metric values and the normalization-dependent loss setup
  should be avoided
- When a brain mask is unavailable (e.g., no T1w-derived parcellation), since the
  loss is defined only over the brain area
- For 2D conditional GAN or 3D Unet baselines, which do not use the B-rmse term in
  their objectives
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SC-GAN
  role: Network whose generator objective includes the brain-area RMSE loss term;
    reference implementation defines the B-rmse weight (--B_weight, default 200)
  repo: https://github.com/Haoyulance/SC-GAN
- name: FreeSurfer 5.3.0
  role: T1w preprocessing and parcellation from which the brain mask used for the
    brain-area RMSE is obtained
- name: TensorFlow
  role: Framework in which the composite loss (adversarial, feature matching, B-rmse)
    is implemented in the reference SC-GAN code
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:a57a42c95ff930d5f9c8b3897229f12f1b1b6bacb99aa2d1592c8c1c39627ff1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - FreeSurfer 5.3.0
  - SC-GAN
  - TensorFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# brain-area-rmse-loss-implementation

## Summary

Implementation of the brain-area RMSE (B-rmse) loss term of the SC-GAN objective, a masked root-mean-square error applied only to voxels within the brain area, weighted by gamma = 200 in the SC-GAN generator loss.

## When to use

Use when implementing or reproducing the SC-GAN generator objective (adversarial + L1 + feature matching + brain-area RMSE terms) and the B-rmse term must be computed on the brain mask only.

## When NOT to use

- When synthesizing quantitative modalities where the article notes normalization negatively affects DTI metric values and the normalization-dependent loss setup should be avoided
- When a brain mask is unavailable (e.g., no T1w-derived parcellation), since the loss is defined only over the brain area
- For 2D conditional GAN or 3D Unet baselines, which do not use the B-rmse term in their objectives

## Inputs

- Generated 3D synthesis output volume (e.g., NIfTI .nii.gz, 256x256x256) from the SC-GAN generator
- Ground-truth target modality volume (e.g., PET av45, FA, or MD map as .nii.gz)
- Brain mask derived from T1w analysis (FreeSurfer 5.3.0 preprocessing/parcellation)

## Outputs

- Scalar B-rmse loss value per training iteration, added to the SC-GAN generator objective with weight gamma = 200
- sc_gan_model.py module containing the composite loss combining adversarial, L1, feature matching, and brain-area RMSE terms

## How to apply

Implement the B-rmse loss as an RMSE computed over voxels inside the brain area rather than the full 256x256x256 volume, and add it to the generator objective with weight gamma = 200 (exposed as --B_weight with default 200 in the training script). Combine it with the cGAN adversarial loss, the L1 loss (weight mu = 200) and the feature matching loss (weight lambda = 20), plus L2 regularization on generator and discriminator (nuG = 0.001, nuD = 0.001). The brain mask is obtained from T1w analysis (FreeSurfer 5.3.0 parcellation/preprocessing), so the loss penalizes synthesis error only where brain tissue exists. In TensorFlow, verify by instantiating the full SC-GAN model (batch size = 1, image size 256) from github.com/Haoyulance/SC-GAN and structurally comparing the loss function against the released reference implementation.

## Related tools

- **SC-GAN** (Network whose generator objective includes the brain-area RMSE loss term; reference implementation defines the B-rmse weight (--B_weight, default 200)) — https://github.com/Haoyulance/SC-GAN
- **FreeSurfer 5.3.0** (T1w preprocessing and parcellation from which the brain mask used for the brain-area RMSE is obtained)
- **TensorFlow** (Framework in which the composite loss (adversarial, feature matching, B-rmse) is implemented in the reference SC-GAN code)

## Examples

```
python training.py --epoches=120 --img_size=256 --data_dir=./data --modalities=flair_t1w_av45 --B_weight=200 --l1_weight=200 --g_reg=0.001 --d_reg=0.001
```

## Evaluation signals

- Instantiating the full model (batch size = 1, image size 256) runs without error and the loss module matches the reference SC-GAN loss structurally
- Training-script hyperparameters show --B_weight default = 200 alongside --l1_weight = 200 and --g_reg/--d_reg = 0.001
- Ablation check: removing the B-rmse term reproduces the 'SC-GAN (3) without brain area RMSE loss' configuration and changes evaluation metrics (normalized RMS error, peak SNR, structural similarity)
- The RMSE is masked: error is computed only over voxels within the T1w-derived brain mask, not the full image cube

## Limitations

- Computational cost is the main limitation of SC-GAN: it requires lengthy training and heavy GPU memory
- Normalizing target DTI images to 0-1 for stable training negatively affects the quantitative value of DTI metrics, so the loss/training setup should avoid normalization when synthesizing quantitative modalities
- The in-house test suggesting no normalization dependency in SC-GAN is not presented in the paper and cannot be verified

## Evidence
