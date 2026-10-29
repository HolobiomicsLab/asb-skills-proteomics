---
name: melting-curve-piecewise-smoothing
description: Use when normalized DSF melting curves need smoothing that preserves
  the original curve shape across the denaturation transition, especially when noisy
  pre- and post-denaturation regions distort peak-derived Tm estimation.
when_to_use_negative:
- Input is raw, un-normalized fluorescence data — SimpleDSFviewer's MC normalization
  step must be run first so the minimum and maximum fluorescence points are correctly
  located.
- The analysis goal is Tm determination or peak reporting rather than curve conditioning
  — the smoothing module is a preprocessing step feeding Tm analysis, not the analysis
  itself.
- The trace lacks a distinguishable minimum-to-maximum denaturation transition, so
  the three-segment partition (before minimum / minimum-to-maximum / after maximum)
  cannot be located reliably.
tools:
- name: SimpleDSFviewer
  role: User-friendly Matlab interface that orchestrates the three-part smoothing
    of normalized melting curves and previews the smoothed output alongside normalization
    and Tm analysis.
  repo: https://github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Runtime environment supplying the built-in smooth function applied independently
    to each of the three melting-curve segments.
- name: Matlab Runtime
  role: Allows SimpleDSFviewer's smoothing workflow to be executed without a full
    Matlab installation.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:267605fa60aa248fba4d996d501bec14e99cd591c42b7cb80740ada6df4ce735
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hscsun/SimpleDSFviewer-5.0
  tools:
  - Matlab
  - Matlab Runtime
  - SimpleDSFviewer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# melting-curve-piecewise-smoothing

## Summary

Smooths normalized differential scanning fluorimetry (DSF) melting curves by applying Matlab's built-in smooth function separately to three segments of each trace (pre-denaturation, denaturation, post-denaturation), preserving the original curve shape for downstream Tm analysis in SimpleDSFviewer.

## When to use

Use when normalized DSF melting curves need smoothing that preserves the original curve shape across the denaturation transition, especially when noisy pre- and post-denaturation regions distort peak-derived Tm estimation.

## When NOT to use

- Input is raw, un-normalized fluorescence data — SimpleDSFviewer's MC normalization step must be run first so the minimum and maximum fluorescence points are correctly located.
- The analysis goal is Tm determination or peak reporting rather than curve conditioning — the smoothing module is a preprocessing step feeding Tm analysis, not the analysis itself.
- The trace lacks a distinguishable minimum-to-maximum denaturation transition, so the three-segment partition (before minimum / minimum-to-maximum / after maximum) cannot be located reliably.

## Inputs

- Normalized melting curve data loaded in the Matlab workspace (post MC-normalization)
- User-defined smoothing range (span parameter for Matlab's smooth function)

## Outputs

- smoothed_curves.csv containing the reassembled, three-part smoothed melting curves

## How to apply

Load the normalized melting curve data into the Matlab workspace, then for each curve locate the minimum and maximum fluorescence points to partition the trace into three segments: before the minimum (before denaturation onset), from the minimum to the maximum (the denaturation part), and after the maximum. Apply Matlab's built-in smooth function independently to each of the three segments, using the user-definable smoothing range supplied as input. Reassemble the three smoothed segments in temperature order so the original curve shape is maintained, then write the result to smoothed_curves.csv. Splitting at the minimum and maximum is the key rationale: it prevents the smoothing window from blurring the sharp denaturation transition that carries the melting-temperature information.

## Related tools

- **SimpleDSFviewer** (User-friendly Matlab interface that orchestrates the three-part smoothing of normalized melting curves and previews the smoothed output alongside normalization and Tm analysis.) — https://github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Runtime environment supplying the built-in smooth function applied independently to each of the three melting-curve segments.)
- **Matlab Runtime** (Allows SimpleDSFviewer's smoothing workflow to be executed without a full Matlab installation.)

## Examples

```
segSm = smooth(segment, span); % applied per-segment in the Matlab workspace to each of the 3 parts, then reassembled to smoothed_curves.csv
```

## Evaluation signals

- The smoothed trace preserves the original curve shape: the minimum fluorescence point and maximum fluorescence point used to partition the curve remain identifiable after smoothing.
- All three segments (before the minimum, minimum-to-maximum denaturation part, after the maximum) were smoothed independently and reassembled in correct temperature order with no gaps or overlaps.
- The same user-definable smoothing range was applied consistently to all three segments of a given curve.
- The output file smoothed_curves.csv exists, is loadable, and aligns one-to-one with the input normalized melting curves (same curves, same temperature points).

## Limitations

- Smoothing quality depends on a well-defined minimum and maximum fluorescence point; ambiguous denaturation transitions will degrade the three-part segmentation.
- The smoothing range is user-definable, so inappropriate span choices can distort the curve the method is designed to preserve.
- Smoothing is a preprocessing step — it requires prior normalized melting curves and does not by itself produce melting temperatures.

## Evidence

- [methods] Smoothing is applied in three parts to preserve the original curve shape: from the beginning to the minimum value (before denaturation onset): [evidence span withheld — anchor status: not_found]
- [methods] from the minimum to the maximum (denaturation part), and from the maximum to the end, with a user-definable smoothing range: [evidence span withheld — anchor status: not_found]
