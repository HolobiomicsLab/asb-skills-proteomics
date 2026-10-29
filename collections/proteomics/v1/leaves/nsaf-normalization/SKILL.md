---
name: nsaf-normalization
description: Use when spectral counts from a Morpheus search of high-resolution accurate-mass
  MS/MS data must be converted into length-normalized, label-free NSAF values, especially
  when peptides are shared across protein families or isoforms/proteoforms.
when_to_use_negative:
- The data were searched with a search engine other than the Morpheus Mass Spectrometry
  Search Engine — MSpC requires Morpheus output.
- Quantification requires MS1 peak alignment and re-extraction or isobaric-tag intensities
  — NSAF is a spectral-count method whose stated advantages are dynamic range and
  speed, not MS1- or reporter-ion-based quantification.
- Peptides are all uniquely mapped and protein-family/isoform resolution is not needed
  — only unique NSAF is informative and the distributed-counting machinery is unnecessary.
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Morpheus Spectral Counter (MSpC)
  role: Performs the NSAF calculation directly from Morpheus output, computing distributed
    and unique NSAF values and writing nsaf_values.csv
  repo: github.com/dcgemperline/Morpheus_SpC
- name: Morpheus
  role: Upstream peptide-MS/MS matching engine whose output file of spectrum-to-protein
    identifications is the required input
  repo: cwenger.github.io/Morpheus
- name: C#
  role: Implementation language of MSpC; the open-source MIT-licensed codebase can
    be inspected or extended
  repo: github.com/dcgemperline/Morpheus_SpC
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201500420
    title: 'Morpheus Spectral Counter: A computational tool for label‐free quantitative
      mass spectrometry using the Morpheus search engine'
  source_content_hash: sha256:12d4de10a0585fd022680a01b217908c8c47b173723dc06e97e98d6ee6730598
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
yes_rate: 0.7857142857142857
n_claims: 14
---

# nsaf-normalization

## Summary

Computes Normalized Spectral Abundance Factor (NSAF) values from Morpheus peptide-MS/MS search output using Morpheus Spectral Counter (MSpC), including both distributed and unique NSAF so protein families and isoforms/proteoforms can be quantified, not just uniquely resolved proteins.

## When to use

Use when spectral counts from a Morpheus search of high-resolution accurate-mass MS/MS data must be converted into length-normalized, label-free NSAF values, especially when peptides are shared across protein families or isoforms/proteoforms.

## When NOT to use

- The data were searched with a search engine other than the Morpheus Mass Spectrometry Search Engine — MSpC requires Morpheus output.
- Quantification requires MS1 peak alignment and re-extraction or isobaric-tag intensities — NSAF is a spectral-count method whose stated advantages are dynamic range and speed, not MS1- or reporter-ion-based quantification.
- Peptides are all uniquely mapped and protein-family/isoform resolution is not needed — only unique NSAF is informative and the distributed-counting machinery is unnecessary.

## Inputs

- Morpheus Mass Spectrometry Search Engine output file containing peptide-MS/MS spectrum matches with protein mappings
- High-resolution accurate-mass MS/MS dataset previously searched with Morpheus

## Outputs

- nsaf_values.csv: per-protein table with spectral counts, distributed NSAF, and unique NSAF columns

## How to apply

Load the Morpheus peptide-MS/MS matching engine output containing spectrum-to-protein identifications and aggregate spectral counts per protein, separating shared peptides from unique peptides based on their protein mappings. Compute distributed NSAF by distributing the spectral counts of shared peptides among the member proteins of each protein family, then apply length normalization by dividing by each protein's sequence length and normalizing by the sum of length-corrected counts across all identified proteins. In parallel, compute unique NSAF using only unique-peptide spectral counts, so isoforms/proteoforms within a family receive non-zero values only from peptides exclusive to them. Emit per-protein rows with spectral counts, distributed NSAF, and unique NSAF columns to nsaf_values.csv. MSpC is the tool that performs this computation directly from Morpheus output; it is developed in C# and requires the data to have been searched with the Morpheus Mass Spectrometry Search Engine.

## Related tools

- **Morpheus Spectral Counter (MSpC)** (Performs the NSAF calculation directly from Morpheus output, computing distributed and unique NSAF values and writing nsaf_values.csv) — github.com/dcgemperline/Morpheus_SpC
- **Morpheus** (Upstream peptide-MS/MS matching engine whose output file of spectrum-to-protein identifications is the required input) — cwenger.github.io/Morpheus
- **C#** (Implementation language of MSpC; the open-source MIT-licensed codebase can be inspected or extended) — github.com/dcgemperline/Morpheus_SpC

## Evaluation signals

- nsaf_values.csv exists and is a well-formed table with distributed-NSAF and unique-NSAF columns present.
- The NSAF columns sum to approximately 1 across all identified proteins.
- There is a row for each protein reported in the Morpheus output.
- For each protein family, the distributed NSAF values sum consistently with the family's total spectral counts.
- Isoforms/proteoforms that share all their peptides show non-zero distributed NSAF but zero unique NSAF.

## Limitations

- Data must have been searched with the Morpheus Mass Spectrometry Search Engine; other search engines' outputs are not supported.
- Quantification is spectral-count based, so it is subject to the dynamic-range and counting-resolution limits of spectral counting relative to MS1- or isobaric-tag-based methods (though NSAF is described as having greater dynamic range than isobaric tags).
- Proteins/isoforms that share all peptides cannot be individually resolved: they receive distributed NSAF but zero unique NSAF.

## Evidence
