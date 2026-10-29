---
name: peptide-sequence-set-intersection
description: Use when you have peptide lists (1% FDR, length 8-25) from several search
  engines on the same MS/MS data and need to quantify the fraction of peptides commonly
  identified by all engines, or how rescoring changed that consensus.
when_to_use_negative:
- The task is to improve per-engine sensitivity via rescoring (e.g., running DeepRescore/Percolator
  itself) rather than to quantify cross-engine consensus after identifications exist
- Input is raw MS/MS spectra (MGF/mzML) that still need database searching — this
  skill operates on already-filtered peptide identification lists
- Engine results are not harmonized to a common FDR definition (1% at both PSM and
  peptide levels), making the sets non-comparable
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Comet
  role: Source of one of the four peptide identification sets being intersected
- name: MS-GF+
  role: Source of one of the four peptide identification sets being intersected
  repo: https://github.com/MSGFPlus/msgfplus
- name: X!Tandem
  role: Source of one of the four peptide identification sets being intersected
- name: MaxQuant
  role: Source of one of the four peptide identification sets being intersected
- name: DeepRescore
  role: Rescores each engine's PSMs with deep learning-derived RTdelta and spectrum
    prediction features; its q-value-filtered outputs define the post-rescoring peptide
    sets
  repo: https://github.com/bzhanglab/DeepRescore
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:f277319e778d6874365b4536498c8967268ae4f09062576369f822d1c08c8ca1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MSGFPlus/msgfplus
  tools:
  - Comet
  - DeepRescore
  - MS-GF+
  - MaxQuant
  - X!Tandem
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# peptide-sequence-set-intersection

## Summary

Quantify the overlap of unique peptide sequences identified by multiple database search engines (Comet, MS-GF+, X!Tandem, MaxQuant) on immunopeptidomics datasets by computing set intersections over the union, before and after DeepRescore rescoring, to measure how much rescoring increases consensus identifications.

## When to use

Use when you have peptide lists (1% FDR, length 8-25) from several search engines on the same MS/MS data and need to quantify the fraction of peptides commonly identified by all engines, or how rescoring changed that consensus.

## When NOT to use

- The task is to improve per-engine sensitivity via rescoring (e.g., running DeepRescore/Percolator itself) rather than to quantify cross-engine consensus after identifications exist
- Input is raw MS/MS spectra (MGF/mzML) that still need database searching — this skill operates on already-filtered peptide identification lists
- Engine results are not harmonized to a common FDR definition (1% at both PSM and peptide levels), making the sets non-comparable

## Inputs

- Per-engine unique peptide sequence lists filtered at 1% FDR (PSM and peptide levels) via PGA target-decoy strategy, peptides of length 8-25
- DeepRescore *_pep_final.tsv percolator outputs with q-value column for post-rescoring intersection
- Dataset identifiers D1 (Mel15 HLA class I, PRIDE PXD004894) and D2 (A*11:01 mono-allelic cell line, MassIVE MSV000084442)

## Outputs

- Per-engine unique peptide counts (e.g., D1: Comet 23400, MS-GF+ 22789, X!Tandem 20727, MaxQuant 17122; D2: 3255, 4935, 4048, 2635)
- Union size of unique peptides across engines (e.g., 31447 for D1)
- Fraction of union peptides identified by all four engines, before and after rescoring (33%→57% D1; 34%→74% D2)
- Fraction of union peptides uniquely identified by a single engine (e.g., 24% pre-rescoring on D1)

## How to apply

First filter peptide identifications from each engine using the target-decoy strategy implemented in PGA, controlling FDR at 1% at both PSM and peptide levels, keeping only peptides of length 8-25. For each dataset (e.g., D1 with 938,131 MS/MS spectra, D2 with 279,700), collect unique peptide sequences per engine and compute the set intersection over the union of all engines. For DeepRescore-rescored results, filter each engine's *_pep_final.tsv output by the 'q-value' column (q-value <= 0.01) before recomputing the intersection. The rationale is that consensus across heterogeneous scoring schemes is a stringent measure of identification reliability; the article reports this consensus fraction rising from 33% to 57% on D1 and from 34% to 74% on D2 after DeepRescore rescoring.

## Related tools

- **Comet** (Source of one of the four peptide identification sets being intersected)
- **MS-GF+** (Source of one of the four peptide identification sets being intersected) — https://github.com/MSGFPlus/msgfplus
- **X!Tandem** (Source of one of the four peptide identification sets being intersected)
- **MaxQuant** (Source of one of the four peptide identification sets being intersected)
- **DeepRescore** (Rescores each engine's PSMs with deep learning-derived RTdelta and spectrum prediction features; its q-value-filtered outputs define the post-rescoring peptide sets) — https://github.com/bzhanglab/DeepRescore

## Examples

```
from functools import reduce; common = reduce(lambda a,b: a & b, [set(d[d['q-value']<=0.01]['peptide']) for d in engine_pep_final_dfs])
```

## Evaluation signals

- Recomputed all-four-engine intersection fraction reproduces the reported pre-rescoring values: ~33% of 31447 unique peptides for D1 and ~34% for D2 within rounding tolerance
- Post-rescoring intersection fractions reproduce the reported increases to 57% (D1) and 74% (D2) after q-value <= 0.01 filtering of *_pep_final.tsv
- Per-engine unique peptide counts match reported values (D1: Comet 23400, MS-GF+ 22789, X!Tandem 20727, MaxQuant 17122; D2: 3255, 4935, 4048, 2635)
- Sanity invariant: intersection size <= each per-engine count and <= union size; single-engine-unique fraction reported (e.g., 24% on D1 pre-rescoring) is consistent with the Venn decomposition

## Limitations

- Because immunopeptidomics experiments do not use enzymatic digestion, the large search space yields high false positive rates and low sensitivity, so unrescored overlap fractions understate the true consensus
- Results depend on the exact filtering scheme (1% FDR at both PSM and peptide levels, peptide length 8-25); peptides passing FDR for 'at least three of four engines' is a different high-quality definition used for AutoRT training and should not be mixed with the all-four intersection
- Post-rescoring filtering relies on DeepRescore's q-value column convention (q-value <= 0.01 in *_pep_final.tsv)

## Evidence
