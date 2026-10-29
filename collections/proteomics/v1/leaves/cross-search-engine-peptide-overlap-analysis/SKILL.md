---
name: cross-search-engine-peptide-overlap-analysis
description: Use when multiple search engines (e.g., Comet, MS-GF+, X!Tandem, MaxQuant)
  have been run on the same immunopeptidomics datasets and you need to quantify peptide-level
  consensus versus single-engine-unique identifications, especially before and after
  Percolator-style rescoring.
when_to_use_negative:
- Only a single search engine's results are available, so no cross-engine intersection
  can be computed
- Peptide identifications have not been FDR-filtered (e.g., raw search-engine output
  without 1% target-decoy filtering via PGA), as unfiltered overlap fractions are
  not comparable to the reported values
- The analysis targets MHC binding affinity or neoantigen prioritization rather than
  search-engine consensus; use binding-prediction workflows (percentile rank <= 2%)
  instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Comet
  role: One of the four search engines whose peptide identifications contribute to
    the overlap computation
- name: MS-GF+
  role: One of the four search engines whose peptide identifications contribute to
    the overlap computation
  repo: https://github.com/MSGFPlus/msgfplus
- name: X!Tandem
  role: One of the four search engines whose peptide identifications contribute to
    the overlap computation
- name: MaxQuant
  role: One of the four search engines whose peptide identifications contribute to
    the overlap computation
- name: DeepRescore
  role: Rescoring tool whose *_pep_final.tsv outputs (filtered at q-value <= 0.01)
    supply the post-rescoring peptide sets for overlap comparison
  repo: https://github.com/bzhanglab/DeepRescore
- name: PGA
  role: Provides the target-decoy strategy used to control FDR at 1% at PSM and peptide
    levels before the overlap analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:7565e9b888d63621f54aeb827f07d2d9fc7749993a77045728e28d4e8efdca15
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MSGFPlus/msgfplus
  tools:
  - Comet
  - DeepRescore
  - MS-GF+
  - MaxQuant
  - PGA
  - X!Tandem
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cross-search-engine-peptide-overlap-analysis

## Summary

Quantifies the proportion of peptide identifications commonly found by all four search engines (Comet, MS-GF+, X!Tandem, MaxQuant) via set-intersection analysis of unique peptide sequences, and measures how DeepRescore rescoring changes this overlap. It exposes the agreement between search engines before and after deep-learning-assisted rescoring.

## When to use

Use when multiple search engines (e.g., Comet, MS-GF+, X!Tandem, MaxQuant) have been run on the same immunopeptidomics datasets and you need to quantify peptide-level consensus versus single-engine-unique identifications, especially before and after Percolator-style rescoring.

## When NOT to use

- Only a single search engine's results are available, so no cross-engine intersection can be computed
- Peptide identifications have not been FDR-filtered (e.g., raw search-engine output without 1% target-decoy filtering via PGA), as unfiltered overlap fractions are not comparable to the reported values
- The analysis targets MHC binding affinity or neoantigen prioritization rather than search-engine consensus; use binding-prediction workflows (percentile rank <= 2%) instead

## Inputs

- Peptide identification results from four search engines (Comet, MS-GF+, X!Tandem, MaxQuant) on immunopeptidomics MS/MS data (MGF-converted raw files) filtered at 1% FDR (PSM and peptide levels) with peptide length 8-25
- DeepRescore-rescored outputs (*_pep_final.tsv) filtered by q-value <= 0.01

## Outputs

- Per-engine unique peptide counts per dataset (e.g., D1: Comet 23400, MS-GF+ 22789, X!Tandem 20727, MaxQuant 17122)
- Union size of unique peptides across all four engines (e.g., 31447 for D1)
- Fraction of peptides commonly identified by all four engines before and after rescoring (e.g., 33% -> 57% for D1; 34% -> 74% for D2)
- Fraction of peptides uniquely identified by only one engine (e.g., 24% pre-rescoring for D1)

## How to apply

