---
name: gain-loss-rescoring-visualization
description: Use when comparing PSM identifications across rescoring conditions (e.g.
  MaxQuant-only, Andromeda+Percolator, Andromeda+Prosit+Percolator) and you need to
  report gained/lost PSM counts and a gain/loss plot per allele or condition.
when_to_use_negative:
- Input spectra have not yet been converted and annotated (raw to mzML via pp.convert_raw_to_mzml,
  then pp.convert_d_to_hdf/pp.load_spectra and peak annotation) — no PSM counts exist
  to compare yet
- Rescoring has not been run (no re.generate_features / re.merge_input / re.rescore_with_percolator
  output), so condition-level PSM sets are undefined
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Oktoberfest
  role: Provides the rescoring pipeline outputs and the pl.plot_gain_loss plotting
    function used to render the Figure 3A-B gain/loss decomposition
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Percolator
  role: Produces the Andromeda+Percolator and Andromeda+Prosit+Percolator PSM sets
    (1% FDR) that are compared against MaxQuant-only counts
- name: mokapot
  role: re.rescore_with_mokapot cross-check of the rescoring results feeding the comparison
- name: Prosit
  role: Supplies predicted fragment intensity/iRT features (via pr.Predictor / pr.Koina)
    that define the improved Andromeda+Prosit+Percolator condition
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:096e1d4e0f2a7d9a207be4a88c520586b1db364667b2c73b67a93e2cc35f0b67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Oktoberfest
  - Percolator
  - Prosit
  - Python
  - mokapot
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# gain-loss-rescoring-visualization

## Summary

Quantifies and visualizes PSM-level gains and losses when Prosit-based rescoring (Oktoberfest) is compared against an original search engine result, e.g. MaxQuant/Andromeda, producing the Figure 3A-B style gain/loss decomposition at 1% PSM FDR.

## When to use

Use when comparing PSM identifications across rescoring conditions (e.g. MaxQuant-only, Andromeda+Percolator, Andromeda+Prosit+Percolator) and you need to report gained/lost PSM counts and a gain/loss plot per allele or condition.

## When NOT to use

- Input spectra have not yet been converted and annotated (raw to mzML via pp.convert_raw_to_mzml, then pp.convert_d_to_hdf/pp.load_spectra and peak annotation) — no PSM counts exist to compare yet
- Rescoring has not been run (no re.generate_features / re.merge_input / re.rescore_with_percolator output), so condition-level PSM sets are undefined

## Inputs

- Rescored PSM result tables per condition (MaxQuant-only, Andromeda+Percolator, Andromeda+Prosit+Percolator) at 1% PSM FDR
- Original published PSM counts or result files serving as the baseline (e.g. 78,842 PSMs across four HLA alleles)
- PSM counts per allele (A*11:01, A*31:01, C*12:03, G*12:03)

## Outputs

- Per-condition, per-allele PSM count summary
- Gain/loss decomposition (retained, gained, lost PSMs) versus the baseline
- Gain/loss figure analogous to Figure 3A-B from pl.plot_gain_loss

## How to apply

After rescoring with re.rescore_with_percolator (and cross-checking with re.rescore_with_mokapot) at 1% FDR on PSM level, count PSMs per condition per allele (MaxQuant-only, Andromeda+Percolator, Andromeda+Prosit+Percolator). Decompose identifications into retained, gained, and lost PSMs relative to the original published result (e.g. the four HLA alleles A*11:01, A*31:01, C*12:03, G*12:03 baseline of 78,842 PSMs), and compute the relative performance change (21% increase to 95,451 PSMs with 9,296 gained PSMs and an 89% reduction in losses). Render the decomposition with pl.plot_gain_loss, reproducing Figure 3A-B. Validate that reproduced condition totals match reported values: 30,866 (MaxQuant, 1% FDR), 38,196 (Andromeda+Percolator), 95,451 (Andromeda+Prosit+Percolator).

## Related tools

- **Oktoberfest** (Provides the rescoring pipeline outputs and the pl.plot_gain_loss plotting function used to render the Figure 3A-B gain/loss decomposition) — https://github.com/wilhelm-lab/oktoberfest
- **Percolator** (Produces the Andromeda+Percolator and Andromeda+Prosit+Percolator PSM sets (1% FDR) that are compared against MaxQuant-only counts)
- **mokapot** (re.rescore_with_mokapot cross-check of the rescoring results feeding the comparison)
- **Prosit** (Supplies predicted fragment intensity/iRT features (via pr.Predictor / pr.Koina) that define the improved Andromeda+Prosit+Percolator condition)

## Examples

```
from oktoberfest import plotting as pl; pl.plot_gain_loss(psm_counts_per_condition, output_dir="figures")  # Figure 3A-B: 95,451 vs 78,842 PSMs, 9,296 gained
```

## Evaluation signals

- Reproduced PSM counts match reported values: 30,866 (MaxQuant, 1% FDR), 38,196 (Andromeda+Percolator), 95,451 (Andromeda+Prosit+Percolator)
- Computed relative gain equals ~21% over the originally published 78,842 PSMs
- Gained PSMs total 9,296 and losses show an 89% reduction relative to the previous results
- Figure 3A-B style plot renders gain/loss decomposition for all four HLA alleles across the three conditions
- All comparisons are made at a consistent 1% PSM-level FDR threshold

## Limitations

- Findings are grounded in the paper's two use cases ('Oktoberfest reproduces and even improves previously published rescoring results on two distinct use cases'); counts will differ for other datasets or FDR thresholds
- Comparison is PSM-level at 1% FDR; peptide- or protein-level gains are not what this decomposition reports
- The baseline must come from a comparable MaxQuant/Andromeda search; the article's discussion section contains no additional changelog content

## Evidence

- [other] Count PSMs per condition per allele (MaxQuant-only, Andromeda+Percolator, Andromeda+Prosit+Percolator), compute the 21% gain over the original 78,842 PSMs, and render the gain/loss decomposition with pl.plot_gain_loss (Figure 3A-B): [evidence span withheld — anchor status: not_found]
- [results] Oktoberfest rescoring identified 95,451 PSMs versus 78,842 PSMs in the previous results, a 21% performance increase, with 9296 gained PSMs and an 89% reduction in losses.: [evidence span withheld — anchor status: not_found]
- [abstract] We demonstrate its ability to reproduce and even improve our results from previously published rescoring analyses on two distinct use cases.: "We demonstrate its ability to reproduce and even improve our results from previously published rescoring analyses on two distinct use cases."
- [readme] It offers an end to end pipeline that takes search results, predicts peptide properties using koina, plots summaries and quality control figures and performs FDR estimation with either mokapot or percolator.: [evidence span withheld — anchor status: external_doc]
