---
name: curve-segment-boundary-detection
description: Use when a normalized DSF melting curve must be partitioned before smoothing
  or downstream Tm analysis, and the denaturation region needs to be delimited by
  the curve's minimum and maximum fluorescence points.
when_to_use_negative:
- The input is raw (un-normalized) DSF fluorescence data — normalize the melting curves
  first
- The goal is Tm estimation on an already-smoothed curve rather than defining smoothing
  boundaries
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SimpleDSFviewer
  role: Implements the min/max boundary detection that partitions each normalized
    melting curve into the three segments used for smoothing
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Environment in which the curve partitioning and subsequent per-segment smooth
    function are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:f2c0db6c5608fe9d4524ecb063d01cbb1d9f8a9dcfaa76c165a7b1fd6afe4479
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - SimpleDSFviewer
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# curve-segment-boundary-detection

## Summary

Locates the minimum and maximum fluorescence points on a normalized melting curve (MC) and uses them as boundaries to partition the trace into three segments — pre-denaturation, denaturation, and post-denaturation — so that smoothing can be applied per-segment without distorting the original curve shape.

## When to use

Use when a normalized DSF melting curve must be partitioned before smoothing or downstream Tm analysis, and the denaturation region needs to be delimited by the curve's minimum and maximum fluorescence points.

## When NOT to use

- The input is raw (un-normalized) DSF fluorescence data — normalize the melting curves first
- The goal is Tm estimation on an already-smoothed curve rather than defining smoothing boundaries

## Inputs

- Normalized melting curve data (fluorescence vs. temperature) loaded in the Matlab workspace

## Outputs

- Per-curve boundary indices for the minimum and maximum fluorescence points
- Three curve segments (pre-minimum, minimum-to-maximum, post-maximum) ready for independent smoothing

## How to apply

Load the normalized melting curve data into the Matlab workspace. For each curve, locate the minimum fluorescence point and the maximum fluorescence point, which delimit the denaturation part of the trace. Partition the curve into three segments: from the beginning to the minimum (before denaturation onset), from the minimum to the maximum (the denaturation part), and from the maximum to the end. These boundary points define the user-definable smoothing range used by SimpleDSFviewer's three-part smoothing step, so the segments can subsequently be smoothed independently with Matlab's built-in smooth function and reassembled in temperature order, preserving the original curve shape.

## Related tools

- **SimpleDSFviewer** (Implements the min/max boundary detection that partitions each normalized melting curve into the three segments used for smoothing) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Environment in which the curve partitioning and subsequent per-segment smooth function are executed)

## Evaluation signals

- The minimum fluorescence point falls before denaturation onset and the maximum after the denaturation transition on each curve
- Exactly three segments are produced per curve, covering the full temperature range with no gaps when reassembled
- Reassembling the three smoothed segments in temperature order reproduces the original curve shape (no discontinuities at segment joins)
- Boundary detection is applied consistently across all curves loaded for a DSF analysis run

## Limitations

- Boundary detection depends on the curves being normalized first; on un-normalized data the min/max points may not correspond to the denaturation boundaries
- The smoothing range defined by these boundaries is user-definable, so different users may produce different segment partitions

## Evidence

- [abstract] Three-part smoothing: "SimpleDSFviewer integrates melting curve (MC) normalization, smoothing, and melting temperature (Tm) analysis and directly previews analyzed data, providing an efficient analysis tool for DSF."