First, filter peptide identifications from each search engine using the target-decoy strategy implemented in PGA, controlling FDR at 1% at both PSM and peptide levels, and keep only peptides of length 8-25. For the pre-rescoring analysis, compute the set intersection of unique peptide sequences across the four engines per dataset (D1: Mel15 HLA class I, 938,131 spectra; D2: A*11:01 mono-allelic cell line, 279,700 spectra) and express it as a fraction of the union. Repeat the computation on DeepRescore-rescored results, filtering by q-value <= 0.01 from the *_pep_final.tsv outputs. Tabulate per-engine unique peptide counts (e.g., for D1: Comet 23400, MS-GF+ 22789, X!Tandem 20727, MaxQuant 17122), union sizes, all-four-engine overlap fractions, and single-engine-unique fractions. The rationale is that immunopeptidomics has a large no-enzyme search space that inflates false positives, so cross-engine consensus is a conservative proxy for identification reliability, and rescoring should increase consensus.

## Related tools

- **Comet** (One of the four search engines whose peptide identifications contribute to the overlap computation)
- **MS-GF+** (One of the four search engines whose peptide identifications contribute to the overlap computation) — https://github.com/MSGFPlus/msgfplus
- **X!Tandem** (One of the four search engines whose peptide identifications contribute to the overlap computation)
- **MaxQuant** (One of the four search engines whose peptide identifications contribute to the overlap computation)
- **DeepRescore** (Rescoring tool whose *_pep_final.tsv outputs (filtered at q-value <= 0.01) supply the post-rescoring peptide sets for overlap comparison) — https://github.com/bzhanglab/DeepRescore
- **PGA** (Provides the target-decoy strategy used to control FDR at 1% at PSM and peptide levels before the overlap analysis)

## Evaluation signals

- Recomputed pre-rescoring all-four-engine overlap fractions reproduce the reported values (33% for D1, 34% for D2) within rounding tolerance
- Post-rescoring overlap fractions show the reported increase (to 57% for D1 and 74% for D2) after DeepRescore rescoring
- Per-engine unique peptide counts match the reported values (D1: Comet 23400, MS-GF+ 22789, X!Tandem 20727, MaxQuant 17122; D2: Comet 3255, MS-GF+ 4935, X!Tandem 4048, MaxQuant 2635)
- D1 union size equals 31447 unique peptides when computed from FDR-filtered, length-8-25 peptide sets
- Filtering consistency check: every peptide in the sets passes 1% FDR (PSM and peptide levels via PGA) pre-rescoring, or q-value <= 0.01 in the *_pep_final.tsv DeepRescore outputs

## Limitations

- Pre- and post-rescoring overlap fractions are only comparable if the same FDR thresholds (1% at PSM and peptide levels pre-rescoring; q-value <= 0.01 post-rescoring) and peptide length filter (8-25) are applied consistently
- Overlap fractions depend on the search space: because immunopeptidomics experiments do not use enzymatic digestion, the large database search space inflates false positives and depresses cross-engine consensus, so absolute fractions are dataset-specific
- The all-four-engine intersection is a strict criterion; a large fraction of identifications are single-engine-unique (24% pre-rescoring in D1), so consensus underestimates the total confidently identified set

## Evidence

- [results] Together, the four search engines identified a total of 31447 unique peptides. Among these peptides, only 33% were commonly identified by all four search engines: "Together, the four search engines identified a total of 31447 unique peptides. Among these peptides, only 33% were commonly identified by all four search engines"
- [methods] FDR controlled at 1% at PSM and peptide levels: "FDR was controlled at 1% at both PSM and peptide levels"
- [methods] q-value <= 0.01 filtering of DeepRescore outputs: "Users can filter the result based on the column `q-value` (for example, q-value <= 0.01)"
- [readme] DeepRescore supports the four search engines: "The current version supports four search engines, [MS-GF+](https://github.com/MSGFPlus/msgfplus), [Comet](http://comet-ms.sourceforge.net/), [X!Tandem](https://www.thegpm.org/TANDEM/), and"
- [methods] Only peptides with length between 8 and 25 were considered: "Only peptides with length between 8 and 25 were considered"
