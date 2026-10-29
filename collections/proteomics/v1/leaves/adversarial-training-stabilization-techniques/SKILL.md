---
name: adversarial-training-stabilization-techniques
description: Use when training a 3D conditional GAN for cross-modality image synthesis
  (e.g. T1w+FLAIR MRI to amyloid PET) shows unstable or non-converging adversarial
  optimization, and you need spectral normalization/feature-matching stabilization.
when_to_use_negative:
- Input task has no adversarial component (e.g. a plain 3D Unet regression) — spectral
  normalization and feature matching specifically target GAN training stabilization
  and do not apply.
- GPU memory or training time is severely constrained — the paper notes computational
  cost is the main limitation of SC-GAN, requiring lengthy training and heavy GPU
  memory.
- You only need image preprocessing/registration (N4ITK correction, ANTs coregistration,
  FSL TOPUP/EDDY) — this skill concerns GAN training stabilization, not upstream data
  preparation.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3306
tools:
- name: SC-GAN
  role: 3D self-attention conditional GAN whose training is stabilized with spectral
    normalization and feature matching; the ablation variants are built by selectively
    removing its components.
  repo: https://github.com/Haoyulance/SC-GAN
- name: TensorFlow
  role: Framework in which the SC-GAN training/testing scripts (training.py, testing.py)
    implementing the stabilized adversarial loss are run.
  repo: https://github.com/Haoyulance/SC-GAN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:04da2ece1298b9944c02a4a6b0f7ba68e470a2caf9564108fdd9a0e12350424d
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
  - http://edamontology.org/topic_3306
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# adversarial-training-stabilization-techniques

## Summary

Stabilizes training of a 3D conditional GAN (SC-GAN) for multimodal neuroimaging synthesis by combining spectral normalization, a feature matching loss, and a modified loss function (L1 + brain-area RMSE), so the generator-discriminator optimization converges for 256x256x256 MRI/PET inputs.

## When to use

Use when training a 3D conditional GAN for cross-modality image synthesis (e.g. T1w+FLAIR MRI to amyloid PET) shows unstable or non-converging adversarial optimization, and you need spectral normalization/feature-matching stabilization.

## When NOT to use

- Input task has no adversarial component (e.g. a plain 3D Unet regression) — spectral normalization and feature matching specifically target GAN training stabilization and do not apply.
- GPU memory or training time is severely constrained — the paper notes computational cost is the main limitation of SC-GAN, requiring lengthy training and heavy GPU memory.
- You only need image preprocessing/registration (N4ITK correction, ANTs coregistration, FSL TOPUP/EDDY) — this skill concerns GAN training stabilization, not upstream data preparation.

## Inputs

- Per-subject training directories containing input modality NIfTI files (e.g. modality_1.nii.gz, modality_2.nii.gz) plus target.nii.gz
- ADNI-3 PET task: T1w + FLAIR MRI inputs (flair_t1w_av45 configuration), 207 training subjects, 256x256x256 volumes
- requirements.txt from https://github.com/Haoyulance/SC-GAN/blob/master/requirements.txt (python 3.6.9 environment)

## Outputs

- Trained SC-GAN generator checkpoint (tensorboard logs under --logdir) with stabilized adversarial optimization
- Synthesized target-modality NIfTI volumes (e.g. amyloid PET) for the test set via testing.py
- Ablation_results.csv with NRMSE, peak SNR, and structural similarity for full SC-GAN and the five ablation conditions

## How to apply

Build SC-GAN from the TensorFlow source at github.com/Haoyulance/SC-GAN, applying spectral normalization and feature matching to stabilize the 3D conditional GAN's adversarial training. Train on Z-score normalized, min-max rescaled images (256x256x256) with the paper's optimal hyperparameters: learning rate 0.001 with cosine decay, batch size 1, l1_weight 200, B-rmse (B_weight) 200, generator and discriminator regularizers 0.001. Use the training script with --modalities=flair_t1w_av45 (last modality is the target) and ~120 epochs, feeding per-subject directories of modality_*.nii.gz plus target.nii.gz. To verify which stabilization component contributes most, run the five-condition ablation (remove self-attention, adversarial learning, B-rmse, spectral normalization, or feature matching) and rank component contributions by NRMSE, peak SNR, and structural similarity degradation on the held-out test set; the paper found adversarial learning and the self-attention module are the major contributors, followed by the spectral normalization module.

## Related tools

- **SC-GAN** (3D self-attention conditional GAN whose training is stabilized with spectral normalization and feature matching; the ablation variants are built by selectively removing its components.) — https://github.com/Haoyulance/SC-GAN
- **TensorFlow** (Framework in which the SC-GAN training/testing scripts (training.py, testing.py) implementing the stabilized adversarial loss are run.) — https://github.com/Haoyulance/SC-GAN

## Examples

```
python training.py --training_size=207 --gpu=0 --epoches=120 --img_size=256 --data_dir=data/train --modalities=flair_t1w_av45 --logdir=logs --lr=0.001 --l1_weight=200 --B_weight=200
```

## Evaluation signals

- Ablation ranking check: removing adversarial learning and the self-attention module produces the largest degradation in NRMSE/peak SNR/structural similarity, followed by spectral normalization — matching the paper's reported finding.
- Optimization convergence: training with spectral normalization and feature matching runs the full ~120 epochs without discriminator/generator divergence, compared to unstable training when spectral normalization is ablated.
- Output volumes are 256x256x256 synthesized PET images evaluated against ground truth with normalized RMS error, peak SNR, and structural similarity.
- Hyperparameter invariants: l1_weight=200, B_weight=200, lr=0.001, generator/discriminator regularizers 0.001, img_size=256.
- Test-set inference via testing.py on the held-out subjects (e.g. the 58-subject ADNI-3 test set) produces synthesis results in the --output directory.

## Limitations

- Computational cost is the main limitation of SC-GAN; it requires lengthy training and heavy computational resources, such as GPU memory.
- The ablation finding is task-specific: major contributors were verified on ADNI-3 PET synthesis; contributions of B-rmse vs spectral normalization ordering differ between the task card and the abstract, so re-rank per task.
- For synthesizing quantitative modalities, the paper warns that min-max normalization (used for stable training) negatively affects the quantitative value of the metrics.

## Evidence

- [readme] --lr|learning rate|0.001|--g_reg|generator regularizer|0.001|--reg|if using regularization|True|--l1_weight|l1 weight|200|--B_weight|B-rmse weight|200: [evidence span withheld — anchor status: not_found]
