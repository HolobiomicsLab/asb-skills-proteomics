---
name: psm-retention-quantification-across-search-spaces
description: Use when comparing PSM retention between database search spaces (e.g.,
  human-only vs Swissprot+IGC) and you need to quantify whether data-driven rescoring
  protects confident PSM identifications against search-space growth.
when_to_use_negative:
- When PSM tables have not been rescored or FDR-estimated (no target/decoy-based q-values
  at PSM and peptide level exist yet)
- When the goal is generation of spectral libraries, collision energy calibration,
  or PTM localization rather than retention quantification across search spaces
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Oktoberfest
  role: Provides the data-driven rescoring pipeline whose rescored PSM tables are
    the input for retention quantification
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: mokapot
  role: Default method for target/decoy separation and FDR estimation on PSM and peptide
    level used to define confident PSMs
- name: spectrum_io
  role: Reads search results from MaxQuant and other engines and transforms them to
    the internal format for rescoring with Oktoberfest
  repo: https://github.com/wilhelm-lab/spectrum_io
- name: Python
  role: Runtime for Oktoberfest and the pandas-based counting/ratio computation of
    retention rates
  repo: https://pypi.org/project/oktoberfest/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:8928f10bde3c0fddfb7a15d0f5c88557c7eb65903612b28d1bf6325cc3fc8e07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Oktoberfest
  - Python
  - mokapot
  - spectrum_io
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# psm-retention-quantification-across-search-spaces

## Summary

Quantifies how many confidently identified PSMs survive when the database search space is enlarged in metaproteomics, comparing data-driven rescoring (Oktoberfest/mokapot) against search-engine-only results (MaxQuant). Provides the 81% vs 3.7% retention and 68% loss-reduction metrics.

## When to use

Use when comparing PSM retention between database search spaces (e.g., human-only vs Swissprot+IGC) and you need to quantify whether data-driven rescoring protects confident PSM identifications against search-space growth.

## When NOT to use

- When PSM tables have not been rescored or FDR-estimated (no target/decoy-based q-values at PSM and peptide level exist yet)
- When the goal is generation of spectral libraries, collision energy calibration, or PTM localization rather than retention quantification across search spaces

## Inputs

- Per-search-space PSM tables from the metaproteomics reproduction (Oktoberfest-rescored results)
- Per-search-space PSM tables from MaxQuant search results
- fdr_estimation_method configuration (mokapot default, percolator alternative)

## Outputs

- retention_rate_comparison.csv with retention rates, raw confident-PSM counts, and loss-reduction value

## How to apply

Load the per-search-space PSM tables from the metaproteomics reproduction (Oktoberfest-rescored and MaxQuant results) with pandas. Identify confident PSMs in each table using target/decoy FDR estimation applied at PSM and peptide level, as configured by fdr_estimation_method (default mokapot, percolator as alternative). Count confident PSMs in the smallest search space (19,124 rescoring; 11,724 MaxQuant) and the largest (15,573 rescoring; 434 MaxQuant). Compute retention as largest/smallest-space confident PSM ratios (81% for rescoring vs 3.7% for MaxQuant) and derive the 68% reduction in PSM losses conferred by rescoring. Write retention rates, raw counts, and loss reduction to retention_rate_comparison.csv.

## Related tools

- **Oktoberfest** (Provides the data-driven rescoring pipeline whose rescored PSM tables are the input for retention quantification) — https://github.com/wilhelm-lab/oktoberfest
- **mokapot** (Default method for target/decoy separation and FDR estimation on PSM and peptide level used to define confident PSMs)
- **spectrum_io** (Reads search results from MaxQuant and other engines and transforms them to the internal format for rescoring with Oktoberfest) — https://github.com/wilhelm-lab/spectrum_io
- **Python** (Runtime for Oktoberfest and the pandas-based counting/ratio computation of retention rates) — https://pypi.org/project/oktoberfest/

## Examples

```
import oktoberfest as ok
ok.re.rescore_with_mokapot(search_results)
```

## Evaluation signals

- Computed retention rate for rescoring must equal 81% (15,573 of 19,124 confident PSMs retained from human-only to Swissprot+IGC search space)
- Computed retention rate for MaxQuant must equal 3.7% (434 of 11,724 confident PSMs)
- Derived PSM loss reduction from rescoring must equal 68%
- Confident PSMs must be defined via target/decoy FDR estimation applied at both PSM and peptide level (mokapot or percolator)
- Output must contain retention rates, raw counts, and the loss-reduction value in retention_rate_comparison.csv

## Limitations

- Retention is sensitive to the FDR estimation method configured (mokapot vs percolator); the default mokapot underlies the reported 81%/3.7%/68% figures
- Numbers are specific to the Swissprot+IGC largest search space and human-only smallest search space comparison in the metaproteomics reproduction; other database pairs require re-counting
- Search-engine-only (MaxQuant) comparisons collapse dramatically (434 PSMs), so conclusions about robustness apply to rescoring workflows, not generic re-searching

## Evidence

- [methods] Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot": "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
- [methods] re.generate_features, re.merge_input, re.rescore_with_mokapot, re.rescore_with_percolator: "re.generate_features
   re.merge_input
   re.rescore_with_mokapot
   re.rescore_with_percolator"
- [readme] read search results from different search engines and transform to internal format for rescoring: [evidence span withheld — anchor status: not_found]
- [abstract] used in various applications, ranging from DIA data analysis to data-driven rescoring of search engine results: "used in various applications, ranging from data-independent acquisition (DIA) data analysis to data-driven rescoring of search engine results"
