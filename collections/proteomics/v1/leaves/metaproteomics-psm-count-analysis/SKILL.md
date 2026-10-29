---
name: metaproteomics-psm-count-analysis
description: Use when comparing confident PSM counts across search spaces of different
  sizes in metaproteomics (e.g., human-only vs Swissprot+IGC) to test whether data-driven
  rescoring robustifies identification against database growth.
when_to_use_negative:
- Input is a single search space only — retention-rate computation requires PSM tables
  from at least two search-space sizes (smallest and largest, e.g., human-only and
  Swissprot+IGC).
- PSM tables lack target/decoy labels, since confident-PSM calling depends on target/decoy
  FDR estimation at PSM and peptide level.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Oktoberfest
  role: Provides the data-driven rescoring pipeline whose rescored PSM tables are
    compared across search spaces
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: mokapot
  role: Performs target/decoy FDR estimation on PSM and peptide level to define confident
    PSMs (default fdr_estimation_method)
- name: Python
  role: Used with pandas to load PSM tables, count confident PSMs, compute retention
    rates, and write retention_rate_comparison.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:b7ee069654340f5192f6aa0cd262c7e8765bf438261e1c7d0ffa33b9aabe4efa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Oktoberfest
  - Python
  - mokapot
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metaproteomics-psm-count-analysis

## Summary

Quantifies how much of the confident PSM identifications survive when the metaproteomics search space is enlarged (e.g., human-only vs Swissprot+IGC), comparing data-driven rescoring (Oktoberfest/mokapot) against MaxQuant baseline, and derives retention rates and PSM-loss reduction.

## When to use

Use when comparing confident PSM counts across search spaces of different sizes in metaproteomics (e.g., human-only vs Swissprot+IGC) to test whether data-driven rescoring robustifies identification against database growth.

## When NOT to use

- Input is a single search space only — retention-rate computation requires PSM tables from at least two search-space sizes (smallest and largest, e.g., human-only and Swissprot+IGC).
- PSM tables lack target/decoy labels, since confident-PSM calling depends on target/decoy FDR estimation at PSM and peptide level.

## Inputs

- per-search-space PSM tables from the metaproteomics reproduction (Oktoberfest-rescored results)
- per-search-space PSM tables from MaxQuant search results
- fdr_estimation_method configuration (mokapot default, percolator alternative)

## Outputs

- retention_rate_comparison.csv containing retention rates (81% rescoring, 3.7% MaxQuant), raw confident-PSM counts (19,124 / 15,573 and 11,724 / 434), and the 68% PSM-loss-reduction value

## How to apply

Load the per-search-space PSM tables from the metaproteomics reproduction (Oktoberfest-rescored and MaxQuant results) with pandas. Identify confident PSMs in each table using target/decoy FDR estimation applied at PSM and peptide level, configured via fdr_estimation_method (default mokapot, percolator as alternative, as in Oktoberfest's re.rescore_with_mokapot / re.rescore_with_percolator). Count confident PSMs in the smallest search space (19,124 total for rescoring; 11,724 for MaxQuant) and the largest (15,573 for rescoring; 434 for MaxQuant). Compute retention rates as largest/smallest-space confident PSM ratios (81% rescoring vs 3.7% MaxQuant) and derive the 68% reduction in PSM losses conferred by rescoring. Write retention rates, raw counts, and the loss-reduction value to retention_rate_comparison.csv.

## Related tools

- **Oktoberfest** (Provides the data-driven rescoring pipeline whose rescored PSM tables are compared across search spaces) — https://github.com/wilhelm-lab/oktoberfest
- **mokapot** (Performs target/decoy FDR estimation on PSM and peptide level to define confident PSMs (default fdr_estimation_method))
- **Python** (Used with pandas to load PSM tables, count confident PSMs, compute retention rates, and write retention_rate_comparison.csv)

## Examples

```
import pandas as pd; rescoring = pd.read_csv('rescoring_psm_tables'); retention_rate_comparison.to_csv('retention_rate_comparison.csv', index=False)
```

## Evaluation signals

- Computed rescoring retention rate equals 81% (15,573 / 19,124).
- Computed MaxQuant retention rate equals 3.7% (434 / 11,724).
- Derived PSM-loss reduction equals 68%.
- Confident-PSM counts derive from target/decoy FDR estimation applied consistently at both PSM and peptide level across all search-space tables.
- retention_rate_comparison.csv contains retention rates, raw counts, and the loss-reduction value.

## Limitations

- Validation depends on exact reproduction of the paper's search spaces and FDR configuration; deviations in fdr_estimation_method (mokapot vs percolator) or search-space definitions change the counts.
- The comparison is anchored to one metaproteomics reproduction (MaxQuant baseline vs Oktoberfest rescoring); other search engines are not covered by these validated numbers.

## Evidence

- [other] Data-driven rescoring retained 15,573 of 19,124 (81%) confidently identified PSMs from the human-only search when re-searched against the largest search space (Swissprot+IGC).: [evidence span withheld — anchor status: not_found]
- [methods] Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot": "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
- [methods] re.generate_features re.merge_input re.rescore_with_mokapot re.rescore_with_percolator: "re.generate_features
re.merge_input
re.rescore_with_mokapot
re.rescore_with_percolator"
- [readme] read search results from different search engines (MaxQuant, MSFragger, Sage, Xisearch) or a generic csv format and transform them to the internal format for rescoring: [evidence span withheld — anchor status: not_found]
