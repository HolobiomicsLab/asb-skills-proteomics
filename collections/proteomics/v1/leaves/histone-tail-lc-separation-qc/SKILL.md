---
name: histone-tail-lc-separation-qc
description: Use when replicate RP/WCX-HILIC LC-MS/MS runs of histone tails are available
  and run-to-run retention-time stability of H3, H4, and H2A N-terminal tail peaks
  must be checked against fixed minute-thresholds before PTM quantification.
when_to_use_negative:
- The goal is MS/MS spectral validation or PTM quantification from Mascot output (use
  Histone Coder/isoScale instead — this skill only assesses LC retention-time reproducibility)
- Only a single LC-MS/MS run is available, so no run-to-run (max–min) variation can
  be computed
- The chromatography data are tryptic bottom-up peptides rather than intact histone
  N-terminal tails separated by RP/WCX-HILIC
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pandas
  role: Loads the replicate-run retention-time table, groups by histone tail class,
    computes max–min run-to-run variation, and writes the pass/fail summary CSV
- name: Histone Coder
  role: Upstream middle-down software whose PTM localization/quantification results
    depend on the reproducible RP/WCX-HILIC tail separation checked by this skill
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: isoScale
  role: Downstream quantification of cofragmenting isobaric histone tail species from
    the MS/MS spectra acquired after the LC separation validated here
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:a664dde34797d5c9633cca9cbc24c0e3d904ba4681a3bd7c3d9f29d2d48973d3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - isoScale
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# histone-tail-lc-separation-qc

## Summary

Quality-control skill that quantifies run-to-run retention-time reproducibility of histone N-terminal tail peaks separated by the hybrid RP trap / weak cation exchange–hydrophilic interaction LC (RP/WCX-HILIC) setup, so that downstream middle-down PTM quantification rests on a stable chromatographic method.

## When to use

Use when replicate RP/WCX-HILIC LC-MS/MS runs of histone tails are available and run-to-run retention-time stability of H3, H4, and H2A N-terminal tail peaks must be checked against fixed minute-thresholds before PTM quantification.

## When NOT to use

- The goal is MS/MS spectral validation or PTM quantification from Mascot output (use Histone Coder/isoScale instead — this skill only assesses LC retention-time reproducibility)
- Only a single LC-MS/MS run is available, so no run-to-run (max–min) variation can be computed
- The chromatography data are tryptic bottom-up peptides rather than intact histone N-terminal tails separated by RP/WCX-HILIC

## Inputs

- Retention times of histone tail chromatographic peaks from four replicate RP/WCX-HILIC LC-MS/MS runs (e.g., Table S1 of Sidoli et al., Proteomics 2014)
- Tail-class labels (H3, H4, H2A N-terminal tails) accompanying each retention-time entry
- Acceptance thresholds: <3 min for H3 and H2A tails, <6 min for H4 tails

## Outputs

- lc_retention_time_variation.csv: summary table with per-tail-class retention-time variation (max–min spread in minutes) and pass/fail flag against thresholds

## How to apply

Load the retention times of the histone tail chromatographic peaks measured across the replicate RP/WCX-HILIC LC-MS/MS runs (e.g., Table S1 of the paper) into pandas. Group the retention times by histone tail class — H3, H4, and H2A N-terminal tails — and compute the run-to-run variation as the max–min spread in minutes for each class. Compare each tail class's observed spread against the acceptance thresholds defined for this setup: less than 3 min for H3 and H2A tails and less than 6 min for H4 tails. Write a summary table listing each tail class's retention-time variation and a pass/fail flag against its threshold to lc_retention_time_variation.csv. The rationale is that the RP trap column coupled to subsequent WCX-HILIC must reproducibly separate hypermodified histone N-terminal tails for coeluting/isobaric species to be quantified consistently by the middle-down workflow.

## Related tools

- **pandas** (Loads the replicate-run retention-time table, groups by histone tail class, computes max–min run-to-run variation, and writes the pass/fail summary CSV)
- **Histone Coder** (Upstream middle-down software whose PTM localization/quantification results depend on the reproducible RP/WCX-HILIC tail separation checked by this skill) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **isoScale** (Downstream quantification of cofragmenting isobaric histone tail species from the MS/MS spectra acquired after the LC separation validated here) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true

## Examples

```
import pandas as pd; rt = pd.read_csv('table_s1.csv'); var = rt.groupby('tail_class')['retention_time_min'].agg(lambda s: s.max()-s.min()); var.to_csv('lc_retention_time_variation.csv')
```

## Evaluation signals

- Computed H3 and H2A N-terminal tail retention-time spread across all four replicate runs is <3 min
- Computed H4 tail retention-time spread across all four replicate runs is <6 min
- Summary CSV contains one row per tail class (H3, H4, H2A) with both a variation value in minutes and an explicit pass/fail flag
- Recomputing the max–min spread from the raw retention-time table reproduces the values written to lc_retention_time_variation.csv
- All four replicate runs are represented in the grouping (no dropped or duplicated run entries)

## Limitations

- Acceptance thresholds (<3 min for H3/H2A, <6 min for H4) are calibrated specifically for this RP/WCX-HILIC histone-tail setup and may not transfer to other LC configurations
- Reproducibility was assessed over only four replicate runs, so the max–min spread is a coarse estimate of run-to-run variation
- The assessment covers retention-time variation only; it does not check peak intensity, peak shape, or MS/MS acquisition quality

## Evidence

- [abstract] Combined an RP trap column with subsequent weak cation exchange-hydrophilic interaction LC for histone tail separation: [evidence span withheld — anchor status: not_found]
- [other] Across four replicate runs, retention-time variation was less than 3 min for histone H3 and H2A N-terminal tails and less than 6 min for histone H4 tails.: [evidence span withheld — anchor status: not_found]
