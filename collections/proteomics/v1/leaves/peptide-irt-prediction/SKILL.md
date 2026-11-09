---
name: peptide-irt-prediction
description: Use when generating an in silico spectral library from a proteome FASTA
  or peptide .csv and the output library needs iRT/retention-time annotation alongside
  fragment intensities for DIA analysis in Spectronaut.
when_to_use_negative:
- Input is already an experimental spectral library with measured retention times
  — rescoring or calibration (CollisionEnergyCalibration / Rescoring job types) is
  the appropriate Oktoberfest path instead
- No Koina server access is available, since iRT predictions are retrieved remotely
  via pr.Koina rather than computed locally in this workflow
edam_operation: http://edamontology.org/operation_0430
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Oktoberfest
  role: Python pipeline that digests the proteome, orchestrates iRT prediction calls
    and assembles the annotated spectral library
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Koina
  role: Prediction server hosting the irt and intensity models that pr.Koina/pr.Predictor
    query for peptide property predictions
- name: Prosit
  role: Deep learning model family for peptide property prediction (fragment intensities,
    iRT) underpinning the in silico library
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:d3a8f7b5dae5537fc5002a8bb97b6ee6e27e31762dfa33803baceb2279532659
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Koina
  - Oktoberfest
  - Prosit
  - Python
  edam_operation: http://edamontology.org/operation_0430
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# peptide-irt-prediction

## Summary

Retrieve indexed retention time (iRT) predictions for peptides from a Koina-hosted prediction model (e.g. Prosit ecosystem) as part of the Oktoberfest spectral library generation pipeline, so the resulting .msp/.csv libraries are retention-time annotated and Spectronaut-compatible.

## When to use

Use when generating an in silico spectral library from a proteome FASTA or peptide .csv and the output library needs iRT/retention-time annotation alongside fragment intensities for DIA analysis in Spectronaut.

## When NOT to use

- Input is already an experimental spectral library with measured retention times — rescoring or calibration (CollisionEnergyCalibration / Rescoring job types) is the appropriate Oktoberfest path instead
- No Koina server access is available, since iRT predictions are retrieved remotely via pr.Koina rather than computed locally in this workflow

## Inputs

- In-silico digested peptide list from a proteome .fasta file (or a .csv of requested precursor peptides)
- Oktoberfest configuration file of type "SpectralLibraryGeneration" specifying irt model name and Koina server settings

## Outputs

- iRT predictions per precursor peptide from the configured Koina irt model
- iRT-annotated Spectronaut-compatible .csv spectral library
- iRT-annotated .msp spectral library

## How to apply

After in-silico digestion of the proteome with pp.digest (trypsin, full digestion, 2 missed cleavages, peptide lengths 7-60, specialAas 'KR', concatenated target-decoy db), submit the precursor peptides to the Koina server (default koina.wilhelmlab.org:443, ssl true) using pr.Koina/pr.Predictor with the configured intensity and irt model names. Fragment-intensity and iRT predictions are retrieved together in batches of 10000 peptides, alongside spectral library options (HCD or CID fragmentation, collision energy 30 default, precursor charges [2,3], at most 1 methionine oxidation, minimal relative intensity threshold 5e-4). The iRT values are then merged with the predicted spectra via pp.generate_metadata, pp.merge_spectra_and_peptides, and pp.annotate_spectral_library before writing the library with pp.gen_lib. The rationale is that Prosit-class deep learning models provide peptide property predictions that enable high-quality in silico reference libraries with accurate retention-time information for downstream DIA extraction.

## Related tools

- **Oktoberfest** (Python pipeline that digests the proteome, orchestrates iRT prediction calls and assembles the annotated spectral library) — https://github.com/wilhelm-lab/oktoberfest
- **Koina** (Prediction server hosting the irt and intensity models that pr.Koina/pr.Predictor query for peptide property predictions)
- **Prosit** (Deep learning model family for peptide property prediction (fragment intensities, iRT) underpinning the in silico library)

## Examples

```
import oktoberfest as ok; ok.pr.Predictor.koina_predict(peptides, koina_server='koina.wilhelmlab.org:443', models=['Prosit_intensity', 'irt'])
```

## Evaluation signals

- The generated output folder contains both requested library formats: Spectronaut-compatible .csv and .msp files
- iRT predictions were returned by the configured Koina server (koina.wilhelmlab.org:443) for the digested precursors before library assembly
- Each precursor in the library carries both a fragment-intensity prediction and an iRT value merged via pp.merge_spectra_and_peptides
- Batch size did not exceed the configured 10000 peptides per prediction request
- Precursors conform to library constraints: peptide lengths 7-60, charges [2,3], at most 1 methionine oxidation

## Limitations

- iRT prediction requires access to a Koina server; the workflow's default endpoint is koina.wilhelmlab.org:443 with ssl true
- Prediction coverage is bounded by the digestion settings (trypsin, full digestion, 2 missed cleavages, lengths 7-60) and the nrOx = 1 methionine oxidation cap
- Minimal relative intensity threshold of 5e-4 filters weakly predicted fragments during library assembly

## Evidence

- [methods] iRT prediction via Koina in Oktoberfest: [evidence span withheld — anchor status: not_found]
- [abstract] Prosit enables in silico libraries: [evidence span withheld — anchor status: not_found]
- [methods] Koina server interface: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
- [readme] Library generation scope: [evidence span withheld — anchor status: external_doc]
