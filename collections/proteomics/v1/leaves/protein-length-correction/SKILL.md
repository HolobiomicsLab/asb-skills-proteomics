---
name: protein-length-correction
description: Use when converting spectral counts from a Morpheus peptide-MS/MS matching
  engine output into per-protein NSAF values, i.e., when raw spectral counts must
  be corrected for protein sequence length and normalized across all identified proteins.
when_to_use_negative:
- The data were not searched with the Morpheus Mass Spectrometry Search Engine — MSpC
  requires Morpheus output as its input.
- The analysis uses isobaric-tag or MS1 peak-based quantification rather than spectral
  counting; NSAF was chosen for its greater dynamic range versus isobaric tags and
  to avoid MS1 alignment/re-extraction, but it cannot replace those methods' inputs.
- Only raw spectral counts are needed without length correction or cross-protein normalization
  — the NSAF pipeline would add unnecessary transformation.
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Morpheus Spectral Counter (MSpC)
  role: Computes NSAF values directly from Morpheus output, performing shared/unique
    spectral count aggregation and length normalization; writes nsaf_values.csv
  repo: http://dcgemperline.github.io/Morpheus_SpC/
- name: Morpheus
  role: Upstream open-source peptide-MS/MS matching engine whose output file supplies
    the spectrum-to-protein identifications used for NSAF calculation
  repo: http://cwenger.github.io/Morpheus/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201500420
    title: 'Morpheus Spectral Counter: A computational tool for label‐free quantitative
      mass spectrometry using the Morpheus search engine'
  source_content_hash: sha256:7ab39fa28c7946f93fb70c5a0441eab1379d138a31ea42a86479553827afd250
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Morpheus
  - Morpheus Spectral Counter (MSpC)
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# protein-length-correction

## Summary

Length-normalizes Morpheus-derived spectral counts into NSAF (Normalized Spectral Abundance Factor) values, computing both distributed and unique NSAF per protein as implemented in Morpheus Spectral Counter (MSpC) for label-free quantitative proteomics.

## When to use

Use when converting spectral counts from a Morpheus peptide-MS/MS matching engine output into per-protein NSAF values, i.e., when raw spectral counts must be corrected for protein sequence length and normalized across all identified proteins.

## When NOT to use

- The data were not searched with the Morpheus Mass Spectrometry Search Engine — MSpC requires Morpheus output as its input.
- The analysis uses isobaric-tag or MS1 peak-based quantification rather than spectral counting; NSAF was chosen for its greater dynamic range versus isobaric tags and to avoid MS1 alignment/re-extraction, but it cannot replace those methods' inputs.
- Only raw spectral counts are needed without length correction or cross-protein normalization — the NSAF pipeline would add unnecessary transformation.

## Inputs

- Morpheus Mass Spectrometry Search Engine output file with spectrum-to-protein identifications from a high-resolution accurate-mass dataset
- Protein sequence lengths (for the length normalization step)

## Outputs

- nsaf_values.csv containing per-protein spectral counts, distributed NSAF, and unique NSAF

## How to apply

Load the Morpheus output file containing spectrum-to-protein identifications from the searched high-resolution accurate-mass dataset. Aggregate spectral counts per protein, separating shared peptides (for distributed counting) from unique peptides (for unique counting). Apply length normalization by dividing each protein's spectral count by its sequence length, following the NSAF formulation. Normalize the length-corrected counts by their sum across all identified proteins to yield distributed NSAF and unique NSAF values per protein, then write the per-protein table to nsaf_values.csv. The dual distributed/unique computation exists so that protein families and isoforms/proteoforms can be analyzed distinctly.

## Related tools

- **Morpheus Spectral Counter (MSpC)** (Computes NSAF values directly from Morpheus output, performing shared/unique spectral count aggregation and length normalization; writes nsaf_values.csv) — http://dcgemperline.github.io/Morpheus_SpC/
- **Morpheus** (Upstream open-source peptide-MS/MS matching engine whose output file supplies the spectrum-to-protein identifications used for NSAF calculation) — http://cwenger.github.io/Morpheus/

## Evaluation signals

- nsaf_values.csv exists and parses as a well-formed table with one row per protein reported in the Morpheus output
- Distributed NSAF column values sum to approximately 1 across all identified proteins (NSAF is a normalized quantity)
- Unique NSAF column values also sum to approximately 1 when unique-peptide counting is used
- Each protein's NSAF is proportional to its spectral count divided by its sequence length, so short proteins with equal counts get higher NSAF than long proteins
- Proteins sharing peptides receive nonzero distributed NSAF while their unique NSAF reflects only unique-peptide counts, distinguishing isoforms/proteoforms

## Limitations

- Requires input to have been searched with the Morpheus Mass Spectrometry Search Engine; other search engines' outputs are not supported
- Spectral count quantification depends on shared-versus-unique peptide assignment; protein families and isoforms/proteoforms require the distributed NSAF interpretation
- The MSpC repository provides no changelog, limiting traceability of method or implementation changes

## Evidence
