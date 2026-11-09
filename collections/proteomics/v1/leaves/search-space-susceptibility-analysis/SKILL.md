---
name: search-space-susceptibility-analysis
description: Use when comparing confident PSM retention across increasing database/search-space
  sizes in metaproteomics, to decide whether rescoring mitigates identification losses
  from expanded search spaces.
when_to_use_negative:
- Input data has not been re-searched against at least two different search spaces,
  so no largest/smallest-space retention comparison can be formed
- PSM tables are search-engine-only results without Oktoberfest rescoring, if the
  goal is to quantify the benefit conferred by rescoring
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Oktoberfest
  role: Generates Prosit-based rescoring features and produces the rescored PSM tables
    compared across search spaces
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: mokapot
  role: Performs target/decoy FDR estimation at PSM and peptide level to define confident
    PSMs (default fdr_estimation_method)
- name: Python
  role: Pandas-based loading of PSM tables, counting confident PSMs, computing retention
    rates, and writing retention_rate_comparison.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:8b53b3d3d11771652a56cca98d5d77245cba7abd1b8fd8ad4e6c2832dfd659ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Oktoberfest
  - Python
  - mokapot
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# search-space-susceptibility-analysis

## Summary

Quantifies how much data-driven rescoring (e.g., Oktoberfest/Prosit features rescored with mokapot) protects confident PSM identifications when the protein search space is enlarged, compared to search-engine-only results (MaxQuant). It computes retention rates and loss reduction across search spaces.

## When to use

Use when comparing confident PSM retention across increasing database/search-space sizes in metaproteomics, to decide whether rescoring mitigates identification losses from expanded search spaces.

## When NOT to use

- Input data has not been re-searched against at least two different search spaces, so no largest/smallest-space retention comparison can be formed
- PSM tables are search-engine-only results without Oktoberfest rescoring, if the goal is to quantify the benefit conferred by rescoring

## Inputs

- Per-search-space PSM tables from Oktoberfest-rescored runs (rescoring features and mokapot scores)
- Per-search-space MaxQuant search result tables
- FDR estimation configuration (fdr_estimation_method: 'mokapot' default or 'percolator') applied at PSM and peptide level

## Outputs

- retention_rate_comparison.csv containing confident-PSM counts per search space, retention rates (largest/smallest ratios), and PSM-loss reduction value
- Validated retention rates (81% rescoring, 3.7% MaxQuant) and 68% loss reduction

## How to apply

Load the per-search-space PSM tables (Oktoberfest-rescored and MaxQuant results) with pandas and identify confident PSMs in each using target/decoy FDR estimation at PSM and peptide level, configured via fdr_estimation_method (mokapot by default, percolator as alternative). Count confident PSMs in the smallest search space (e.g., human-only: 19,124 for rescoring, 11,724 for MaxQuant) and in the largest search space (e.g., Swissprot+IGC: 15,573 for rescoring, 434 for MaxQuant). Compute retention rates as largest/smallest-space confident PSM ratios (81% for rescoring vs 3.7% for MaxQuant) and derive the relative reduction in PSM losses conferred by rescoring (68%). Write the counts, retention rates, and loss-reduction value to retention_rate_comparison.csv. The rationale is that Prosit-based rescoring features stabilize score distributions so that identifications degrade far less when the search space grows.

## Related tools

- **Oktoberfest** (Generates Prosit-based rescoring features and produces the rescored PSM tables compared across search spaces) — https://github.com/wilhelm-lab/oktoberfest
- **mokapot** (Performs target/decoy FDR estimation at PSM and peptide level to define confident PSMs (default fdr_estimation_method))
- **Python** (Pandas-based loading of PSM tables, counting confident PSMs, computing retention rates, and writing retention_rate_comparison.csv)

## Examples

```
import pandas as pd; df = pd.read_csv('psm_table.csv'); confident = df[df['q-value'] <= 0.01]; retention = len(confident_largest) / len(confident_smallest)
```

## Evaluation signals

- Rescoring retention rate equals 81% (15,573/19,124 confident PSMs retained from human-only to Swissprot+IGC search space)
- MaxQuant retention rate equals 3.7% (434/11,724 confident PSMs retained)
- Derived PSM-loss reduction from rescoring equals 68%
- Confident-PSM identification uses the same target/decoy FDR estimation at PSM and peptide level across all search spaces
- retention_rate_comparison.csv contains raw counts, retention rates, and the loss-reduction value consistent with the paper's quantification

## Limitations

- Validation thresholds are specific to this metaproteomics benchmark (human-only vs Swissprot+IGC spaces); other datasets will yield different retention values
- Results depend on the configured fdr_estimation_method (mokapot vs percolator), which may shift confident-PSM counts
- The comparison quantifies relative retention only; absolute identifications in the largest search space are still reduced (434 MaxQuant PSMs)

## Evidence

- [results] Data-driven rescoring retained 15,573 of 19,124 (81%) confidently identified PSMs from the human-only search when re-searched against the largest search space (Swissprot+IGC).: [evidence span withheld — anchor status: not_found]
- [methods] Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot": "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
- [abstract] used in various applications, ranging from data-independent acquisition (DIA) data analysis to data-driven rescoring of search engine results: "used in various applications, ranging from data-independent acquisition (DIA) data analysis to data-driven rescoring of search engine results"
