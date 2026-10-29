---
name: rescoring-vs-search-engine-performance-comparison
description: Use when you have PSM-level result tables from multiple database search
  spaces (e.g. human-only vs Swissprot+IGC in metaproteomics) and need to quantify
  whether rescoring mitigates identification losses from search-space growth.
when_to_use_negative:
- Input is a single search space only — retention rates require matched smallest/largest
  search-space PSM tables.
- The comparison targets peptide/protein-level or spectral-library metrics rather
  than confident PSM retention across search spaces.
- Rescoring was performed with a tool other than Oktoberfest's Prosit-based feature
  generation, so the 81%/68% reference values do not apply.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Oktoberfest
  role: Generates Prosit-based rescoring features and produces the rescored PSM tables
    whose retention is benchmarked
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: mokapot
  role: Performs target/decoy FDR estimation on PSM and peptide level to define confident
    PSMs (default fdr_estimation_method)
- name: spectrum_io
  role: Reads search results from MaxQuant (and other engines) in generic csv format
    and transforms them to Oktoberfest's internal rescoring format
  repo: https://github.com/wilhelm-lab/spectrum_io
- name: Python
  role: Environment for loading PSM tables with pandas and computing retention rates
    and loss reduction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:fe2460f01d60058cba2bbfc4c3d7fa467fb18c737dcb5b2767e9cd92be4c232b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Oktoberfest
  - Python
  - mokapot
  - spectrum_io
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# rescoring-vs-search-engine-performance-comparison

## Summary

Quantifies how well Prosit-based data-driven rescoring (Oktoberfest + mokapot/percolator) preserves confidently identified PSMs as the search space grows, benchmarked against raw search-engine results (MaxQuant). It converts per-search-space PSM counts into retention rates and a PSM-loss reduction figure.

## When to use

Use when you have PSM-level result tables from multiple database search spaces (e.g. human-only vs Swissprot+IGC in metaproteomics) and need to quantify whether rescoring mitigates identification losses from search-space growth.

## When NOT to use

- Input is a single search space only — retention rates require matched smallest/largest search-space PSM tables.
- The comparison targets peptide/protein-level or spectral-library metrics rather than confident PSM retention across search spaces.
- Rescoring was performed with a tool other than Oktoberfest's Prosit-based feature generation, so the 81%/68% reference values do not apply.

## Inputs

- Per-search-space PSM tables from Oktoberfest-rescored results (e.g. csv/tsv per search space)
- Per-search-space PSM tables from MaxQuant search results
- fdr_estimation_method configuration ('mokapot' default or 'percolator')

## Outputs

- retention_rate_comparison.csv with retention rates (81% rescoring vs 3.7% MaxQuant), raw confident-PSM counts per search space, and the 68% PSM-loss reduction

## How to apply

Load the per-search-space PSM tables from the Oktoberfest-rescored and MaxQuant results with pandas. Identify confident PSMs in each table using target/decoy FDR estimation applied at PSM and peptide level, configured via fdr_estimation_method (mokapot by default, percolator as the alternative). Count confident PSMs in the smallest search space (19,124 for rescoring; 11,724 for MaxQuant) and the largest (15,573 for rescoring; 434 for MaxQuant), then compute retention rates as largest/smallest-space ratios (81% vs 3.7%). Derive the 68% reduction in PSM losses conferred by rescoring, write the rates, raw counts, and loss-reduction value to retention_rate_comparison.csv, and validate the outputs against the expected 81%/3.7%/68% values. The rationale is that search-space expansion inflates the target-decoy competition and collapses search-engine-only identifications, while Prosit-based rescoring features stabilize scoring.

## Related tools

- **Oktoberfest** (Generates Prosit-based rescoring features and produces the rescored PSM tables whose retention is benchmarked) — https://github.com/wilhelm-lab/oktoberfest
- **mokapot** (Performs target/decoy FDR estimation on PSM and peptide level to define confident PSMs (default fdr_estimation_method))
- **spectrum_io** (Reads search results from MaxQuant (and other engines) in generic csv format and transforms them to Oktoberfest's internal rescoring format) — https://github.com/wilhelm-lab/spectrum_io
- **Python** (Environment for loading PSM tables with pandas and computing retention rates and loss reduction)

## Examples

```
import oktoberfest as ok
```

## Evaluation signals

- Computed rescoring retention rate must equal 81% (15,573/19,124 confident PSMs).
- Computed MaxQuant retention rate must equal 3.7% (434/11,724 confident PSMs).
- Derived PSM-loss reduction must equal 68%.
- Confident-PSM identification must rely on target/decoy FDR estimation at both PSM and peptide level under the configured fdr_estimation_method.
- retention_rate_comparison.csv must contain the retention rates, raw counts per search space, and the loss-reduction value.

## Limitations

- Benchmark values (19,124, 11,724, 15,573, 434, 81%, 3.7%, 68%) come from the specific human-only vs Swissprot+IGC metaproteomics reproduction; other datasets will yield different rates.
- FDR estimation method choice ('mokapot' vs 'percolator') affects which PSMs count as confident and thus the retention rates.

## Evidence

- [methods] FDR estimation at PSM and peptide level with mokapot or percolator: "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
- [readme] MaxQuant search results are convertible to Oktoberfest's internal rescoring format: [evidence span withheld — anchor status: not_found]
- [methods] Oktoberfest provides rescore_with_mokapot and rescore_with_percolator entry points: "re.rescore_with_mokapot
   re.rescore_with_percolator"
- [abstract] Oktoberfest is demonstrated by reproducing published rescoring analyses: "We demonstrate its ability to reproduce and even improve our results from previously published rescoring analyses on two distinct use cases."
