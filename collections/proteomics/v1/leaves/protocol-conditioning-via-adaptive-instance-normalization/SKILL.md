---
name: protocol-conditioning-via-adaptive-instance-normalization
description: Use when training a conditional GAN (e.g., SC-GAN) for diffusion-MRI
  superresolution where each volume has distinct acquisition-protocol parameters that
  must condition the discriminator's feature maps.
when_to_use_negative:
- The task lacks acquisition-protocol metadata to encode — as the authors note for
  MUDI, 'we lacked the additional acquisition protocol data required to perform further
  experiments and analyses', making protocol conditioning impossible
- The conditioning variable is another image modality (e.g., T1w/FLAIR for PET or
  FA/MD synthesis) rather than scalar acquisition parameters — use the repository's
  standard --modalities conditioning path instead of the AdaIN protocol vector
edam_topics:
- http://edamontology.org/topic_3399
- http://edamontology.org/topic_3473
tools:
- name: SC-GAN
  role: Backbone conditional GAN whose discriminator feature maps receive the protocol
    vector via AdaIN during MUDI superresolution training
  repo: https://github.com/Haoyulance/SC-GAN
- name: 3D conditional GAN
  role: Baseline without protocol-aware AdaIN conditioning used for fair comparison
    in the repeat training
- name: 3D Unet
  role: Non-GAN baseline trained with the same data to benchmark the conditioned SC-GAN's
    superresolution accuracy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:86d9db4018d6435836d41cb5be862467e9b9c3a0486d39cc0538abe339cef600
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
  - http://edamontology.org/topic_3399
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# protocol-conditioning-via-adaptive-instance-normalization

## Summary

Injects acquisition-protocol metadata (e.g., diffusion-MRI gradient directions, b-value, TE, TI) into a conditional GAN's discriminator as a compact parameter vector via adaptive instance normalization (AdaIN), so generated superresolution predictions honor the specific acquisition protocol.

## When to use

Use when training a conditional GAN (e.g., SC-GAN) for diffusion-MRI superresolution where each volume has distinct acquisition-protocol parameters that must condition the discriminator's feature maps.

## When NOT to use

- The task lacks acquisition-protocol metadata to encode — as the authors note for MUDI, 'we lacked the additional acquisition protocol data required to perform further experiments and analyses', making protocol conditioning impossible
- The conditioning variable is another image modality (e.g., T1w/FLAIR for PET or FA/MD synthesis) rather than scalar acquisition parameters — use the repository's standard --modalities conditioning path instead of the AdaIN protocol vector

## Inputs

- Subsampled multidimensional diffusion MRI (MUDI) volumes of size 41×46×28×1344 from the Computational Diffusion MRI Workshop 2020 MUDI challenge dataset
- Acquisition-protocol parameters per volume (gradient-encoding directions x/y/z, b-value, TE, TI) encoded as a 6-element vector
- Training patches formed from independent volumes of the subsampled images (41×46×28) and one validation subject of the same size

## Outputs

- Superresolved volumes at the 82×92×56 target resolution, further refined by a second SC-GAN stage
- Comparison metrics (normalized RMSE, peak SNR, structural similarity, mean squared error) compiled per model, e.g., mudi_superresolution_results.csv

## How to apply

Encode the acquisition-protocol information as a 6-element vector containing gradient-encoding directions x/y/z, b-value, TE, and TI. Inject this vector into each discriminator feature map via adaptive instance normalization (AdaIN) with trained affine transformations, rather than concatenating it as an input channel. During training (batch size 4, learning rate 0.001 with cosine decay), use a progressive two-step strategy: first superresolve the 41×46×28 low-resolution MUDI volumes to the 82×92×56 target resolution, then train a second SC-GAN to refine the reconstruction and further reduce mean squared error. The rationale is that protocol parameters determine the mapping from subsampled to full-resolution data, so the discriminator needs protocol-aware feature statistics to judge whether generated volumes are consistent with the stated acquisition.

## Related tools

- **SC-GAN** (Backbone conditional GAN whose discriminator feature maps receive the protocol vector via AdaIN during MUDI superresolution training) — https://github.com/Haoyulance/SC-GAN
- **3D conditional GAN** (Baseline without protocol-aware AdaIN conditioning used for fair comparison in the repeat training)
- **3D Unet** (Non-GAN baseline trained with the same data to benchmark the conditioned SC-GAN's superresolution accuracy)

## Examples

```
python training.py --trainig_size=4 --gpu=0 --epoches=120 --img_size=256 --data_dir=data/train --modalities=flair_t1w_av45 --lr=0.001
```

## Evaluation signals

- The protocol vector has exactly 6 elements (gradient directions x/y/z, b-value, TE, TI) and AdaIN affine transformations are trainable parameters updated during training
- On the validation subject, SC-GAN achieves the lowest average mean squared error (~2.15) versus 3D cGAN (~2.41) and 3D Unet (~2.52)
- Normalized RMSE decreases while peak SNR and structural similarity increase relative to both baselines, consistent with SC-GAN's reported superior prediction
- Validation is run on a held-out subject (same 41×46×28×1344 dimensions) never seen during patch-based training

## Limitations

- Computational cost is the main limitation of SC-GAN: it requires lengthy training and heavy GPU memory
- For the MUDI superresolution application the authors lacked additional acquisition protocol data required for further experiments and analyses
- Normalization choices matter: normalizing target quantitative modalities to 0–1 for stable training negatively affects the quantitative value of DTI metrics

## Evidence

- [other] Encode the acquisition-protocol information as a 6-element vector (gradient-encoding directions x/y/z, b-value, TE, TI) and inject it into each discriminator feature map via AdaIN: [evidence span withheld — anchor status: not_found]
