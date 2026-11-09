---
name: per-well-plate-data-processing
description: Use when raw DSF fluorescence-versus-temperature data from a multi-well
  plate must be compared across wells whose absolute fluorescence intensities differ,
  i.e. when each melting curve needs per-curve min-max normalization prior to smoothing
  or Tm analysis.
when_to_use_negative:
- The fluorescence data are not melting curves from a DSF plate experiment and per-curve
  0–1 rescaling would be meaningless
- Each melting curve is already min-max normalized to the 0–1 range, so re-applying
  Equation 1 would not change values
- The goal is absolute fluorescence-intensity comparison across wells, where the per-curve
  rescaling of Equation 1 would destroy the intensity information
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3369
tools:
- name: SimpleDSFviewer
  role: Implements the MC normalization (Equation 1) as the first step of its analysis
    pipeline, applied per well, before smoothing and Tm analysis
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Environment in which SimpleDSFviewer and its normalization routine are developed
    and run
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab Runtime
  role: Alternative execution environment for running SimpleDSFviewer without a full
    Matlab installation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:802f17dda0a42d81592aacb1fcc58cdbd732f128462a9920ab04b5767bcf946a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - Matlab Runtime
  - SimpleDSFviewer
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3369
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# per-well-plate-data-processing

## Summary

Applies SimpleDSFviewer's melting-curve (MC) normalization per well to differential scanning fluorimetry (DSF) fluorescence-versus-temperature data: subtracting each curve's minimum and dividing by its (max − min) range (Equation 1) to rescale every MC into the 0–1 interval before smoothing and Tm analysis.

## When to use

Use when raw DSF fluorescence-versus-temperature data from a multi-well plate must be compared across wells whose absolute fluorescence intensities differ, i.e. when each melting curve needs per-curve min-max normalization prior to smoothing or Tm analysis.

## When NOT to use

- The fluorescence data are not melting curves from a DSF plate experiment and per-curve 0–1 rescaling would be meaningless
- Each melting curve is already min-max normalized to the 0–1 range, so re-applying Equation 1 would not change values
- The goal is absolute fluorescence-intensity comparison across wells, where the per-curve rescaling of Equation 1 would destroy the intensity information

## Inputs

- Raw DSF fluorescence-versus-temperature data per plate well (e.g. His-FGF7 / His-FGF10 exemplar wells, as imported by SimpleDSFviewer)
- Well identifier / plate layout to define per-curve groupings

## Outputs

- Per-well normalized melting curves scaled to the 0–1 range (e.g. normalized_melting_curves.csv)
- Previewed normalized curves within the SimpleDSFviewer interface, ready for smoothing and Tm analysis

## How to apply

Load the raw DSF fluorescence-versus-temperature data for each plate well (exemplars in the paper are His-FGF7 and His-FGF10 wells) into the SimpleDSFviewer pipeline in Matlab. For each well independently, apply the MC normalization of Equation 1: subtract the curve's minimum fluorescence value from every point, then divide by the curve's (max − min) range, which maps the curve onto the 0–1 interval per curve. Perform this per well rather than across the whole plate, because each curve is scaled relative only to its own minimum and maximum; the normalization rescales intensities but does not alter the shape of the melting curve. The normalized curves can then be exported (e.g. to normalized_melting_curves.csv) for downstream smoothing and melting-temperature (Tm) analysis within SimpleDSFviewer.

## Related tools

- **SimpleDSFviewer** (Implements the MC normalization (Equation 1) as the first step of its analysis pipeline, applied per well, before smoothing and Tm analysis) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Environment in which SimpleDSFviewer and its normalization routine are developed and run) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab Runtime** (Alternative execution environment for running SimpleDSFviewer without a full Matlab installation)

## Examples

```
normalized = (f - min(f)) / (max(f) - min(f)); % per-well MC normalization (Equation 1) in Matlab
```

## Evaluation signals

- Every normalized fluorescence value lies within the [0, 1] range for each well, confirming correct min-max normalization per curve
- The minimum of each normalized curve equals 0 and its maximum equals 1
- The shape of each melting curve is unchanged after normalization (same transition temperature/inflection as the raw curve)
- The output file normalized_melting_curves.csv exists and contains one normalized curve per input well
- Each curve was scaled using only its own min and max, not plate-wide statistics

## Limitations

- Normalization requires a well-defined minimum and maximum per curve; a flat or monotonic trace with no melting transition yields a degenerate (max − min) range
- Per-curve min-max rescaling removes absolute fluorescence intensity information, so only curve shape and Tm remain comparable
- DSF data analysis remains a challenge, requiring improved methods (per the article's abstract), so normalized output may still need careful inspection of previewed curves

## Evidence

- [methods] MC normalization maps each curve to 0-1: [evidence span withheld — anchor status: not_found]
