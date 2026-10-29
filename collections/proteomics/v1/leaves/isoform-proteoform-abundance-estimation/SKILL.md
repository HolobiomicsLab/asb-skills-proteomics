---
name: isoform-proteoform-abundance-estimation
description: Use when peptide-spectrum matches from Morpheus contain peptides shared
  across multiple proteins, and abundance estimates are needed for protein families
  or isoforms/proteoforms rather than only individually resolved proteins.
when_to_use_negative:
- The data were not searched with the Morpheus Mass Spectrometry Search Engine — MSpC
  requires Morpheus output and cannot parse other engines' results
- The analysis targets individually resolved proteins only and shared-peptide ambiguity
  is irrelevant, in which case a single plain NSAF computation suffices
- Quantification requires MS1 peak-based (feature-intensity) methods rather than spectral
  counting
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Morpheus Spectral Counter (MSpC)
  role: C# application that directly calculates NSAF values, including distributed
    and unique NSAF, from Morpheus output
  repo: github.com/dcgemperline/Morpheus_SpC
- name: Morpheus
  role: Search engine whose peptide-MS/MS matching output provides the PSMs and protein
    mappings consumed by MSpC
  repo: cwenger.github.io/Morpheus
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201500420
    title: 'Morpheus Spectral Counter: A computational tool for label‐free quantitative
      mass spectrometry using the Morpheus search engine'
  source_content_hash: sha256:3f1de5a4599833ec76f2528c6e5a044a7196016ffbaa6f74ea973e5f6981002c
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
yes_rate: 0.8
n_claims: 10
---

# isoform-proteoform-abundance-estimation

## Summary

Estimates label-free protein abundance for protein families and isoforms/proteoforms by computing distributed NSAF (spectral counts of shared peptides divided among family members) and unique NSAF (counts from family-member-exclusive peptides) directly from Morpheus search-engine output using MSpC.

## When to use

Use when peptide-spectrum matches from Morpheus contain peptides shared across multiple proteins, and abundance estimates are needed for protein families or isoforms/proteoforms rather than only individually resolved proteins.

## When NOT to use

- The data were not searched with the Morpheus Mass Spectrometry Search Engine — MSpC requires Morpheus output and cannot parse other engines' results
- The analysis targets individually resolved proteins only and shared-peptide ambiguity is irrelevant, in which case a single plain NSAF computation suffices
- Quantification requires MS1 peak-based (feature-intensity) methods rather than spectral counting

## Inputs

- Morpheus search-engine output (peptide-spectrum matches with peptide-to-protein mappings)
- High-mass-accuracy datasets analyzed with the Morpheus Mass Spectrometry Search Engine

## Outputs

- Per-protein table with distributed NSAF and unique NSAF columns (nsaf_values.csv)

## How to apply

Load the Morpheus output (PSMs with peptide-to-protein mappings) and group peptides into protein families based on shared peptide-to-protein mappings, distinguishing shared from unique peptides. Compute distributed NSAF by splitting the spectral counts of shared peptides equally among the member proteins of each family, then normalizing by protein length and total spectra. Compute unique NSAF using only unique-peptide spectral counts, so that isoforms/proteoforms within a family receive non-zero values only from peptides exclusive to them. Emit per-protein rows with both distributed-NSAF and unique-NSAF columns. This dual-output design is what permits analyses of both protein families and isoforms/proteoforms.

## Related tools

- **Morpheus Spectral Counter (MSpC)** (C# application that directly calculates NSAF values, including distributed and unique NSAF, from Morpheus output) — github.com/dcgemperline/Morpheus_SpC
- **Morpheus** (Search engine whose peptide-MS/MS matching output provides the PSMs and protein mappings consumed by MSpC) — cwenger.github.io/Morpheus

## Evaluation signals

- For each protein family, the distributed NSAF values sum consistently with the family's total spectral counts
- Isoforms/proteoforms sharing all peptides show non-zero distributed NSAF but zero unique NSAF
- nsaf_values.csv exists and contains both the distributed-NSAF and unique-NSAF columns (field_present checks)
- Protein-concentration determinations should be linear over several orders of magnitude, e.g. on datasets spiked with purified Arabidopsis 20S proteasomes

## Limitations

- Shared peptides carry no isoform/proteoform specificity: members of a family that share all peptides are indistinguishable by unique NSAF and receive zero unique values
- NSAF-based quantification is spectral counting, which has lower quantitative resolution than intensity- or isobaric-tag-based methods, though it offers greater dynamic range than isobaric tags
- Requires data searched with the Morpheus engine; MSpC cannot be applied to outputs of other search engines

## Evidence

- [abstract] additionally calculates both distributed and unique NSAF values to permit analyses of both protein families and isoforms/proteoforms: "additionally calculates both distributed and unique NSAF values to permit analyses of both protein families and isoforms/proteoforms"
- [abstract] first computational tool that directly calculates NSAF values from output obtained from Morpheus: "we present Morpheus Spectral Counter (MSpC) as the first computational tool that directly calculates NSAF values from output obtained from Morpheus"
- [intro] MSpC requires Morpheus-searched data: "To use this software your data must be searched with the Morpheus Mass Spectrometry Search Engine."
- [readme] README tool description: "This is a an application for Spectral Count Based Label-free Quantitative (LFQ) proteomics that integrates with the Morpheus Mass Spectrometry Search Engine."
