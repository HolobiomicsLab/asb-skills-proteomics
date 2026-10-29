---
name: cross-model-benchmark-comparison
description: Use when several trained synthesis/generation models must be compared
  on identical test data with quantitative image-quality metrics to establish which
  architecture outperforms the baselines.
when_to_use_negative:
- Input is a single model's output with no baseline predictions to compare against
  — use absolute quality assessment instead
- Test subjects differ across models (each model was evaluated on its own split) —
  comparison is then confounded and invalid
- Target modality is a quantitative map (e.g., DTI metrics) where 0-1 min-max rescaling
  distorts the quantitative values — the article warns normalization negatively affects
  metric interpretation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_3474
tools:
- name: SC-GAN
  role: Primary model under benchmark; generates synthesized MD maps evaluated against
    ground truth
  repo: https://github.com/Haoyulance/SC-GAN
- name: 2D conditional GAN
  role: Baseline trained with mu=100, batch size 4, lr 0.0002; its 3D output is evaluated
    directly in 3D for fair comparison
- name: 3D conditional GAN
  role: Baseline trained with mu=200, batch size 1, lr 0.002; reference for 3D architecture
    comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:8f58b67742bb8694dc3482589626df9fbf4602e9c5e0819267ecff2b14eaaa57
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 2D conditional GAN
  - 3D conditional GAN
  - SC-GAN
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# cross-model-benchmark-comparison

## Summary

Benchmark multiple generative synthesis models (SC-GAN vs 2D cGAN, 3D cGAN, Att cGAN) on a shared test split by computing normalized RMSE, peak SNR, and structural similarity between synthesized and ground-truth images, then assembling a comparison table to quantify relative performance (e.g., SC-GAN's 29% lower NRMSE than 2D cGAN on the ADNI-3 MD task).

## When to use

Use when several trained synthesis/generation models must be compared on identical test data with quantitative image-quality metrics to establish which architecture outperforms the baselines.

## When NOT to use

- Input is a single model's output with no baseline predictions to compare against — use absolute quality assessment instead
- Test subjects differ across models (each model was evaluated on its own split) — comparison is then confounded and invalid
- Target modality is a quantitative map (e.g., DTI metrics) where 0-1 min-max rescaling distorts the quantitative values — the article warns normalization negatively affects metric interpretation

## Inputs

- Synthesized MD (or PET/FA) volumes from each model on the same 99-subject ADNI-3 test split (256 x 256 x 256 NIfTI)
- Ground-truth MD maps for the test subjects
- Trained model checkpoints/labels for SC-GAN, 2D cGAN, 3D cGAN, and Att cGAN

## Outputs

- Comparison table of NRMSE, peak SNR, and structural similarity per model (md_synthesis_comparison.csv)
- Ranking of models consistent across all three evaluation metrics

## How to apply

Run each trained model's testing script over the same held-out test split (e.g., 99 ADNI-3 subjects, T1w and FLAIR inputs, MD targets at 256 x 256 x 256). Generate synthesized maps for SC-GAN and each baseline (2D cGAN with mu=100, 3D cGAN with mu=200, Att cGAN with mu=200), evaluating the 3D output of the 2D network directly in 3D for fairness. For every subject, compute normalized RMSE, peak SNR, and structural similarity between synthesized and ground-truth volumes. Assemble the per-model metrics into a table (e.g., md_synthesis_comparison.csv) and verify the ranking is consistent with the paper's reported result that SC-GAN beat all baselines across all metrics, with ~29% lower prediction error than 2D cGAN on the MD task.

## Related tools

- **SC-GAN** (Primary model under benchmark; generates synthesized MD maps evaluated against ground truth) — https://github.com/Haoyulance/SC-GAN
- **2D conditional GAN** (Baseline trained with mu=100, batch size 4, lr 0.0002; its 3D output is evaluated directly in 3D for fair comparison)
- **3D conditional GAN** (Baseline trained with mu=200, batch size 1, lr 0.002; reference for 3D architecture comparison)

## Examples

```
python testing.py --test_size=99 --gpu=0 --img_size=256 --data_dir=data/test --modalities=flair_t1w_md --output=results/scgan_md
```

## Evaluation signals

- Comparison table contains NRMSE, peak SNR, and structural similarity for all four networks
- SC-GAN ranks first on all three metrics, consistent with the paper's claim of outperforming 3D conditional GAN 'in all three tasks across all evaluation metrics'
- SC-GAN's NRMSE on the MD task is approximately 29% lower than 2D conditional GAN's
- All models evaluated on the identical 99-subject test split, with the 2D network's output assessed in 3D for fairness
- Metrics computed voxel-wise between synthesized and ground-truth volumes on the same intensity-normalized scale (0-1 min-max rescaled)

## Limitations

- Computational cost is the main limitation of SC-GAN: lengthy training and heavy GPU memory requirements, which constrains how extensively baselines can be re-tuned
- Hyperparameters were tuned via 5-fold cross-validation; differences in tuning budget across baselines could bias the comparison
- The paper's normalization to 0-1 for quantitative DTI targets negatively affects the quantitative value of metrics, so benchmark numbers are scale-dependent

## Evidence
