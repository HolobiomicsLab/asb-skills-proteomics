---
name: immunopeptidomics-peptide-validation-tabulation
description: Use when a paper reports a subset of MHC-binding or neoantigen peptide
  identifications selected for synthetic peptide validation and you must independently
  confirm detection status by searching a synthetic-peptide-only database and comparing
  to the reported outcome.
when_to_use_negative:
- The input data are from standard enzymatic-digestion proteomics, where unspecific
  cleavage searching would inflate the search space unnecessarily; this skill is specific
  to immunopeptidomics (MHC-bound peptide) validation.
- The candidate peptide list is not already filtered to the paper's validation set
  — do not apply this to unfiltered PSM tables or peptides without purchased synthetic
  counterparts.
- A confirmation table already exists; this skill is for independent detection and
  comparison against the reported outcome, not for re-annotating an existing table.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: ProteoWizard
  role: Converts the synthetic peptide LC-MS/MS raw file to MGF format for downstream
    searching (v3.0.19014)
- name: Comet
  role: Searches the synthetic-peptide MGF data against a database of only the selected
    synthetic peptides with unspecific cleavage, 10 ppm precursor and 0.02 Da MS/MS
    tolerance (v2018.01 rev. 4)
  repo: http://comet-ms.sourceforge.net/
- name: DeepRescore
  role: Source of the candidate peptide list and q-value column used to select validation
    peptides
  repo: https://github.com/bzhanglab/DeepRescore
- name: PDV
  role: Visualization of the synthetic peptide identification results
  repo: https://github.com/wenbostar/PDV
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:d268a4b3db6555e54ff3992caa6365f0d0c3eeb5deff27d2b30a30de45969bb4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Comet
  - DeepRescore
  - PDV
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# immunopeptidomics-peptide-validation-tabulation

## Summary

Validates a filtered set of candidate immunopeptidomics peptide identifications (e.g., DeepRescore-unique peptides selected at q-value <= 0.01) by searching LC-MS/MS data of purchased synthetic peptides and tabulating which candidates were experimentally confirmed via spectral similarity (PCC > 0.9).

## When to use

Use when a paper reports a subset of MHC-binding or neoantigen peptide identifications selected for synthetic peptide validation and you must independently confirm detection status by searching a synthetic-peptide-only database and comparing to the reported outcome.

## When NOT to use

- The input data are from standard enzymatic-digestion proteomics, where unspecific cleavage searching would inflate the search space unnecessarily; this skill is specific to immunopeptidomics (MHC-bound peptide) validation.
- The candidate peptide list is not already filtered to the paper's validation set — do not apply this to unfiltered PSM tables or peptides without purchased synthetic counterparts.
- A confirmation table already exists; this skill is for independent detection and comparison against the reported outcome, not for re-annotating an existing table.

## Inputs

- Peptide-level identification result from DeepRescore or another search/rescoring engine (e.g., *_pep_final.tsv with a q-value column)
- List of candidate peptides selected for synthetic validation (e.g., 18 peptides at q-value <= 0.01 from dataset D1)
- Raw LC-MS/MS file of synthetic peptides (e.g., Orbitrap Fusion HCD CE32% Top10 DDA)
- Reported synthetic peptide validation table/figure from the paper

## Outputs

- Tabulated confirmation record (peptide sequence, detection status, agreement with reported outcome), e.g., synthetic_peptide_validation.tsv
- Comet identification results for the synthetic-peptide-only database passing 1% FDR at PSM and peptide levels
- Optional PDV visualization of the identification result

## How to apply

Start from the tool's peptide-level result (e.g., DeepRescore's *_pep_final.tsv) and extract the candidates selected for validation by filtering the q-value column at q-value <= 0.01. Convert the synthetic peptide LC-MS/MS raw file (e.g., Orbitrap Fusion, HCD CE32%, Top10 DDA) to MGF with ProteoWizard (v3.0.19014), then search the MGF against a database containing only the selected synthetic peptides using Comet (v2018.01 rev. 4) with unspecific cleavage (since immunopeptidomics peptides are not enzymatically digested), oxidation of methionine as variable and carbamidomethyl of cysteine as fixed modification, 10 ppm precursor tolerance, 0.02 Da MS/MS tolerance, peptide length 8-25, and 1% FDR at both PSM and peptide levels via target-decoy. Compare detected synthetic peptides to the paper's validation table/figure, and optionally assess spectral similarity between experimental and synthetic spectra (high agreement, PCC > 0.9, comparable to control peptides). Write one row per candidate peptide to a tabulated confirmation record and load identifications into PDV for visualization.

## Related tools

- **ProteoWizard** (Converts the synthetic peptide LC-MS/MS raw file to MGF format for downstream searching (v3.0.19014))
- **Comet** (Searches the synthetic-peptide MGF data against a database of only the selected synthetic peptides with unspecific cleavage, 10 ppm precursor and 0.02 Da MS/MS tolerance (v2018.01 rev. 4)) — http://comet-ms.sourceforge.net/
- **DeepRescore** (Source of the candidate peptide list and q-value column used to select validation peptides) — https://github.com/bzhanglab/DeepRescore
- **PDV** (Visualization of the synthetic peptide identification results) — https://github.com/wenbostar/PDV

## Examples

```
msconvert synthetic_peptides.raw --mgf --outdir mgf/
```

## Evaluation signals

- The confirmation table contains an entry for each of the selected validation peptides (e.g., all 18 candidates from D1) with a detected/not-detected status.
- Synthetic peptide identifications pass 1% FDR at both PSM and peptide levels via the target-decoy strategy.
- Each detected/not-detected status matches the paper's reported synthetic peptide validation outcome.
- Spectral similarity between experimental and synthetic peptide spectra reaches PCC > 0.9, comparable to control peptides.
- Comet search parameters are constrained as reported: unspecific cleavage, oxidized methionine variable, carbamidomethyl cysteine fixed, 10 ppm precursor, 0.02 Da MS/MS tolerance, peptide length 8-25.

## Limitations

- Unspecific cleavage in immunopeptidomics creates an inflated search space that can lead to a high false positive rate and low sensitivity, so strict FDR control (1% at PSM and peptide levels) is essential.
- Validation only covers the peptides for which synthetic counterparts were purchased and searched; peptides not in the synthetic database cannot be confirmed.
- Similarity assessment depends on matching acquisition conditions (e.g., HCD CE32% Orbitrap Fusion Top10 DDA) between experimental and synthetic runs for meaningful PCC comparison.

## Evidence
