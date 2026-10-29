---
name: dsf-fluorescence-data-processing
description: Use when normalized DSF melting curves contain fluorescence noise that
  would distort melting temperature (Tm) estimation, and a shape-preserving smoothing
  step is needed before Tm analysis.
when_to_use_negative:
- The input traces are raw, un-normalized DSF fluorescence data that have not yet
  passed SimpleDSFviewer's melting curve (MC) normalization step
- The goal is melting temperature (Tm) estimation on already-smoothed curves, for
  which the Tm analysis module rather than the smoothing module is appropriate
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: SimpleDSFviewer
  role: Provides the three-part smoothing module for normalized DSF melting curves,
    with a user-definable smoothing range
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Host environment in which the smooth function and SimpleDSFviewer smoothing
    module execute
- name: Matlab Runtime
  role: Alternative runtime for running SimpleDSFviewer outside the Matlab workspace
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:a991dbf0703662a411742633f87ff83012d26de7b069d95cec7031723443bd5c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - Matlab Runtime
  - SimpleDSFviewer
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# dsf-fluorescence-data-processing

## Summary

Smoothing of normalized differential scanning fluorimetry (DSF) melting curves using SimpleDSFviewer's three-part module, which applies Matlab's smooth function segment-by-segment so the original curve shape is preserved for downstream Tm analysis.

## When to use

Use when normalized DSF melting curves contain fluorescence noise that would distort melting temperature (Tm) estimation, and a shape-preserving smoothing step is needed before Tm analysis.

## When NOT to use

- The input traces are raw, un-normalized DSF fluorescence data that have not yet passed SimpleDSFviewer's melting curve (MC) normalization step
- The goal is melting temperature (Tm) estimation on already-smoothed curves, for which the Tm analysis module rather than the smoothing module is appropriate

## Inputs

- Normalized melting curve fluorescence data (temperature vs. fluorescence traces, post-normalization step of SimpleDSFviewer)
- User-defined smoothing range

## Outputs

- smoothed_curves.csv (smoothed melting curve traces, three segments reassembled in temperature order)

## How to apply

Load the normalized melting curve (MC) data into the Matlab workspace, since SimpleDSFviewer is developed in Matlab and runs in the Matlab workspace or with Matlab Runtime. For each curve, locate the minimum and maximum fluorescence points to partition the trace into three segments: from the beginning to the minimum (before denaturation onset), from the minimum to the maximum (the denaturation part), and from the maximum to the end. Apply Matlab's built-in smooth function independently to each of the three segments, using the user-definable smoothing range provided as input. Reassemble the three smoothed segments in temperature order so the original curve shape is maintained, and write the result to smoothed_curves.csv for downstream melting temperature analysis.

## Related tools

- **SimpleDSFviewer** (Provides the three-part smoothing module for normalized DSF melting curves, with a user-definable smoothing range) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Host environment in which the smooth function and SimpleDSFviewer smoothing module execute)
- **Matlab Runtime** (Alternative runtime for running SimpleDSFviewer outside the Matlab workspace)

## Evaluation signals

- Each smoothed curve is partitioned at the minimum and maximum fluorescence points into exactly three segments before smoothing
- The three smoothed segments are reassembled in temperature order with no gaps, overlaps, or reordered temperatures, so the original curve shape is maintained
- smoothed_curves.csv exists and contains one smoothed trace per input normalized curve
- The smoothing range used equals the user-definable range supplied as input
- The denaturation part (minimum-to-maximum segment) retains its minimum and maximum fluorescence positions after smoothing

## Limitations

- Smoothing is segment-wise rather than global; if minimum/maximum detection misidentifies denaturation onset, segment boundaries and thus the smoothed curve may be wrong
- The smoothing range must be user-supplied; no default range is documented in the provided context
- No changelog was found for the repository, so version-specific behavior of the smoothing module (github.com/hscsun/SimpleDSFviewer-5.0) is unverified

## Evidence

- [methods] three-part segment smoothing: [evidence span withheld — anchor status: not_found]
- [methods] segment-wise smooth function application: [evidence span withheld — anchor status: not_found]
