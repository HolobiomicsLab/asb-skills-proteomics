---
name: derivative-curve-peak-identification
description: Use when raw multi-well DSF fluorescence melting curves (e.g. a 96-well
  plate) need Tm values and the FD method (Tm = temperature at max of the FDC) should
  be applied, including verification that smoothing preserves Tm.
when_to_use_negative:
- Input data are not DSF fluorescence melting curves (e.g. already-summarized Tm tables
  or other thermal-shift output formats), so no FDC can be computed.
- The analysis goal requires model-based (fitting) Tm estimation rather than the FD
  method of taking the temperature at the maximum of the first derivative curve.
- Curves lack a discernible FDC maximum (e.g. no folded-protein transition), so max(FDC)
  Tm identification is not meaningful.
tools:
- name: SimpleDSFviewer
  role: Loads raw DSF melting curve data, performs melting curve smoothing, computes
    first derivative curves (FDC), and extracts Tm as the temperature at max(FDC)
    per Equation 3.
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Environment in which SimpleDSFviewer is developed and run to compute the FDC-based
    Tm values for all 96 wells.
- name: Matlab Runtime
  role: Alternative runtime allowing SimpleDSFviewer to execute the FDC/Tm analysis
    without a full Matlab installation.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:a20a25685dac04815cc454f327c14e9ed95fe04f430c7130547487041b5ee242
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - Matlab Runtime
  - SimpleDSFviewer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# derivative-curve-peak-identification

## Summary

Determines protein melting temperatures (Tm) from differential scanning fluorimetry (DSF) melting curves using the FD method: Tm is the temperature at the maximum of the first derivative curve (FDC), as implemented in SimpleDSFviewer. The skill also supports checking that curve smoothing does not shift the reported Tm values.

## When to use

Use when raw multi-well DSF fluorescence melting curves (e.g. a 96-well plate) need Tm values and the FD method (Tm = temperature at max of the FDC) should be applied, including verification that smoothing preserves Tm.

## When NOT to use

- Input data are not DSF fluorescence melting curves (e.g. already-summarized Tm tables or other thermal-shift output formats), so no FDC can be computed.
- The analysis goal requires model-based (fitting) Tm estimation rather than the FD method of taking the temperature at the maximum of the first derivative curve.
- Curves lack a discernible FDC maximum (e.g. no folded-protein transition), so max(FDC) Tm identification is not meaningful.

## Inputs

- Raw DSF fluorescence melting curve data for a multi-well plate (e.g. 96 wells) loadable into SimpleDSFviewer
- Temperature ramp series paired with fluorescence readings per well

## Outputs

- Per-well Tm values from unsmoothed melting curves via max(FDC)
- Per-well Tm values from smoothed melting curves via max(FDC)
- Comparison table of paired unsmoothed vs smoothed Tm values (e.g. tm_unsmoothed_vs_smoothed.csv)

## How to apply

Load the raw 96-well DSF fluorescence melting curve data into SimpleDSFviewer (a Matlab tool usable in the Matlab workspace or via Matlab Runtime). For each well, compute the first derivative curve (FDC) of the unsmoothed melting curve and take Tm as the temperature at max(FDC), per Equation 3 of the article. Optionally apply the SimpleDSFviewer melting curve smoothing routine and repeat the max(FDC) Tm computation on the smoothed curves. Tabulate the paired unsmoothed and smoothed Tm values per well (e.g. tm_unsmoothed_vs_smoothed.csv) and compare them; for wells with folded proteins the two sets should be very similar, demonstrating that smoothing does not change Tm values.

## Related tools

- **SimpleDSFviewer** (Loads raw DSF melting curve data, performs melting curve smoothing, computes first derivative curves (FDC), and extracts Tm as the temperature at max(FDC) per Equation 3.) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Environment in which SimpleDSFviewer is developed and run to compute the FDC-based Tm values for all 96 wells.)
- **Matlab Runtime** (Alternative runtime allowing SimpleDSFviewer to execute the FDC/Tm analysis without a full Matlab installation.)

## Examples

```
>> SimpleDSFviewer  % launch in Matlab workspace (or via Matlab Runtime); load 96-well DSF melting curves, compute FDC, take Tm = temperature at max(FDC)
```

## Evaluation signals

- Each reported Tm corresponds to the temperature at the maximum of the well's first derivative curve (FDC), consistent with Equation 3 of the article.
- Paired Tm values from smoothed and unsmoothed melting curves are very similar for wells with folded proteins — smoothing should not change Tm values.
- The output table contains paired unsmoothed and smoothed Tm entries for all 96 wells of the exemplar plate.
- All Tm values fall within the temperature range of the DSF temperature ramp.
- The comparison is written to the expected file (e.g. tm_unsmoothed_vs_smoothed.csv) with one row per well.

## Limitations

- DSF data analysis remains a challenge, requiring improved methods (per the article's abstract), so FD-derived Tm should be interpreted alongside curve quality.
- If smoothed and unsmoothed Tm values diverge substantially for a well, the curve may be noisy or the protein not properly folded, warranting manual inspection.
- No changelog found for the SimpleDSFviewer repository, so version-specific behavior of the smoothing routine is not documented.

## Evidence

- [abstract] SimpleDSFviewer integrates melting curve (MC) normalization, smoothing, and melting temperature (Tm) analysis and directly previews analyzed data, providing an efficient analysis tool for DSF.: "SimpleDSFviewer integrates melting curve (MC) normalization, smoothing, and melting temperature (Tm) analysis and directly previews analyzed data, providing an efficient analysis tool for DSF."
- [abstract] SimpleDSFviewer is developed in Matlab, and it is freely available for all users to use in Matlab workspace or with Matlab Runtime.: "SimpleDSFviewer is developed in Matlab, and it is freely available for all users to use in Matlab workspace or with Matlab Runtime."
- [abstract] However, data analysis remains a challenge, requiring improved methods.: "However, data analysis remains a challenge, requiring improved methods."
