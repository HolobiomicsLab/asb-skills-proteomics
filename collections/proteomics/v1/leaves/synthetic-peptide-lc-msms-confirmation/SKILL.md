---
name: synthetic-peptide-lc-msms-confirmation
description: Use when a shortlist of candidate MHC-binding or neoantigen peptides
  (e.g. selected at q-value <= 0.01 from a DeepRescore result) needs experimental
  confirmation against synthetic peptide standards, or when verifying that rescued
  peptide IDs correspond to real analytes.
when_to_use_negative:
- The peptides of interest have not been synthesized or no synthetic standard LC-MS/MS
  data exist — spectral comparison to synthetic counterparts is impossible
- The goal is sensitivity benchmarking of search/rescoring engines across full datasets
  rather than confirmation of a specific shortlist of peptides
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ProteoWizard
  role: Converts the synthetic peptide Orbitrap Fusion raw MS/MS file to MGF format
    for downstream searching
- name: Comet
  role: Searches the synthetic peptide MGF data against a database containing only
    the selected synthetic peptides (v2018.01 rev. 4, unspecific cleavage, 10 ppm
    precursor, 0.02 Da MS/MS tolerance, length 8-25)
- name: DeepRescore
  role: 'Source of the candidate shortlist: peptides filtered from the peptide-level
    result (*_pep_final.tsv) at q-value <= 0.01 are the targets for synthetic validation'
  repo: https://github.com/bzhanglab/DeepRescore
- name: PDV
  role: Visualization of the synthetic peptide identification results after the confirmation
    table is built
  repo: https://github.com/wenbostar/PDV
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:ec854f5384eb6d34c95624133e7879eda3eb04280297c3f681d7f78d3ef835d1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bzhanglab/DeepRescore
  tools:
  - Comet
  - DeepRescore
  - PDV
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# synthetic-peptide-lc-msms-confirmation

## Summary

Validates immunopeptidomics peptide identifications (including DeepRescore-unique neoantigens and reference-protein-derived peptides) by acquiring LC-MS/MS of synthetic peptide standards and comparing experimental spectra to synthetic spectra, confirming identifications at high spectral similarity (PCC > 0.9) under 1% FDR.

## When to use

Use when a shortlist of candidate MHC-binding or neoantigen peptides (e.g. selected at q-value <= 0.01 from a DeepRescore result) needs experimental confirmation against synthetic peptide standards, or when verifying that rescued peptide IDs correspond to real analytes.

## When NOT to use

- The peptides of interest have not been synthesized or no synthetic standard LC-MS/MS data exist — spectral comparison to synthetic counterparts is impossible
- The goal is sensitivity benchmarking of search/rescoring engines across full datasets rather than confirmation of a specific shortlist of peptides

## Inputs

- DeepRescore peptide-level result file (*_pep_final.tsv) with a q-value column
- List of candidate peptide sequences selected for synthesis (e.g. 18 peptides at q-value <= 0.01 from dataset D1)
- Synthetic peptide LC-MS/MS raw file (Orbitrap Fusion, HCD CE32%, Top10 DDA)
- Experimental immunopeptidomics spectra corresponding to the candidate peptides

## Outputs

- Tabulated confirmation record (synthetic_peptide_validation.tsv) with peptide sequence, detected/not-detected status, and agreement with the reported outcome
- Comet search results against the synthetic-peptide-only database passing 1% FDR at PSM and peptide levels
- Spectral similarity comparison (PCC) between experimental and synthetic peptide spectra
- Optional PDV visualization of the synthetic peptide identifications

## How to apply

Start from the candidate peptide shortlist, e.g. filter the DeepRescore peptide-level result (*_pep_final.tsv) on the q-value column with q-value <= 0.01 to select the peptides to synthesize (here 18 peptides from dataset D1). Acquire LC-MS/MS data for the synthetic peptides on an Orbitrap Fusion using HCD at 32% collision energy with Top10 DDA, convert the raw file to MGF with ProteoWizard (v3.0.19014), and search the MGF against a database containing only the selected synthetic peptide sequences using Comet (v2018.01 rev. 4) with unspecific cleavage, variable oxidation of methionine, fixed carbamidomethyl of cysteine, 10 ppm precursor tolerance, 0.02 Da MS/MS tolerance, and peptide length 8-25. Control FDR at 1% at both PSM and peptide levels via the target-decoy strategy, then compare each detected synthetic peptide spectrum to the corresponding experimental sample spectrum; peptides showing high spectral similarity (PCC > 0.9, comparable to control peptides) are considered confirmed. Write a tabulated confirmation record (peptide sequence, detection status, agreement with the reported outcome) and optionally load identifications into PDV for visualization.

## Related tools

- **ProteoWizard** (Converts the synthetic peptide Orbitrap Fusion raw MS/MS file to MGF format for downstream searching)
- **Comet** (Searches the synthetic peptide MGF data against a database containing only the selected synthetic peptides (v2018.01 rev. 4, unspecific cleavage, 10 ppm precursor, 0.02 Da MS/MS tolerance, length 8-25))
- **DeepRescore** (Source of the candidate shortlist: peptides filtered from the peptide-level result (*_pep_final.tsv) at q-value <= 0.01 are the targets for synthetic validation) — https://github.com/bzhanglab/DeepRescore
- **PDV** (Visualization of the synthetic peptide identification results after the confirmation table is built) — https://github.com/wenbostar/PDV

## Examples

```
msconvert synthetic_raw.RAW --mgf -o . && comet -P comet.params synthetic.mgf  # unspecific cleavage, 10 ppm precursor, 0.02 Da MS/MS tol, length 8-25, 1% FDR
```

## Evaluation signals

- The confirmation table contains an entry for each of the 18 selected peptides with its detected/not-detected status
- Each entry's detection status matches the paper's reported synthetic peptide validation outcome
- All identifications pass 1% FDR at both PSM and peptide levels via the target-decoy strategy
- Confirmed DeepRescore-unique peptides show high spectral similarity between experimental and synthetic spectra (PCC > 0.9), comparable to control peptides
- Comet search parameters are consistent with the protocol: unspecific cleavage, oxidized M (variable), carbamidomethyl C (fixed), 10 ppm precursor, 0.02 Da fragment tolerance, length 8-25

## Limitations

- Validation coverage is limited to the synthesized shortlist (18 peptides here); 1 of the 12 DeepRescore-unique reference-protein-derived peptides did not show high similarity to its synthetic counterpart, so not every shortlisted peptide confirms
- Confirmation depends on the synthetic peptide being detectable under the acquisition settings used (Orbitrap Fusion, HCD CE32%, Top10 DDA); absence of detection does not by itself falsify the original identification

## Evidence

- [results] All three DeepRescore-unique neoantigens and 11 of the 12 DeepRescore-unique reference protein-derived peptides showed high similarity (PCC > 0.9) between experimental and synthetic peptide spectra, comparable to the control peptides.: [evidence span withheld — anchor status: not_found]
