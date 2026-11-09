---
name: spectral-count-protein-quantification
description: Use when raw LC-MS/MS data have been searched with the Morpheus peptide-MS/MS
  matching engine and per-protein relative abundances must be quantified label-free
  from spectral counts, especially across protein families or isoforms.
when_to_use_negative:
- Your data were searched with an engine other than Morpheus — the README states data
  must be searched with the Morpheus Mass Spectrometry Search Engine
- Quantification requires MS1 peak intensities (e.g., isobaric-tag or extracted-ion-chromatogram
  workflows) rather than spectral counting
- Input is already a normalized protein abundance table — this skill operates on raw
  peptide-MS/MS identification output
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Morpheus Spectral Counter (MSpC)
  role: Application that directly calculates distributed and unique NSAF values from
    Morpheus output; the tool performing this quantification
  repo: github.com/dcgemperline/Morpheus_SpC
- name: Morpheus
  role: Upstream peptide-MS/MS matching engine that must search the data before NSAF
    computation; output feeds directly into MSpC
  repo: cwenger.github.io/Morpheus
- name: C#
  role: Implementation language of MSpC; shared- and unique-peptide counting logic
    is written in C#
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201500420
    title: 'Morpheus Spectral Counter: A computational tool for label‐free quantitative
      mass spectrometry using the Morpheus search engine'
  source_content_hash: sha256:6c24369f7d6d25d993e5478b4bd83fd5f2706888f13659550c8792b612e45d3d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C#
  - Morpheus
  - Morpheus Spectral Counter (MSpC)
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spectral-count-protein-quantification

## Summary

Compute Normalized Spectral Abundance Factor (NSAF) values for label-free quantitative proteomics by aggregating spectral counts per protein from Morpheus search-engine output, including distributed NSAF for shared peptides and unique NSAF for isoforms/proteoforms.

## When to use

Use when raw LC-MS/MS data have been searched with the Morpheus peptide-MS/MS matching engine and per-protein relative abundances must be quantified label-free from spectral counts, especially across protein families or isoforms.

## When NOT to use

- Your data were searched with an engine other than Morpheus — the README states data must be searched with the Morpheus Mass Spectrometry Search Engine
- Quantification requires MS1 peak intensities (e.g., isobaric-tag or extracted-ion-chromatogram workflows) rather than spectral counting
- Input is already a normalized protein abundance table — this skill operates on raw peptide-MS/MS identification output

## Inputs

- Morpheus search-engine output file with peptide-spectrum matches and peptide-to-protein mappings (from high-resolution accurate-mass LC-MS/MS data)
- Protein sequence lengths for length normalization

## Outputs

- nsaf_values.csv — per-protein table of spectral counts, distributed NSAF, and unique NSAF

## How to apply

Load the Morpheus output file containing spectrum-to-protein identifications from the searched high-resolution accurate-mass dataset. Aggregate spectral counts per protein, separating shared peptides (distributed counting within protein families) from unique peptides (unique counting). Apply length normalization by dividing each protein's spectral count by its sequence length per the NSAF formulation, then normalize the length-corrected counts by their sum across all identified proteins to yield distributed NSAF and unique NSAF values. Write the per-protein table of spectral counts, distributed NSAF, and unique NSAF to nsaf_values.csv. Decision point: isoforms/proteoforms within a family receive non-zero unique NSAF only from peptides exclusive to them, while distributed NSAF spreads shared spectra equally among family members after length correction.

## Related tools

- **Morpheus Spectral Counter (MSpC)** (Application that directly calculates distributed and unique NSAF values from Morpheus output; the tool performing this quantification) — github.com/dcgemperline/Morpheus_SpC
- **Morpheus** (Upstream peptide-MS/MS matching engine that must search the data before NSAF computation; output feeds directly into MSpC) — cwenger.github.io/Morpheus
- **C#** (Implementation language of MSpC; shared- and unique-peptide counting logic is written in C#)

## Evaluation signals

- nsaf_values.csv exists and is a well-formed table with distributed-NSAF and unique-NSAF columns
- NSAF columns sum to approximately 1 across all identified proteins
- Every protein reported in the Morpheus output has a corresponding row in nsaf_values.csv
- For each protein family, distributed NSAF values sum consistently with the family's total spectral counts
- Isoforms sharing all peptides show non-zero distributed NSAF but zero unique NSAF

## Limitations

- Data must first be searched with the Morpheus Mass Spectrometry Search Engine; other search engines are not supported
- Quantification is spectral-count based, which has a greater dynamic range than isobaric tags but relies on identification counts rather than MS1 peak extraction

## Evidence
