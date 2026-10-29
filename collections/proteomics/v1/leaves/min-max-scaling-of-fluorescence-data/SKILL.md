---
name: min-max-scaling-of-fluorescence-data
description: Use when raw DSF fluorescence-vs-temperature melting curves must be placed
  on a common 0–1 scale per well (e.g. before smoothing or Tm analysis in SimpleDSFviewer),
  so curves from different wells are comparable.
when_to_use_negative:
- The input is already normalized melting-curve output (values within [0, 1] per curve)
  from a prior SimpleDSFviewer run
- The goal is Tm extraction on raw unnormalized curves rather than producing the 0–1
  scaled MC used downstream
- The data are not DSF fluorescence-versus-temperature melting curves (e.g. isothermal
  fluorescence reads or spectra), where per-curve min-max scaling has no melting-curve
  interpretation
edam_operation: http://edamontology.org/operation_3435
tools:
- name: SimpleDSFviewer
  role: 'Implements the melting-curve (MC) normalization step (Equation 1: min-subtraction
    and max-division) and writes the normalized curves'
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Environment in which SimpleDSFviewer runs the DSF data import and normalization;
    curves can be processed in the Matlab workspace
- name: Matlab Runtime
  role: Alternative runtime allowing SimpleDSFviewer's normalization to be executed
    without a full Matlab installation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:1f4e4e9e37c9e929df8377992f1bcd798e70347682d5666cb6211f1df8061a64
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - Matlab Runtime
  - SimpleDSFviewer
  edam_operation: http://edamontology.org/operation_3435
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# min-max-scaling-of-fluorescence-data

## Summary

Applies SimpleDSFviewer's melting-curve (MC) normalization (Equation 1): each DSF fluorescence-versus-temperature curve is min-subtracted and divided by its max-minus-min range, rescaling every curve to the 0–1 interval without altering curve shape.

## When to use

Use when raw DSF fluorescence-vs-temperature melting curves must be placed on a common 0–1 scale per well (e.g. before smoothing or Tm analysis in SimpleDSFviewer), so curves from different wells are comparable.

## When NOT to use

- The input is already normalized melting-curve output (values within [0, 1] per curve) from a prior SimpleDSFviewer run
- The goal is Tm extraction on raw unnormalized curves rather than producing the 0–1 scaled MC used downstream
- The data are not DSF fluorescence-versus-temperature melting curves (e.g. isothermal fluorescence reads or spectra), where per-curve min-max scaling has no melting-curve interpretation

## Inputs

- Raw DSF fluorescence-versus-temperature melting curve data per well (e.g. from Matlab-imported DSF runs such as His-FGF7/His-FGF10 exemplar wells)

## Outputs

- normalized_melting_curves.csv containing per-well fluorescence curves rescaled to the 0–1 range

## How to apply

Load the raw DSF fluorescence-versus-temperature data (e.g. His-FGF7 and His-FGF10 exemplar wells) into Matlab via SimpleDSFviewer's data import. For each well, apply the MC normalization of Equation 1: subtract the curve's minimum fluorescence value from every point, then divide by the curve's (max - min) range, mapping the curve to the [0, 1] interval. Because the transform is affine per curve, the shape of the melting curve — and therefore the melting temperature (Tm) subsequently extracted — is unchanged. Write the normalized per-well curves to normalized_melting_curves.csv and validate that every normalized value lies within [0, 1].

## Related tools

- **SimpleDSFviewer** (Implements the melting-curve (MC) normalization step (Equation 1: min-subtraction and max-division) and writes the normalized curves) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Environment in which SimpleDSFviewer runs the DSF data import and normalization; curves can be processed in the Matlab workspace)
- **Matlab Runtime** (Alternative runtime allowing SimpleDSFviewer's normalization to be executed without a full Matlab installation)

## Examples

```
norm = (raw - min(raw, [], 1)) ./ (max(raw, [], 1) - min(raw, [], 1)); writetable(array2table(norm), 'normalized_melting_curves.csv')
```

## Evaluation signals

- Every normalized fluorescence value in normalized_melting_curves.csv lies within the [0, 1] range
- Each well's minimum maps to exactly 0 and maximum to exactly 1 after normalization
- The affine transform leaves the curve shape unchanged, so the derived melting temperature (Tm) per well matches that from the unnormalized curve
- Output file exists with one normalized curve per input well (row/column correspondence preserved with the raw import)

## Limitations

- Min-max scaling is per-curve: wells with very low signal-to-noise will have noise amplified to the full 0–1 range, though the article does not give a cutoff for excluding such wells
- SimpleDSFviewer is a Matlab program, requiring either a Matlab installation or Matlab Runtime to run
- The article notes DSF data analysis remains a challenge, requiring improved methods, so normalization alone does not guarantee reliable Tm values

## Evidence
