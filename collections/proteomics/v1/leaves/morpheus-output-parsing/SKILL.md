---
name: morpheus-output-parsing
description: Use when input data consist of Morpheus search-engine output (peptide-spectrum
  matches with protein mappings) from high-resolution accurate-mass instruments and
  the goal is spectral-count-based label-free quantification (NSAF).
when_to_use_negative:
- Data were searched with a search engine other than Morpheus — the README states
  'To use this software your data must be searched with the Morpheus Mass Spectrometry
  Search Engine.'
- The goal is quantification via isobaric tags or MS1 peak extraction rather than
  spectral counting — NSAF's stated advantages (greater dynamic range than isobaric
  tags, no MS1 realignment, increased speed) apply only to spectral-count LFQ
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Morpheus Spectral Counter (MSpC)
  role: Application that directly calculates NSAF values (distributed and unique)
    from Morpheus output and writes nsaf_values.csv
  repo: github.com/dcgemperline/Morpheus_SpC
- name: Morpheus
  role: Upstream peptide-MS/MS matching engine whose output file provides the spectrum-to-protein
    identifications parsed by MSpC
  repo: cwenger.github.io/Morpheus
- name: C#
  role: Implementation language of MSpC, open sourced under an MIT License
  repo: github.com/dcgemperline/Morpheus_SpC
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201500420
    title: 'Morpheus Spectral Counter: A computational tool for label‐free quantitative
      mass spectrometry using the Morpheus search engine'
  source_content_hash: sha256:5f8e875b8387313c78fee96b2135de073f62a0c580f733c1984d282bc7c8956c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C#
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
yes_rate: 0.8
n_claims: 10
---

# morpheus-output-parsing

## Summary

Parse output files from the Morpheus peptide-MS/MS matching engine and feed them into MSpC to compute normalized spectral abundance factor (NSAF) values for label-free quantitative proteomics, including distributed and unique NSAF for protein families and isoforms/proteoforms.

## When to use

Use when input data consist of Morpheus search-engine output (peptide-spectrum matches with protein mappings) from high-resolution accurate-mass instruments and the goal is spectral-count-based label-free quantification (NSAF).

## When NOT to use

- Data were searched with a search engine other than Morpheus — the README states 'To use this software your data must be searched with the Morpheus Mass Spectrometry Search Engine.'
- The goal is quantification via isobaric tags or MS1 peak extraction rather than spectral counting — NSAF's stated advantages (greater dynamic range than isobaric tags, no MS1 realignment, increased speed) apply only to spectral-count LFQ

## Inputs

- Morpheus search-engine output file with peptide-spectrum matches and protein mappings
- High-resolution accurate-mass MS/MS datasets previously searched with the Morpheus Mass Spectrometry Search Engine

## Outputs

- nsaf_values.csv — per-protein table with spectral counts, distributed NSAF, and unique NSAF columns

## How to apply

Load the Morpheus output file containing spectrum-to-protein identifications with pandas (or the C# MSpC application). Aggregate spectral counts per protein, separating shared peptides (distributed among protein-family members) from unique peptides. Apply length normalization by dividing each protein's spectral count by its sequence length, then normalize by the sum across all identified proteins to yield distributed NSAF and unique NSAF per protein. Write the per-protein table (spectral counts, distributed NSAF, unique NSAF) to nsaf_values.csv. Rationale: MSpC is the first tool to directly calculate NSAF values from Morpheus output, and the dual distributed/unique NSAF values permit analyses of both protein families and isoforms/proteoforms.

## Related tools

- **Morpheus Spectral Counter (MSpC)** (Application that directly calculates NSAF values (distributed and unique) from Morpheus output and writes nsaf_values.csv) — github.com/dcgemperline/Morpheus_SpC
- **Morpheus** (Upstream peptide-MS/MS matching engine whose output file provides the spectrum-to-protein identifications parsed by MSpC) — cwenger.github.io/Morpheus
- **C#** (Implementation language of MSpC, open sourced under an MIT License) — github.com/dcgemperline/Morpheus_SpC

## Examples

```
import pandas as pd; morpheus_psm = pd.read_csv('morpheus_output.csv'); # aggregate spectral counts per protein, split shared/unique peptides, compute distributed and unique NSAF, write nsaf_values.csv
```

## Evaluation signals

- nsaf_values.csv exists and parses as a well-formed table with one row per protein reported in the Morpheus output
- Distributed-NSAF and unique-NSAF columns are present (field_present checks) and NSAF values sum to approximately 1 across identified proteins
- For each protein family, distributed NSAF values sum consistently with the family's total spectral counts
- Isoforms/proteoforms that share all peptides show distributed but zero unique NSAF
- Quantification is linear: MSpC protein-concentration determinations should be linear over several orders of magnitude (as shown on datasets from PRIDE or total cell extracts spiked with purified Arabidopsis 20S proteasomes)

## Limitations

- Only works on data searched with the Morpheus Mass Spectrometry Search Engine; other search engines' output is not supported
- Shared-peptide handling means isoforms/proteoforms sharing all peptides receive zero unique NSAF and must be compared via distributed NSAF
- No changelog is available for the repository, so version-specific behavior of the output format cannot be traced
- Validation in the source was based on a limited set of high-mass-accuracy datasets (PRIDE or cell extracts spiked with Arabidopsis 20S proteasomes)

## Evidence

- [abstract] MSpC directly calculates NSAF values from Morpheus output: "we present Morpheus Spectral Counter (MSpC) as the first computational tool that directly calculates NSAF values from output obtained from Morpheus"
- [abstract] distributed and unique NSAF for families and isoforms: "additionally calculates both distributed and unique NSAF values to permit analyses of both protein families and isoforms/proteoforms"
- [readme] Morpheus requirement from README: "To use this software your data must be searched with the Morpheus Mass Spectrometry Search Engine."
