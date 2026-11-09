---
name: metaproteomics-search-space-comparison
description: Use when a MaxQuant (or similar) search over large or growing FASTA search
  spaces (e.g. metaproteomics against Swiss-Prot plus IGC) yields collapsing PSM counts,
  and you want to quantify how deep-learning-based rescoring restores identifications.
when_to_use_negative:
- Input is DIA data or a spectral-library-generation/collision-energy-calibration
  job, not a MaxQuant search-results rescoring task
- Search results come from a search engine other than MaxQuant without an Oktoberfest
  conversion path, or spectra are not RAW/mzML for the FTMS profile assumed by the
  20 ppm fragment tolerance
- No Koina server access is available for Prosit predictions, since feature generation
  depends on predicted fragment intensities
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Oktoberfest
  role: 'End-to-end rescoring pipeline: converts MaxQuant results, generates Prosit-predicted
    libraries, builds rescoring features, and runs mokapot FDR estimation; provides
    pp.*, re.*, and pl.* functions used per search space'
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Prosit
  role: Deep-learning model providing fragment-intensity and iRT predictions used
    as the spectral reference for rescoring
- name: Koina
  role: Prediction server (koina.wilhelmlab.org:443, ssl true) interfaced by pr.Predictor
    to retrieve Prosit predictions
- name: mokapot
  role: PSM-level target/decoy FDR estimation engine invoked via re.rescore_with_mokapot
    (fdr_estimation_method 'mokapot')
- name: MaxQuant
  role: Native search engine whose per-search-space PSM counts form the baseline comparison
    for the rescoring results
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:25b573baedf65c37da27f85beed2db5a7f96ff67008ea1915b08a7885f7e4f6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Koina
  - MaxQuant
  - Oktoberfest
  - Prosit
  - Python
  - mokapot
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# metaproteomics-search-space-comparison

## Summary

Reproduces the Oktoberfest-based data-driven rescoring analysis on the metaproteomics dataset PXD010871 searched against increasingly large search spaces (Swiss-Prot Human, +Bacteria, +IGC), showing that Prosit/mokapot rescoring with Koina-predicted fragment intensities largely rescues the PSM loss that MaxQuant suffers as the search space grows.

## When to use

Use when a MaxQuant (or similar) search over large or growing FASTA search spaces (e.g. metaproteomics against Swiss-Prot plus IGC) yields collapsing PSM counts, and you want to quantify how deep-learning-based rescoring restores identifications.

## When NOT to use

- Input is DIA data or a spectral-library-generation/collision-energy-calibration job, not a MaxQuant search-results rescoring task
- Search results come from a search engine other than MaxQuant without an Oktoberfest conversion path, or spectra are not RAW/mzML for the FTMS profile assumed by the 20 ppm fragment tolerance
- No Koina server access is available for Prosit predictions, since feature generation depends on predicted fragment intensities

## Inputs

- PXD010871 MaxQuant search results from PRIDE (msms.txt and related output per search space)
- PXD010871 RAW spectra files from PRIDE
- FASTA search-space databases: Swiss-Prot Human, Swiss-Prot Human + Bacteria, Swiss-Prot Human + Bacteria + IGC
- Koina server access for Prosit fragment-intensity and iRT predictions

## Outputs

- Converted/split MaxQuant search results in Oktoberfest internal format per search space
- Prosit-predicted spectral libraries and iRT values (Koina)
- mokapot-rescored PSM tables with PSM-level FDR per search space
- PSM count table (MaxQuant-only vs. rescoring) per search space
- psm_rescoring_comparison.png reproducing the Figure 3C-D comparison

## How to apply

Download PXD010871 MaxQuant search results and RAW spectra from PRIDE and configure an Oktoberfest 'Rescoring' job with search_results_type 'Maxquant', spectra_type 'raw', and fdr_estimation_method 'mokapot'. Convert the MaxQuant output for each of the three search spaces via pp.convert_search and pp.split_search, then prepare peptides with pp.filter_peptides_for_model. Retrieve Prosit fragment-intensity and iRT predictions through pr.Predictor interfacing the Koina server (prediction_server 'koina.wilhelmlab.org:443', ssl true). Annotate observed peaks and build rescoring features with re.generate_features using massTolerance 20 ppm for FTMS and a 1.2 Da precursor peak-removal window (p_window), then rescore each search space with re.rescore_with_mokapot at PSM-level target/decoy FDR. Tabulate MaxQuant-only versus rescoring PSM counts per search space and reproduce the Figure 3C-D comparison (MaxQuant collapsing 11,724 to 434 PSMs; rescoring recovering 19,124 to 114,510 PSMs) with oktoberfest.pl plotting functions, saving psm_rescoring_comparison.png.

## Related tools

- **Oktoberfest** (End-to-end rescoring pipeline: converts MaxQuant results, generates Prosit-predicted libraries, builds rescoring features, and runs mokapot FDR estimation; provides pp.*, re.*, and pl.* functions used per search space) — https://github.com/wilhelm-lab/oktoberfest
- **Prosit** (Deep-learning model providing fragment-intensity and iRT predictions used as the spectral reference for rescoring)
- **Koina** (Prediction server (koina.wilhelmlab.org:443, ssl true) interfaced by pr.Predictor to retrieve Prosit predictions)
- **mokapot** (PSM-level target/decoy FDR estimation engine invoked via re.rescore_with_mokapot (fdr_estimation_method 'mokapot'))
- **MaxQuant** (Native search engine whose per-search-space PSM counts form the baseline comparison for the rescoring results)

## Examples

```
import oktoberfest as ok; ok.pp.convert_search(...); ok.re.generate_features(...); ok.re.rescore_with_mokapot(...); ok.pl.plot_psm_rescoring_comparison(out='psm_rescoring_comparison.png')
```

## Evaluation signals

- Reproduced PSM counts match the reported pattern: MaxQuant collapses from 11,724 to 434 PSMs across the three search spaces, while rescoring recovers 19,124 to 114,510 PSMs on Swissprot+IGC
- Human-only reference identifications after rescoring retain 81% (15,573 of 19,124) versus 3.7% for MaxQuant on the largest search space
- psm_rescoring_comparison.png exists and reproduces the Figure 3C-D MaxQuant-vs-rescoring comparison
- Rescoring runs complete at 1% PSM-level target/decoy FDR as estimated by mokapot for each of the three search spaces
- Koina predictions succeed over SSL (koina.wilhelmlab.org:443) and feature generation respects massTolerance 20 ppm (FTMS) and p_window 1.2 Da precursor peak removal

## Limitations

- The pipeline assumes MaxQuant search output and RAW input; other search engines require their own conversion path and spectra_type handling
- Prediction quality depends on access to the Koina server and Prosit model applicability to the observed fragmentation/charge regimes
- Results are reported for a specific dataset (PXD010871) and search-space set (Swiss-Prot Human / +Bacteria / +IGC); absolute counts will differ on other data

## Evidence

- [abstract] we present Oktoberfest, an open source Python package of our spectral library generation and rescoring pipeline: "we present Oktoberfest, an open source Python package of our spectral library generation and rescoring pipeline"
- [methods] Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot": "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
- [methods] Access to functions that interface either a Koina server to retrieve predictions from various prediction models.: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
- [readme] It offers an end to end pipeline that takes search results, predicts peptide properties using koina, plots summaries and quality control figures and performs FDR estimation with either mokapot or percolator.: [evidence span withheld — anchor status: external_doc]
- [methods] re.rescore_with_mokapot: "re.generate_features
   re.merge_input
   re.rescore_with_mokapot
   re.rescore_with_percolator"
