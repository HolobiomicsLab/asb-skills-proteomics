---
name: spectral-normalization-implementation
description: Use when building or reproducing a GAN (e.g., 3D conditional GAN for
  MRI/PET synthesis) whose training is unstable, and you need weight normalization
  in generator and discriminator layers as done in SC-GAN.
when_to_use_negative:
- The task is a 2D synthesis problem or non-imaging data — the reference implementation
  and its stabilization recipe are built for 3D conditional GAN on 256x256x256 neuroimaging
  volumes
- 'Normalization of target intensities is unacceptable: for quantitative modalities
  such as DTI metrics, the article warns that Z-score/min-max normalization ''negatively
  affects the quantitative value of the metrics'' and should be avoided'
- You need fast, lightweight training — spectral normalization plus 3D convolutions
  makes SC-GAN computationally expensive, requiring lengthy training and heavy GPU
  memory
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_3384
tools:
- name: SC-GAN
  role: Reference 3D conditional GAN implementation in which spectral normalization
    is applied to stabilize training; reference code for structural comparison
  repo: https://github.com/Haoyulance/SC-GAN
- name: TensorFlow
  role: Deep-learning framework used to implement the spectrally normalized generator
    and discriminator
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:3aea7dfe360f656b16d21609a33aebf301db90d6d79e355e5d50bf60f909e741
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - SC-GAN
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_3384
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# spectral-normalization-implementation

## Summary

Implementation of spectral normalization as a training-stabilization technique in the SC-GAN 3D conditional GAN, applied to weights in both the generator and the discriminator to stabilize training and ensure optimization convergence for multimodal neuroimaging synthesis.

## When to use

Use when building or reproducing a GAN (e.g., 3D conditional GAN for MRI/PET synthesis) whose training is unstable, and you need weight normalization in generator and discriminator layers as done in SC-GAN.

## When NOT to use

- The task is a 2D synthesis problem or non-imaging data — the reference implementation and its stabilization recipe are built for 3D conditional GAN on 256x256x256 neuroimaging volumes
- Normalization of target intensities is unacceptable: for quantitative modalities such as DTI metrics, the article warns that Z-score/min-max normalization 'negatively affects the quantitative value of the metrics' and should be avoided
- You need fast, lightweight training — spectral normalization plus 3D convolutions makes SC-GAN computationally expensive, requiring lengthy training and heavy GPU memory

## Inputs

- Paired multimodal neuroimaging volumes (.nii.gz) organized as data/train, data/val, data/test subject folders with input modality_*.nii.gz and target.nii.gz
- Training configuration: --img_size 256, --modalities such as flair_t1w_av45, --lr 0.001, --g_reg 0.001, --d_reg 0.001

## Outputs

- SC-GAN generator/discriminator TensorFlow model code with spectral normalization applied in both networks
- Synthesized target modality volumes (e.g., av45 PET, FA, or MD maps) from the trained normalized network
- Tensorboard training logs (--logdir)

## How to apply

Implement spectral normalization inside both the generator and the discriminator of the 3D conditional GAN, alongside the 3D self-attention module, using the TensorFlow environment specified in the SC-GAN repository's requirements.txt (Python 3.6.9). Pair it with the composite SC-GAN loss — cGAN adversarial term, L1 loss (weight 200), feature matching loss (weight 20), brain-area RMSE loss (weight 200), and L2 regularization with g_reg = d_reg = 0.001 — since stabilization relies on this combination rather than normalization alone. Train with batch size 1 on 256x256x256 inputs (e.g., T1w/FLAIR to av45 PET synthesis) and verify training converges without the divergence typical of unnormalized 3D GANs. Structurally compare your normalized generator/discriminator layers against the reference implementation at github.com/Haoyulance/SC-GAN. Note that the ablation study ranked spectral normalization as a contributor to performance after adversarial learning and the self-attention module, so it should be ablated (SC-GAN with vs. without the module) rather than assumed dominant.

## Related tools

- **SC-GAN** (Reference 3D conditional GAN implementation in which spectral normalization is applied to stabilize training; reference code for structural comparison) — https://github.com/Haoyulance/SC-GAN
- **TensorFlow** (Deep-learning framework used to implement the spectrally normalized generator and discriminator)

## Examples

```
python training.py --training_size= --gpu= --epoches=120 --img_size=256 --data_dir=data --modalities=flair_t1w_av45 --lr=0.001 --g_reg=0.001 --d_reg=0.001 --logdir=logs
```

## Evaluation signals

- Model instantiation of the spectrally normalized generator and discriminator runs without error (sc_gan_model.py builds all layers and attention blocks)
- Structural comparison against github.com/Haoyulance/SC-GAN shows spectral normalization present in both generator and discriminator
- Training converges over the default 120 epochs at lr 0.001 with g_reg = d_reg = 0.001 without mode divergence
- Synthesis quality on held-out ADNI test subjects matches or approaches reference levels on normalized RMS error, peak SNR, and structural similarity
- Ablation removing spectral normalization degrades performance, consistent with the paper's finding that it contributes after adversarial learning and self-attention

## Limitations

- Computational cost is the main limitation of SC-GAN: it requires lengthy training and heavy computational resources such as GPU memory
- The ablation experiment showed the major contributors to improved performance are adversarial learning and the self-attention module, followed by spectral normalization — normalization alone is not sufficient
- Normalizing target images to 0-1 for stable training negatively affects the quantitative value of DTI metrics, so the stabilization/normalization pipeline interacts with intensity preprocessing choices

## Evidence
