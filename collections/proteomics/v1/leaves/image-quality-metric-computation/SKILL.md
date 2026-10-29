---
name: image-quality-metric-computation
description: Use when 3D medical image synthesis outputs (e.g., 256x256x256 PET volumes
  from T1w/FLAIR MRI) must be scored against ground-truth volumes to compare competing
  networks (SC-GAN vs 2D/3D cGAN, Att cGAN).
when_to_use_negative:
- 'When synthesizing quantitative modalities such as DTI metrics: normalizing target
  DTI images to 0-1 for stable training negatively affects the quantitative value
  of DTI metrics, so the normalization underlying comparable metric computation should
  be avoided per the authors'
- When ground-truth target volumes are unavailable for the test subjects, since NRMSE/peak
  SNR/SSIM all require reference images
edam_operation: http://edamontology.org/operation_0286
tools:
- name: SC-GAN
  role: Best-performing synthesis network whose PET outputs are scored with NRMSE,
    peak SNR, and SSIM; prediction error was 24% lower than 2D conditional GAN for
    the PET task
  repo: https://github.com/Haoyulance/SC-GAN
- name: 2D conditional GAN
  role: Baseline network whose synthesized PET volumes are evaluated with the same
    three image-quality metrics for comparison
  repo: https://github.com/Haoyulance/SC-GAN
- name: 3D conditional GAN
  role: Baseline network whose synthesized PET volumes are evaluated with the same
    three image-quality metrics for comparison
  repo: https://github.com/Haoyulance/SC-GAN
- name: Attention cGAN (Att cGAN)
  role: Baseline network whose synthesized PET volumes are evaluated with the same
    three image-quality metrics for comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:44e492cd81786272131f798c630ba10f39f9b4367539438816afee16f0bca78f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 2D conditional GAN
  - 3D conditional GAN
  - Attention cGAN (Att cGAN)
  - SC-GAN
  edam_operation: http://edamontology.org/operation_0286
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# image-quality-metric-computation

## Summary

Quantitatively assess GAN-synthesized neuroimages (e.g., ADNI-3 PET predicted from multimodal MRI) against ground truth using normalized RMS error (NRMSE), peak SNR, and structural similarity (SSIM), assembling per-network comparison tables. This is the standard evaluation protocol used in the SC-GAN study to rank synthesis networks.

## When to use

Use when 3D medical image synthesis outputs (e.g., 256x256x256 PET volumes from T1w/FLAIR MRI) must be scored against ground-truth volumes to compare competing networks (SC-GAN vs 2D/3D cGAN, Att cGAN).

## When NOT to use

- When synthesizing quantitative modalities such as DTI metrics: normalizing target DTI images to 0-1 for stable training negatively affects the quantitative value of DTI metrics, so the normalization underlying comparable metric computation should be avoided per the authors
- When ground-truth target volumes are unavailable for the test subjects, since NRMSE/peak SNR/SSIM all require reference images

## Inputs

- Synthesized PET volumes (NIfTI, .nii.gz, 256x256x256) from each network's testing.py output directory
- Ground-truth PET volumes for the 58 held-out ADNI-3 testing subjects
- Preprocessed input images (Z-score normalized, min-max rescaled to 0-1)

## Outputs

- Per-network NRMSE, peak SNR, and SSIM values
- Comparison table (e.g., pet_synthesis_comparison_table.csv) ranking networks across all three metrics

## How to apply

After running inference with each trained network on held-out subjects (e.g., 58 ADNI-3 testing subjects), compute normalized RMSE, peak SNR, and structural similarity between each synthesized PET volume and its ground truth, evaluating the 3D output directly (for 2D networks, evaluate the stacked 3D volume for comparability). All images should already be preprocessed with Z-score normalization followed by min-max rescaling of voxel intensities to 0-1 so metrics are computed on comparable intensity scales. Assemble per-network metric values into a comparison table (e.g., pet_synthesis_comparison_table.csv) and rank networks across all three metrics. Correct application is confirmed when the best model (SC-GAN in the source study) shows the lowest NRMSE (approximately 24% lower error than 2D cGAN) and the highest peak SNR and SSIM among all tested networks.

## Related tools

- **SC-GAN** (Best-performing synthesis network whose PET outputs are scored with NRMSE, peak SNR, and SSIM; prediction error was 24% lower than 2D conditional GAN for the PET task) — https://github.com/Haoyulance/SC-GAN
- **2D conditional GAN** (Baseline network whose synthesized PET volumes are evaluated with the same three image-quality metrics for comparison) — https://github.com/Haoyulance/SC-GAN
- **3D conditional GAN** (Baseline network whose synthesized PET volumes are evaluated with the same three image-quality metrics for comparison) — https://github.com/Haoyulance/SC-GAN
- **Attention cGAN (Att cGAN)** (Baseline network whose synthesized PET volumes are evaluated with the same three image-quality metrics for comparison)

## Examples

```
python testing.py --test_size= --gpu= --img_size= --data_dir= --modalities= --logdir= --output=
```

## Evaluation signals

- SC-GAN should show the lowest NRMSE of the four networks, approximately 24% lower error than 2D conditional GAN on the PET task
- The same network should rank best across all three metrics (lowest NRMSE, highest peak SNR, highest SSIM); inconsistent cross-metric ranking suggests a metric computation or preprocessing mismatch
- All volumes must be evaluated on identically preprocessed intensity scales (Z-score normalization then min-max rescaling to 0-1) before metric computation
- The 2D network's output must be evaluated as 3D volumes for direct comparability with 3D networks
- Metrics cover all 58 held-out ADNI-3 testing subjects with no subjects dropped

## Limitations

- Computational cost is the main limitation of SC-GAN; it requires lengthy training and heavy computational resources, such as GPU memory, which constrains how extensively synthesis-then-evaluation can be iterated
- The in-house test suggesting no normalization dependency in SC-GAN is not presented, so it cannot be verified; metric values may depend on the normalization scheme applied
- For quantitative modalities (DTI metrics), normalization to 0-1 negatively affects the quantitative value of the metrics and should be avoided

## Evidence

- [methods] Z-score normalization then min-max rescaling preprocessing before training/metric computation: [evidence span withheld — anchor status: not_found]
