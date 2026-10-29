---
name: chromatographic-retention-time-reproducibility-analysis
description: Use when replicate LC-MS/MS runs of histone tail separations are available
  and the goal is to quantify run-to-run retention-time variation per histone tail
  class (H3, H4, H2A) against explicit minute-level acceptance thresholds.
when_to_use_negative:
- The goal is quantifying relative abundance of histone PTM marks (use isoScale/Histone
  Coder quantification instead of retention-time reproducibility assessment)
- Retention times come from different LC configurations or gradients rather than replicate
  runs of the same RP/WCX-HILIC hybrid setup — the <3 min / <6 min thresholds are
  specific to this platform
- Only a single run is available, so a max-min run-to-run spread cannot be computed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: pandas
  role: Load retention times from the replicate LC-MS/MS runs, group by histone tail
    class, and compute the max-min run-to-run variation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:2b88f49cdc41ffefcd78172c97810b489b2c896139290cfbcb488de8dff8b8a5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# chromatographic-retention-time-reproducibility-analysis

## Summary

Assesses run-to-run retention-time stability of histone N-terminal tail peaks separated by the RP trap column + weak cation exchange-HILIC (WCX-HILIC) hybrid LC setup by computing the max-min spread of chromatographic peak retention times across replicate LC-MS/MS runs and comparing against tail-class-specific thresholds.

## When to use

Use when replicate LC-MS/MS runs of histone tail separations are available and the goal is to quantify run-to-run retention-time variation per histone tail class (H3, H4, H2A) against explicit minute-level acceptance thresholds.

## When NOT to use

- The goal is quantifying relative abundance of histone PTM marks (use isoScale/Histone Coder quantification instead of retention-time reproducibility assessment)
- Retention times come from different LC configurations or gradients rather than replicate runs of the same RP/WCX-HILIC hybrid setup — the <3 min / <6 min thresholds are specific to this platform
- Only a single run is available, so a max-min run-to-run spread cannot be computed

## Inputs

- Retention times of histone tail chromatographic peaks from four replicate RP/WCX-HILIC LC-MS/MS runs (e.g. Table S1)
- Histone tail class labels (H3, H4, H2A N-terminal tails) for each peak

## Outputs

- Per-tail-class retention-time variation table (max-min spread in minutes) with pass/fail flag against thresholds (<3 min H3/H2A; <6 min H4), e.g. lc_retention_time_variation.csv

## How to apply

Load the retention times of the histone tail chromatographic peaks measured across the four replicate RP/WCX-HILIC LC-MS/MS runs (e.g. from Table S1) with pandas. Group retention times by histone tail class (H3, H4, and H2A N-terminal tails) and compute the run-to-run variation as the max-min spread in minutes for each class. Compare each tail class's observed variation against the acceptance thresholds: <3 min for H3 and H2A tails and <6 min for H4 tails — H4 tails are allowed a wider window because their separation behaviour in the hybrid RP/WCX-HILIC setup is more variable. Write a summary table listing per-tail-class retention-time variation and a pass/fail flag against the thresholds, e.g. to lc_retention_time_variation.csv. The rationale is that the combined RP trap column with subsequent weak cation exchange-HILIC separation must be stable enough that hypermodified tail peaks elute reproducibly for automated PTM-localization sequencing by ETD MS/MS.

## Related tools

- **pandas** (Load retention times from the replicate LC-MS/MS runs, group by histone tail class, and compute the max-min run-to-run variation)

## Examples

```
import pandas as pd; rt = pd.read_csv('table_s1_retention_times.csv'); var = rt.groupby('tail_class')['retention_time_min'].apply(lambda s: s.max()-s.min())
```

## Evaluation signals

- Computed max-min retention-time spread is <3 min for H3 and H2A N-terminal tails and <6 min for H4 tails across the four replicate runs
- Summary CSV (e.g. lc_retention_time_variation.csv) contains one row per tail class with a numeric variation value and a pass/fail flag consistent with the thresholds
- All four replicate runs are represented for every tail class before computing the spread (no silently dropped replicates)
- Variation is reported in minutes and matches values derivable from the Table S1 retention times

## Limitations

- Thresholds (<3 min for H3/H2A, <6 min for H4) are specific to the RP trap + WCX-HILIC hybrid setup and should be re-derived for other LC configurations
- Assessment captures run-to-run variation only; it does not validate peak identification or PTM localization quality
- The software repository README provides no changelog or retention-time tooling — this analysis is a manual/pandas workflow, not part of Histone Coder or isoScale

## Evidence

- [other] Retention-time reproducibility thresholds: [evidence span withheld — anchor status: not_found]
