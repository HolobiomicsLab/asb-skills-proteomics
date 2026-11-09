---
name: diffusion-mri-superresolution
description: Use when low-resolution multidimensional diffusion MRI volumes (e.g.,
  41×46×28 MUDI images with 1344 diffusion directions) must be upsampled to a higher
  target grid (e.g., 82×92×56), or when comparing GAN-based superresolution against
  3D Unet/3D cGAN baselines.
when_to_use_negative:
- The goal is cross-modal synthesis (e.g., PET, FA, or MD prediction from multimodal
  MRI) rather than spatial superresolution of diffusion MRI volumes
- Additional acquisition protocol data beyond the 6-element vector are required for
  further experiments — the authors lacked such data and could not extend the analyses
- Training-time or GPU memory budget cannot accommodate a 3D GAN — computational cost
  is the main limitation, requiring lengthy training and heavy GPU memory
edam_topics:
- http://edamontology.org/topic_3444
tools:
- name: SC-GAN
  role: Primary 3D conditional GAN (self-attention, spectral normalization, feature
    matching, AdaIN protocol injection) used to superresolve the MUDI data
  repo: https://github.com/Haoyulance/SC-GAN
- name: 3D Unet
  role: Baseline backbone substituted for SC-GAN to enable a fair comparison of superresolution
    performance
- name: 3D conditional GAN
  role: Baseline GAN backbone compared against SC-GAN on the MUDI superresolution
    task
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:1439fb5a9a590ebe625c4675e0cc2bc2215ea32e988eaf6e488d94ebaf8987c2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 3D Unet
  - 3D conditional GAN
  - SC-GAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# diffusion-mri-superresolution

## Summary

Superresolve low-resolution multidimensional diffusion MRI (MUDI) volumes to a higher target resolution using a 3D self-attention conditional GAN (SC-GAN), and benchmark against 3D Unet and 3D conditional GAN baselines using NRMSE, peak SNR, and structural similarity.

## When to use

Use when low-resolution multidimensional diffusion MRI volumes (e.g., 41×46×28 MUDI images with 1344 diffusion directions) must be upsampled to a higher target grid (e.g., 82×92×56), or when comparing GAN-based superresolution against 3D Unet/3D cGAN baselines.

## When NOT to use

- The goal is cross-modal synthesis (e.g., PET, FA, or MD prediction from multimodal MRI) rather than spatial superresolution of diffusion MRI volumes
- Additional acquisition protocol data beyond the 6-element vector are required for further experiments — the authors lacked such data and could not extend the analyses
- Training-time or GPU memory budget cannot accommodate a 3D GAN — computational cost is the main limitation, requiring lengthy training and heavy GPU memory

## Inputs

- MUDI challenge training sets of image size 41 × 46 × 28 × 1344 (four sets) and one validation set of the same size
- Training patches of size 41 × 46 × 28 formed from independent volumes of the subsampled images
- 6-element acquisition-protocol vector (gradient-encoding directions x/y/z, b-value, TE, TI)

## Outputs

- Superresolved validation volumes at the 82 × 92 × 56 target resolution
- Comparison table of NRMSE, peak SNR, and structural similarity metrics for SC-GAN vs 3D Unet vs 3D cGAN (e.g., mudi_superresolution_results.csv)

## How to apply

Load the Computational Diffusion MRI Workshop 2020 MUDI challenge data (four training sets and one validation set of image size 41×46×28×1344) and form training patches of size 41×46×28 from independent volumes of the subsampled images. Encode the acquisition-protocol information as a 6-element vector (gradient-encoding directions x/y/z, b-value, TE, TI) and inject it into each discriminator feature map via adaptive instance normalization (AdaIN) with trained affine transformations. Train SC-GAN with batch size 4 and learning rate 0.001 with cosine decay using a progressive two-step strategy: first superresolve to the 82×92×56 target resolution, then train a second SC-GAN to refine the reconstructed data and further reduce mean squared error. Retrain with the backbone swapped to 3D Unet and 3D cGAN to keep the comparison fair, then evaluate on the validation subject with normalized RMS error, peak SNR, and structural similarity, and compile results into a comparison table (e.g., mudi_superresolution_results.csv). SC-GAN achieved the lowest average MSE (2.15) versus 3D Unet (2.52) and 3D cGAN (2.41).

## Related tools

- **SC-GAN** (Primary 3D conditional GAN (self-attention, spectral normalization, feature matching, AdaIN protocol injection) used to superresolve the MUDI data) — https://github.com/Haoyulance/SC-GAN
- **3D Unet** (Baseline backbone substituted for SC-GAN to enable a fair comparison of superresolution performance)
- **3D conditional GAN** (Baseline GAN backbone compared against SC-GAN on the MUDI superresolution task)

## Examples

```
python training.py --trainig_size= --gpu= --epoches= --img_size= --data_dir= --modalities= --logdir=
```

## Evaluation signals

- SC-GAN achieves lower NRMSE on the validation subject than both 3D Unet and 3D cGAN (consistent with the paper's reported superiority)
- SC-GAN attains the lowest average mean squared error (≈2.15) versus 3D Unet (≈2.52) and 3D cGAN (≈2.41)
- Peak SNR and structural similarity for SC-GAN are higher than both baselines on the validation data
- Output volumes have the 82 × 92 × 56 target resolution, and the 6-element protocol vector is injected into every discriminator feature map via AdaIN

## Limitations

- Computational cost is the main limitation of SC-GAN; it requires lengthy training and heavy computational resources, such as GPU memory
- The authors lacked additional acquisition protocol data required to perform further experiments and analyses on the superresolution application
- Correlation analysis between acquisition protocol data and diffusion MRI superresolution performance was left as future work

## Evidence
