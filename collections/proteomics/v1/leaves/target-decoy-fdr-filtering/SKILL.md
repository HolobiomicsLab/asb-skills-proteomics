---
name: target-decoy-fdr-filtering
description: Use when peptide identifications from search engines (Comet, MS-GF+,
  X!Tandem, MaxQuant) must be filtered to 1% FDR at PSM and peptide levels before
  overlap, ablation, or neoantigen analyses.
when_to_use_negative:
- Input is already filtered, e.g. DeepRescore *_pep_final.tsv rows where q-value has
  already been thresholded — re-filtering would double-truncate the set
- Analyzing enzymatically digested tryptic proteomics data where the search space
  is small; this skill targets the inflated search space of immunopeptidomics (unspecific
  cleavage) searches
- Defining MHC binders from predicted binding affinity, where the correct cutoff is
  a percentile rank score <= 2%, not FDR filtering
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: PGA
  role: Implements the target-decoy strategy used to control FDR at 1% at both PSM
    and peptide levels
- name: Comet
  role: Search engine whose Evalue-scored target/decoy PSMs are filtered at 1% FDR
- name: MS-GF+
  role: Search engine whose Evalue-scored target/decoy PSMs are filtered at 1% FDR
  repo: https://github.com/MSGFPlus/msgfplus
- name: X!Tandem
  role: Search engine whose Evalue-scored target/decoy PSMs are filtered at 1% FDR
- name: MaxQuant
  role: Search engine whose PEP-scored target/decoy PSMs are filtered at 1% FDR
- name: DeepRescore
  role: Produces rescored PSM/peptide tables with a q-value column used for q-value
    <= 0.01 filtering
  repo: https://github.com/bzhanglab/DeepRescore
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_005
  - task_007
  - task_008
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:6ea576fa3898960faacc5a88c7242179fa589e45990ef7342f8b839390cfda3e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MSGFPlus/msgfplus
  tools:
  - ANNOVAR
  - AutoRT
  - Comet
  - Customprodbj
  - DeepRescore
  - Docker
  - HLAthena
  - MHCflurry
  - MS-GF+
  - MaxQuant
  - NeoFlow
  - NetMHCpan
  - NextFlow
  - PDV
  - PGA
  - ProteoWizard
  - X!Tandem
  - pDeep2
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# target-decoy-fdr-filtering

## Summary

Filter peptide-spectrum matches and peptide identifications using the target-decoy strategy (e.g., implemented in PGA) so that FDR is controlled at 1% at both PSM and peptide levels, with peptides restricted to length 8-25 for immunopeptidomics searches.

## When to use

Use when peptide identifications from search engines (Comet, MS-GF+, X!Tandem, MaxQuant) must be filtered to 1% FDR at PSM and peptide levels before overlap, ablation, or neoantigen analyses.

## When NOT to use

- Input is already filtered, e.g. DeepRescore *_pep_final.tsv rows where q-value has already been thresholded — re-filtering would double-truncate the set
- Analyzing enzymatically digested tryptic proteomics data where the search space is small; this skill targets the inflated search space of immunopeptidomics (unspecific cleavage) searches
- Defining MHC binders from predicted binding affinity, where the correct cutoff is a percentile rank score <= 2%, not FDR filtering

## Inputs

- Search engine output files (Comet pep.xml, MS-GF+ mzIdentML, X!Tandem, MaxQuant tables) with target and decoy PSM scores
- DeepRescore result tables (*_psm_final.tsv, *_pep_final.tsv) containing q-value and Percolator_score columns
- Target-decoy protein database FASTA (e.g., RefSeq 56,659 sequences + variants + 245 contaminants + decoys)

## Outputs

- Filtered PSM and peptide tables at 1% FDR (q-value <= 0.01) at both levels
- Unique peptide counts per search engine and dataset (e.g., D1: 23400 Comet, 22789 MS-GF+, 20727 X!Tandem, 17122 MaxQuant)
- High-quality peptide sets for AutoRT transfer learning (1% FDR per at least three of four engines)

## How to apply

Search MGF-converted spectra against a target-decoy database (decoy prefix e.g. XXX_) with unspecific cleavage, 10 ppm precursor tolerance, 0.02 Da MS/MS tolerance, oxidation of methionine as variable and carbamidomethyl of cysteine as fixed modification, keeping peptide lengths 8-25. Apply the target-decoy strategy implemented in PGA to control FDR at 1% at both PSM and peptide levels, using each engine's scores (Evalue for MS-GF+/Comet/X!Tandem, PEP for MaxQuant). For DeepRescore-rescored results, filter directly on the q-value column of the *_psm_final.tsv and *_pep_final.tsv outputs with q-value <= 0.01. Record unique peptide counts after filtering to enable downstream comparison, and define 'high-quality peptides' for AutoRT transfer learning as those passing 1% FDR per at least three of the four search engines (or per engine separately for single-engine applications).

## Related tools

- **PGA** (Implements the target-decoy strategy used to control FDR at 1% at both PSM and peptide levels)
- **Comet** (Search engine whose Evalue-scored target/decoy PSMs are filtered at 1% FDR)
- **MS-GF+** (Search engine whose Evalue-scored target/decoy PSMs are filtered at 1% FDR) — https://github.com/MSGFPlus/msgfplus
- **X!Tandem** (Search engine whose Evalue-scored target/decoy PSMs are filtered at 1% FDR)
- **MaxQuant** (Search engine whose PEP-scored target/decoy PSMs are filtered at 1% FDR)
- **DeepRescore** (Produces rescored PSM/peptide tables with a q-value column used for q-value <= 0.01 filtering) — https://github.com/bzhanglab/DeepRescore

## Examples

```
nextflow run DeepRescore.nf --se comet --ms_instrument Lumos --ms_energy 0.34 --decoy_prefix XXX_   # then filter results: q-value <= 0.01 in *_psm_final.tsv / *_pep_final.tsv
```

## Evaluation signals

- Filtered result sets must contain q-value <= 0.01 entries at both PSM and peptide levels
- Recomputed unique peptide counts should match reported baselines within rounding tolerance (e.g., D1 Comet ~23400 pre-rescoring, 35683 post-rescoring)
- All-four-engine overlap fractions recomputed from filtered sets should reproduce ~33% (D1) and ~34% (D2) pre-rescoring, rising to 57% and 74% post-rescoring
- All retained peptides must fall within the length 8-25 range for identification analyses (8-11 for binding affinity evaluation)
- AutoRT high-quality training sets must consist of peptides passing 1% FDR in at least three of the four search engines

## Limitations

- The inflated search space of immunopeptidomics (no enzymatic digestion) can lead to high false positive rate and low sensitivity, making strict FDR control essential but limiting raw identification counts
- FDR filtering alone is score-threshold dependent; per-engine score types differ (Evalue vs PEP), so cross-engine comparability requires consistent 1% FDR application at both PSM and peptide levels
- DeepRescore q-value filtering (q-value <= 0.01) assumes the Percolator-rescored tables were generated with decoy prefix (e.g., XXX_) correctly specified

## Evidence

- [methods] FDR was controlled at 1% at both PSM and peptide levels: [evidence span withheld — anchor status: not_found]
