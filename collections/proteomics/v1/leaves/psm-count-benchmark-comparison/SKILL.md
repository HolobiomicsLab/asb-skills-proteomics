---
name: psm-count-benchmark-comparison
description: Use when a reanalysis must be validated against published PSM counts,
  or when reporting how much Prosit-based rescoring improves identification rates
  over a baseline search engine score at a fixed FDR.
when_to_use_negative:
- You only have final identification lists without the underlying search engine scores
  and predicted features needed for rescoring — counting alone cannot reproduce the
  conditions under test.
- The comparison targets peptide- or protein-level FDR rather than PSM level, since
  the reported benchmark counts (30,866 / 38,196 / 95,451) are all at 1% PSM-level
  FDR.
- The search results are not from a supported engine convertible via pp.convert_search
  (this workflow is built around MaxQuant results).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Oktoberfest
  role: Provides the rescoring functions (re.generate_features, re.merge_input, re.rescore_with_percolator,
    re.rescore_with_mokapot) and plotting (pl.plot_gain_loss) used to produce and
    compare PSM counts per condition
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Percolator
  role: Semi-supervised rescoring engine for the Andromeda+Percolator and Andromeda+Prosit+Percolator
    conditions whose PSM counts at 1% FDR are benchmarked
- name: mokapot
  role: Alternative FDR estimation backend (re.rescore_with_mokapot) used to cross-check
    Percolator PSM counts
- name: Koina
  role: Prediction server (koina.wilhelmlab.org:443, ssl=true) queried via pr.Predictor/pr.Koina
    for Prosit fragment intensity and iRT predictions feeding the rescoring features
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:27ba5a4399df0c808b9ca481e7ab79902b094a2473b947a16c24e263539e56d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Koina
  - Oktoberfest
  - Percolator
  - Python
  - mokapot
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# psm-count-benchmark-comparison

## Summary

Benchmark data-driven rescoring performance by counting PSMs at 1% PSM-level FDR across conditions (search-engine-only, Andromeda+Percolator, Andromeda+Prosit+Percolator) and comparing them against previously published counts to quantify gains and losses.

## When to use

Use when a reanalysis must be validated against published PSM counts, or when reporting how much Prosit-based rescoring improves identification rates over a baseline search engine score at a fixed FDR.

## When NOT to use

- You only have final identification lists without the underlying search engine scores and predicted features needed for rescoring — counting alone cannot reproduce the conditions under test.
- The comparison targets peptide- or protein-level FDR rather than PSM level, since the reported benchmark counts (30,866 / 38,196 / 95,451) are all at 1% PSM-level FDR.
- The search results are not from a supported engine convertible via pp.convert_search (this workflow is built around MaxQuant results).

## Inputs

- MaxQuant search results for the four HLA allele runs, converted to Oktoberfest internal format (pp.convert_search, pp.split_search)
- Spectra files converted raw → mzML (pp.convert_raw_to_mzml) and loaded as HDF (pp.convert_d_to_hdf / pp.load_spectra)
- Merged rescoring feature tables combining the MaxQuant (Andromeda) score with Prosit-predicted features (spectral angle, intensity-based features) via re.merge_input
- Published baseline PSM counts from the original rescoring study (78,842 PSMs)

## Outputs

- PSM counts per condition per allele at 1% PSM-level FDR (MaxQuant-only, Andromeda+Percolator, Andromeda+Prosit+Percolator)
- Gain/loss decomposition versus the published baseline (e.g. 95,451 vs 78,842 PSMs, 21% increase, 9296 gained, 89% loss reduction)
- Gain/loss plots (Figure 3A–B style) rendered with pl.plot_gain_loss

## How to apply

Run the rescoring pipeline (Oktoberfest: pp.convert_search/pp.split_search on MaxQuant results, Prosit intensity/irt predictions via pr.Predictor or pr.Koina against koina.wilhelmlab.org:443 with ssl=true, re.generate_features + re.merge_input) and then rescore with re.rescore_with_percolator and, as a cross-check, re.rescore_with_mokapot, targeting 1% FDR at PSM level. Count PSMs per condition per sample/allele (here the four HLA alleles A*11:01, A*31:01, C*12:03, G*12:03) for MaxQuant-only, Andromeda+Percolator, and Andromeda+Prosit+Percolator. Compare the best condition against the published baseline (78,842 PSMs) and decompose the difference into gained and lost PSMs, rendered with pl.plot_gain_loss. The rationale is that identical FDR thresholds make raw PSM counts a directly comparable sensitivity metric across scoring schemes, and gain/loss decomposition reveals whether gains come from new identifications rather than swaps. Validation requires the reproduced counts to match reported values exactly (30,866 / 38,196 / 95,451), yielding the ~21% increase, 9296 gained PSMs, and an 89% reduction in losses.

## Related tools

- **Oktoberfest** (Provides the rescoring functions (re.generate_features, re.merge_input, re.rescore_with_percolator, re.rescore_with_mokapot) and plotting (pl.plot_gain_loss) used to produce and compare PSM counts per condition) — https://github.com/wilhelm-lab/oktoberfest
- **Percolator** (Semi-supervised rescoring engine for the Andromeda+Percolator and Andromeda+Prosit+Percolator conditions whose PSM counts at 1% FDR are benchmarked)
- **mokapot** (Alternative FDR estimation backend (re.rescore_with_mokapot) used to cross-check Percolator PSM counts)
- **Koina** (Prediction server (koina.wilhelmlab.org:443, ssl=true) queried via pr.Predictor/pr.Koina for Prosit fragment intensity and iRT predictions feeding the rescoring features)

## Examples

```
import oktoberfest as ok; ok.re.merge_input(...)  # then re.rescore_with_percolator at 1% PSM FDR and pl.plot_gain_loss to compare 95,451 vs 78,842 PSMs
```

## Evaluation signals

- Reproduced MaxQuant-only count at 1% PSM FDR equals 30,866.
- Andromeda+Percolator count equals 38,196 and Andromeda+Prosit+Percolator count equals 95,451 PSMs.
- Computed improvement is ~21% (95,451 vs published 78,842), with 9296 gained PSMs and an 89% reduction in losses.
- mokapot and Percolator rescoring of the same merged features yield consistent PSM counts (cross-check).
- Gain/loss plot (pl.plot_gain_loss) reproduces the Figure 3A–B decomposition.

## Limitations

- Count comparisons are only valid under matched thresholds: all conditions must be filtered to 1% FDR at PSM level.
- Results depend on peak-annotation mass tolerances matched to the mass analyzer (20 for FTMS, 40 for TOF, 0.35 for ITMS, in da or ppm via unitMassTolerance); mismatched tolerances shift PSM counts.
- Percolator and mokapot can disagree, so counts should be cross-checked across both FDR estimation backends before reporting.

## Evidence

- [abstract] Oktoberfest reproduces and even improves previously published rescoring results on two distinct use cases: "We demonstrate its ability to reproduce and even improve our results from previously published rescoring analyses on two distinct use cases."
- [methods] FDR estimation backends: "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
- [methods] rescoring function inventory: "re.generate_features
   re.merge_input
   re.rescore_with_mokapot
   re.rescore_with_percolator"
- [readme] end-to-end pipeline scope: [evidence span withheld — anchor status: external_doc]
