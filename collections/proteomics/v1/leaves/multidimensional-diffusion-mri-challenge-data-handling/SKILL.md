---
name: multidimensional-diffusion-mri-challenge-data-handling
description: Use when training or evaluating superresolution models on the Computational
  Diffusion MRI Workshop 2020 MUDI challenge dataset, where subsampled low-resolution
  volumes of size 41 × 46 × 28 must be superresolved to the 82 × 92 × 56 target resolution.
when_to_use_negative:
- The synthesis task is cross-modal prediction (PET, fractional anisotropy, or mean
  diffusivity from multimodal MRI) rather than MUDI superresolution — use the ADNI-3-based
  synthesis workflow instead.
- Additional acquisition-protocol data beyond what the MUDI challenge provides are
  needed for further experiments and analyses; the authors note they lacked such data.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_3473
tools:
- name: SC-GAN
  role: Primary superresolution model trained on MUDI challenge data in a progressive
    two-step strategy with AdaIN-injected acquisition-protocol conditioning
  repo: https://github.com/Haoyulance/SC-GAN
- name: 3D Unet
  role: Baseline backbone replacing the SC-GAN generator for fair comparison on the
    MUDI superresolution task
- name: 3D conditional GAN
  role: Baseline backbone replacing the SC-GAN generator for fair comparison on the
    MUDI superresolution task
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:e288589decec461a15b6fc26b52b6abb8fb7e4cb19d1f2d8b7345ca2d2ddbf30
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 3D Unet
  - 3D conditional GAN
  - SC-GAN
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# multidimensional-diffusion-mri-challenge-data-handling

## Summary

Handles the Computational Diffusion MRI Workshop 2020 MUDI challenge data (four training sets of 41 × 46 × 28 × 1344 and one validation set of the same size) for superresolution of multidimensional diffusion MRI with SC-GAN, 3D Unet, and 3D cGAN.

## When to use

Use when training or evaluating superresolution models on the Computational Diffusion MRI Workshop 2020 MUDI challenge dataset, where subsampled low-resolution volumes of size 41 × 46 × 28 must be superresolved to the 82 × 92 × 56 target resolution.

## When NOT to use

- The synthesis task is cross-modal prediction (PET, fractional anisotropy, or mean diffusivity from multimodal MRI) rather than MUDI superresolution — use the ADNI-3-based synthesis workflow instead.
- Additional acquisition-protocol data beyond what the MUDI challenge provides are needed for further experiments and analyses; the authors note they lacked such data.

## Inputs

- Computational Diffusion MRI Workshop 2020 MUDI challenge dataset: four training sets of image size 41 × 46 × 28 × 1344 and one validation set of the same size
- Acquisition-protocol information per volume: gradient-encoding directions x/y/z, b-value, TE, TI (encoded as a 6-element vector)

## Outputs

- Training patches of size 41 × 46 × 28 formed from independent volumes of the subsampled images
- Superresolved volumes at the 82 × 92 × 56 target resolution
- Comparison table of NRMSE, peak SNR, and structural similarity for SC-GAN vs 3D Unet vs 3D cGAN (e.g., mudi_superresolution_results.csv)

## How to apply

Load the MUDI challenge data (four training sets and one validation set, each of image size 41 × 46 × 28 × 1344, where 1344 is the number of acquisition samples) and form training patches from independent volumes of the subsampled images with spatial size 41 × 46 × 28. Encode the acquisition-protocol information as a 6-element vector (gradient-encoding directions x/y/z, b-value, TE, TI) and inject it into each discriminator feature map via adaptive instance normalization (AdaIN) with trained affine transformations. Train SC-GAN with batch size 4 and learning rate 0.001 using cosine decay, following a progressive two-step strategy: first superresolve the low-resolution data to the 82 × 92 × 56 target resolution, then train a second SC-GAN to refine the reconstructed data. Repeat training with the SC-GAN backbone replaced by 3D Unet and 3D cGAN for a fair comparison, and evaluate with normalized RMSE, peak SNR, and structural similarity on the validation subject.

## Related tools

- **SC-GAN** (Primary superresolution model trained on MUDI challenge data in a progressive two-step strategy with AdaIN-injected acquisition-protocol conditioning) — https://github.com/Haoyulance/SC-GAN
- **3D Unet** (Baseline backbone replacing the SC-GAN generator for fair comparison on the MUDI superresolution task)
- **3D conditional GAN** (Baseline backbone replacing the SC-GAN generator for fair comparison on the MUDI superresolution task)

## Examples

```
python training.py --trainig_size=4 --gpu=0 --epoches=120 --img_size=28 --data_dir=data/train --modalities=mudi_lr_mudi_hr --logdir=logs/scgan_mudi
```

## Evaluation signals

- SC-GAN achieves the lowest average mean squared error (2.15) versus 3D Unet (2.52) and 3D cGAN (2.41) on the validation subject
- SC-GAN achieves lower normalized RMSE (and higher peak SNR and structural similarity) than both the 3D Unet and 3D cGAN baselines on the validation data
- Training patches are 41 × 46 × 28 and the two-step progressive output matches the 82 × 92 × 56 target resolution
- Acquisition-protocol conditioning is a 6-element vector injected via AdaIN into every discriminator feature map

## Limitations

- Computational cost is the main limitation of SC-GAN; it requires lengthy training and heavy GPU memory.
- For the superresolution application, the authors lacked the additional acquisition protocol data required to perform further experiments and analyses.
- Correlation analysis between acquisition protocol data and superresolution performance is left as future work.

## Evidence
