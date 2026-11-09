---
name: spectral-counting-quantification
description: Use when quantifying histone PTM peptide groups label-free from Mascot
  database-search results, or when benchmarking spectral counting against MS/MS ion-intensity
  (isoScale) and XIC quantification across technical replicates.
when_to_use_negative:
- When co-fragmenting isobaric peptides sharing the same sequence but distinct PTM
  localizations must be resolved — spectral counting cannot separate them; use isoScale
  with FIRR calculation instead
- When the input is already a normalized quantification table (e.g. isoScale or XIC
  output) rather than raw Mascot PSM lists
- For quantifying unmodified or extensively modified histone peptides, where spectral
  counting was shown to be inaccurate against the XIC reference
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Mascot
  role: Database searching engine whose .csv output supplies the peptide-spectrum
    matches counted for spectral-counting quantification
- name: Histone Coder
  role: Validates PTM localization by counting site-determining MS/MS ions and converts
    Mascot output to Brno nomenclature before quantification
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: isoScale
  role: Comparator quantification method based on total MS/MS ion intensity with FIRR
    for co-fragmenting isobaric species
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Java
  role: Runtime requirement to launch Histone Coder and isoScale jar tools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:7c190cb4f46ad6be34eb8ef889973db35e6fca51df7db35623742da45e4b20f7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Java
  - Mascot
  - isoScale
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# spectral-counting-quantification

## Summary

Label-free quantification of histone PTM peptide groups by counting Mascot-validated peptide-spectrum matches, used in middle-down proteomics as a comparator against isoScale MS/MS ion-intensity quantification and XIC reference quantification. In the benchmark on Suz12-/- ESC histone H3, spectral counting was inaccurate for unmodified and extensively modified peptides.

## When to use

Use when quantifying histone PTM peptide groups label-free from Mascot database-search results, or when benchmarking spectral counting against MS/MS ion-intensity (isoScale) and XIC quantification across technical replicates.

## When NOT to use

- When co-fragmenting isobaric peptides sharing the same sequence but distinct PTM localizations must be resolved — spectral counting cannot separate them; use isoScale with FIRR calculation instead
- When the input is already a normalized quantification table (e.g. isoScale or XIC output) rather than raw Mascot PSM lists
- For quantifying unmodified or extensively modified histone peptides, where spectral counting was shown to be inaccurate against the XIC reference

## Inputs

- Mascot database search output in .csv format (peptide-spectrum matches)
- Histone Coder result file with validated, PTM-localized spectra in Brno nomenclature
- Extracted ion chromatogram (XIC) quantification per peptide group across three technical replicates
- Replicate assignment for Suz12-/- ESC purified histone H3

## Outputs

- Spectral-counting relative abundances per peptide group per replicate (normalized)
- Comparison table fig3_quantification_comparison.csv recording isoScale vs XIC vs spectral-counting agreement for the 22 peptide groups
- Per-group agreement status (agreement/no-agreement) against the XIC reference

## How to apply

Parse the Mascot database-search output (.csv) together with the Histone Coder result file to obtain validated, PTM-localized peptide-spectrum matches in Brno nomenclature (e.g. K4me3K14ac). For each of the 22 peptide groups (0-3 acetyl, 0-9 methyl PTM equivalents) in purified histone H3, count spectra per group in each of the three technical replicates, then normalize relative abundances per replicate. Run the identical groups in parallel through isoScale (ETD fragmentation selected, MS/MS ion tolerance chosen for fragment-ion searching) and through extracted ion chromatograms (XIC) to establish a reference. Accept the comparison only if isoScale agrees with the XIC reference on at least 18 of the 22 groups, and record per-group agreement; expect spectral counting to fail for unmodified and heavily modified peptides because co-isolated/co-fragmented isobaric species are not resolved by counting PSMs.

## Related tools

- **Mascot** (Database searching engine whose .csv output supplies the peptide-spectrum matches counted for spectral-counting quantification)
- **Histone Coder** (Validates PTM localization by counting site-determining MS/MS ions and converts Mascot output to Brno nomenclature before quantification) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **isoScale** (Comparator quantification method based on total MS/MS ion intensity with FIRR for co-fragmenting isobaric species) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Java** (Runtime requirement to launch Histone Coder and isoScale jar tools)

## Evaluation signals

- isoScale quantification agrees with the XIC reference quantification on at least 18 of the 22 peptide groups; spectral-counting agreement falls below this threshold for unmodified and extensively modified peptides
- Relative abundances are normalized per replicate before cross-method comparison
- All 22 peptide groups (0-3 acetyl, 0-9 methyl PTM equivalents) are quantified consistently across the three technical replicates
- Comparison output written as a per-group agreement table (fig3_quantification_comparison.csv)
- PTM-localized PSMs carry Histone Coder-validated localization counts and Brno nomenclature labels (e.g. K4me3K14ac)

## Limitations

- Spectral counting is inaccurate for unmodified and extensively modified peptides relative to the XIC reference
- Counting PSMs cannot handle relative quantification of isobaric peptides co-fragmented in MS/MS spectra; isoScale's FIRR approach is required for those species
- Software tools are GUI jars requiring Java; isoScale expects a Histone Coder result file plus a Mascot .csv data file as inputs

## Evidence
