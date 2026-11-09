---
name: signal-smoothing-window-selection
description: Use when a normalized melting curve from DSF analysis needs smoothing
  before Tm analysis and noise reduction must not distort the pre- and post-denaturation
  baseline shape.
when_to_use_negative:
- The curve is a raw (un-normalized) DSF trace — the article applies smoothing to
  melting curves after MC normalization.
- The goal is direct Tm analysis on already-smoothed data — this skill is only the
  smoothing step, not the downstream melting temperature (Tm) analysis.
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SimpleDSFviewer
  role: User-friendly Matlab interface that implements the three-part smoothing step
    on normalized melting curves as part of its DSF analysis pipeline
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Runtime environment in which SimpleDSFviewer is developed and whose built-in
    smooth function is applied to each of the three curve segments
- name: Matlab Runtime
  role: Alternative execution mode allowing SimpleDSFviewer (and its smoothing module)
    to be run without a full Matlab installation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:e98a2c7433eaeedc6cfab6816c806f03909ac359871cdcb3a38a25666a008859
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - Matlab Runtime
  - SimpleDSFviewer
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# signal-smoothing-window-selection

## Summary

Smoothing of normalized melting curves (MCs) in SimpleDSFviewer by applying Matlab's smooth function in three curve segments — before the minimum, between the minimum and maximum (denaturation part), and after the maximum — with a user-definable smoothing range, to reduce noise while preserving the original curve shape.

## When to use

Use when a normalized melting curve from DSF analysis needs smoothing before Tm analysis and noise reduction must not distort the pre- and post-denaturation baseline shape.

## When NOT to use

- The curve is a raw (un-normalized) DSF trace — the article applies smoothing to melting curves after MC normalization.
- The goal is direct Tm analysis on already-smoothed data — this skill is only the smoothing step, not the downstream melting temperature (Tm) analysis.

## Inputs

- normalized melting curve (MC) data loaded in the Matlab workspace
- user-defined smoothing range (span) value

## Outputs

- smoothed melting curves written to smoothed_curves.csv
- smoothed curve preview within SimpleDSFviewer

## How to apply

Load the normalized melting curve into the Matlab workspace where SimpleDSFviewer runs. For each curve, locate the minimum and maximum fluorescence points and partition the trace into three segments: beginning-to-minimum (before denaturation onset), minimum-to-maximum (the denaturation part), and maximum-to-end. Apply Matlab's built-in smooth function independently to each segment, using the user-definable smoothing range supplied as input. Reassemble the three smoothed segments in temperature order so the original curve shape is maintained, and write the result to smoothed_curves.csv. The three-part strategy is the rationale: smoothing the denaturation region separately from the baselines preserves the characteristic transition shape needed for subsequent Tm analysis.

## Related tools

- **SimpleDSFviewer** (User-friendly Matlab interface that implements the three-part smoothing step on normalized melting curves as part of its DSF analysis pipeline) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Runtime environment in which SimpleDSFviewer is developed and whose built-in smooth function is applied to each of the three curve segments)
- **Matlab Runtime** (Alternative execution mode allowing SimpleDSFviewer (and its smoothing module) to be run without a full Matlab installation)

## Examples

```
smoothed = smooth(meltingCurve, span); % applied per segment within SimpleDSFviewer (Matlab)
```

## Evaluation signals

- The smoothed output is split at the located minimum and maximum fluorescence points, with smoothing applied independently to each of the three segments.
- The original curve shape is maintained after reassembly — no distortion of the baseline regions or the denaturation transition.
- The smoothing range used matches the user-definable value supplied as input.
- The smoothed curves are written to smoothed_curves.csv in temperature order.
- The smoothed curves preview correctly in SimpleDSFviewer before downstream Tm analysis.

## Limitations

- Requires locating a minimum and maximum in each trace; curves without a clear denaturation transition (e.g. flat or ambiguous traces) cannot be partitioned into the three segments as designed.
- Data analysis in DSF 'remains a challenge, requiring improved methods', so smoothing choices may still require manual tuning via the user-definable range.
- Smoothing is coupled to the paper's normalization and Tm analysis workflow within SimpleDSFviewer rather than being a standalone general-purpose smoother.

## Evidence

- [methods] Three-part smoothing preserves curve shape: [evidence span withheld — anchor status: not_found]
- [results] Three-part segmentation rationale: [evidence span withheld — anchor status: not_found]
