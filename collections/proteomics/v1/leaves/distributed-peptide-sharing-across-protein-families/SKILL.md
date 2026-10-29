---
name: distributed-peptide-sharing-across-protein-families
description: Use when Morpheus search-engine output maps peptides to multiple proteins
  within a protein family or isoform/proteoform group, and label-free quantification
  must attribute shared spectra across those members rather than discarding them.
when_to_use_negative:
- The data were not searched with the Morpheus Mass Spectrometry Search Engine — MSpC
  requires Morpheus output as its input.
- The analysis uses isobaric-tag or MS1 peak-intensity quantification rather than
  spectral counting; NSAF offers advantages over isobaric tags but this skill only
  computes spectral-count-based values.
- The goal is only individually resolved proteins with no shared peptides — the distributed
  counting step is unnecessary when all peptides are unique.
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Morpheus Spectral Counter (MSpC)
  role: C# application that groups shared peptides into protein families, computes
    distributed and unique NSAF values from Morpheus output, and writes nsaf_values.csv
  repo: http://dcgemperline.github.io/Morpheus_SpC
- name: Morpheus
  role: Upstream peptide-MS/MS matching engine whose output files supply the spectrum-to-protein
    identifications used for shared/unique peptide classification
  repo: http://cwenger.github.io/Morpheus/
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201500420
    title: 'Morpheus Spectral Counter: A computational tool for label‐free quantitative
      mass spectrometry using the Morpheus search engine'
  source_content_hash: sha256:caaa1e96eb43bcbea731d9da387124d8d9f490c0a7d42470c874cf8178442aa2
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
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# distributed-peptide-sharing-across-protein-families

## Summary

A spectral-count quantification approach implemented in Morpheus Spectral Counter (MSpC) that distributes spectra of peptides shared within a protein family among its member proteins to yield distributed NSAF values, alongside unique NSAF values computed solely from unique peptides.

## When to use

Use when Morpheus search-engine output maps peptides to multiple proteins within a protein family or isoform/proteoform group, and label-free quantification must attribute shared spectra across those members rather than discarding them.

## When NOT to use

- The data were not searched with the Morpheus Mass Spectrometry Search Engine — MSpC requires Morpheus output as its input.
- The analysis uses isobaric-tag or MS1 peak-intensity quantification rather than spectral counting; NSAF offers advantages over isobaric tags but this skill only computes spectral-count-based values.
- The goal is only individually resolved proteins with no shared peptides — the distributed counting step is unnecessary when all peptides are unique.

## Inputs

- Morpheus search-engine output file containing peptide-spectrum matches with peptide-to-protein mappings
- Protein sequence lengths (from the searched FASTA) for length normalization

## Outputs

- nsaf_values.csv with per-protein rows of spectral counts, distributed NSAF, and unique NSAF columns
- Per-protein-family assignment of shared vs. unique peptides

## How to apply

Load the Morpheus peptide-MS/MS matching output (peptide-spectrum matches with protein mappings) and group peptides into protein families based on shared peptide-to-protein mappings, classifying each peptide as shared or unique. For distributed NSAF, distribute the spectral counts of shared peptides equally among the member proteins of each family, then apply the NSAF formulation by dividing each protein's spectral count by its sequence length and normalizing the length-corrected counts by their sum across all identified proteins. For unique NSAF, use only unique-peptide spectral counts through the same length-normalization and normalization steps, so isoforms/proteoforms within a family receive non-zero values only from peptides exclusive to them. Emit per-protein rows containing spectral counts, distributed NSAF, and unique NSAF columns to nsaf_values.csv. The rationale is that distributed NSAF permits analyses of protein families while unique NSAF preserves isoform/proteoform resolution within a family.

## Related tools

- **Morpheus Spectral Counter (MSpC)** (C# application that groups shared peptides into protein families, computes distributed and unique NSAF values from Morpheus output, and writes nsaf_values.csv) — http://dcgemperline.github.io/Morpheus_SpC
- **Morpheus** (Upstream peptide-MS/MS matching engine whose output files supply the spectrum-to-protein identifications used for shared/unique peptide classification) — http://cwenger.github.io/Morpheus/

## Evaluation signals

- nsaf_values.csv exists and parses as a well-formed table with distributed-NSAF and unique-NSAF columns present
- For each protein, the length-corrected counts normalized by their sum across identified proteins yield NSAF columns that sum to approximately 1 across identified proteins
- For each protein family, the distributed NSAF values sum consistently with the family's total spectral counts
- Isoforms/proteoforms that share all peptides show non-zero distributed NSAF but zero unique NSAF
- Every protein reported in the Morpheus output has a corresponding row in nsaf_values.csv

## Limitations

- Data must be searched with the Morpheus Mass Spectrometry Search Engine; other search engines' output is not supported by MSpC.
- Isoforms/proteoforms within a family that share all their peptides cannot be distinguished by unique NSAF — they receive zero unique NSAF and are only resolvable via distributed counting.
- Spectral-count-based NSAF is subject to the general dynamic-range and accuracy limits of spectral counting; linearity was demonstrated over several orders of magnitude but not unlimited range.

## Evidence

- [abstract] MSpC directly calculates NSAF (Normalized Spectral Abundance Factor) values from output files produced by the Morpheus peptide-MS/MS matching engine: "we present Morpheus Spectral Counter (MSpC) as the first computational tool that directly calculates NSAF values from output obtained from Morpheus"
- [abstract] Distributed and unique NSAF handling of shared peptides: "additionally calculates both distributed and unique NSAF values to permit analyses of both protein families and isoforms/proteoforms"
- [methods] Shared peptides distributed among family members; unique NSAF from unique peptides only: [evidence span withheld — anchor status: not_found]
- [readme] MSpC integrates with Morpheus and is developed in C#: "This is a an application for Spectral Count Based Label-free Quantitative (LFQ) proteomics that integrates with the Morpheus Mass Spectrometry Search Engine."
