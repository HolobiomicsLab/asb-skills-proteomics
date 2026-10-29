---
name: scientific-task-formulation
description: Use when a research question must be converted into an executable analysis
  sub-task but the available section text lacks grounded methods steps, so the task
  scope, required tools, and validation criterion must be defined before analysis
  can proceed.
when_to_use_negative:
- The provided section text already contains grounded methods spans from which verbatim
  workflow steps and a validation metric can be selected — then execute the analysis
  rather than re-formulate the task
- The research question concerns a technique other than DSF thermal-stability analysis
  (e.g. sequencing or mass-spectrometry data), where the FD-method/DSF framing does
  not transfer
- The analysis goal is parameter tuning of SimpleDSFviewer itself rather than defining
  what a new sub-task should measure
tools:
- name: SimpleDSFviewer
  role: Target analysis environment named in the formulated sub-task; its melting
    curve (MC) normalization, smoothing, and Tm analysis capabilities define what
    the scoped FD-method peak-detection task would run on
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Runtime platform in which the formulated sub-task would be executed, since
    SimpleDSFviewer runs in the Matlab workspace
- name: Matlab Runtime
  role: Alternative execution environment for SimpleDSFviewer if the analyst lacks
    a full Matlab workspace license
- name: manual expert review
  role: Fallback verification step when the section text provides no grounded metric,
    as was the case for this task card
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:2deafedad689199d0a44685f1d769cdb630ebe1c21e43cc1e5cb73da97e0f66f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - Matlab Runtime
  - SimpleDSFviewer
  - manual expert review
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# scientific_task_formulation

## Summary

Formulates a scoped, testable scientific sub-task from a research question, mapping the question onto concrete inputs (e.g. DSF melting curves from His-FGF10 wells with 0.625 μM heparin), a named analysis method (the FD method in SimpleDSFviewer), and a falsifiable success criterion (resolution of two distinct Tm values).

## When to use

Use when a research question must be converted into an executable analysis sub-task but the available section text lacks grounded methods steps, so the task scope, required tools, and validation criterion must be defined before analysis can proceed.

## When NOT to use

- The provided section text already contains grounded methods spans from which verbatim workflow steps and a validation metric can be selected — then execute the analysis rather than re-formulate the task
- The research question concerns a technique other than DSF thermal-stability analysis (e.g. sequencing or mass-spectrometry data), where the FD-method/DSF framing does not transfer
- The analysis goal is parameter tuning of SimpleDSFviewer itself rather than defining what a new sub-task should measure

## Inputs

- Research question card (e.g. how free, non-stabilized and heparin-bound His-FGF10 species manifest in DSF melting behavior at 0.625 μM heparin)
- Article-derived entity index (tools, workflow_steps, filter_steps, findings) with section-level evidence spans
- Scoped sub-task description from the source card (FD-method peak detection on His-FGF10 + 0.625 μM heparin DSF wells)

## Outputs

- Scoped, testable sub-task statement naming tool, method mode, inputs, and expected output (two distinct Tm values via the FD method)
- Explicit grounding-gap note listing which required methods evidence was absent from the provided section text
- Validation criterion statement (or a documented statement that no criterion can be grounded without methods text)

## How to apply

Decompose the research question into the minimal measurable quantity: here, the number of Tm species observable in DSF melting curves for His-FGF10 titrated with 0.625 μM heparin. Identify the analysis toolchain named by the article — SimpleDSFviewer, which performs melting curve (MC) normalization, smoothing, and Tm analysis — and specify the FD (first-derivative) peak-detection mode as the mechanism expected to resolve separate peaks for the non-exchanging heparin-bound and free FGF10 populations. Explicitly check grounding: in this task the supplied section text contained no methods content, so the workflow could only state the sub-task (FD-method peak detection on His-FGF10 + 0.625 μM heparin DSF wells yielding two Tm species) and record that no validation criterion could be anchored to a verbatim evidence span. The output of formulation is a scoped sub-task statement plus an honest gap note, not an executed analysis.

## Related tools

- **SimpleDSFviewer** (Target analysis environment named in the formulated sub-task; its melting curve (MC) normalization, smoothing, and Tm analysis capabilities define what the scoped FD-method peak-detection task would run on) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Runtime platform in which the formulated sub-task would be executed, since SimpleDSFviewer runs in the Matlab workspace)
- **Matlab Runtime** (Alternative execution environment for SimpleDSFviewer if the analyst lacks a full Matlab workspace license)
- **manual expert review** (Fallback verification step when the section text provides no grounded metric, as was the case for this task card)

## Evaluation signals

- The scoped sub-task names a concrete tool mode (FD-method peak detection), a concrete condition (His-FGF10 with 0.625 μM heparin), and a concrete expected outcome (two distinct Tm values) rather than generic 'analyze the data' language
- Each workflow step in the formulation either cites a verbatim evidence span or is explicitly marked as ungrounded, as this card correctly did ('No workflow steps can be grounded')
- The expected two-Tm outcome is traceable to the finding that the FD method resolves non-exchanging heparin-bound and free FGF10 species
- The validation criterion is either grounded in an extracted finding/metric or honestly reported as unstatable without methods text

## Limitations

- Task formulation cannot anchor a validation criterion when the supplied section text contains no methods content from which verbatim evidence spans could be selected (the failure mode recorded for task_008)
- The two-Tm interpretation assumes the heparin-bound and free FGF10 species are non-exchanging on the DSF timescale; if exchange occurred, a single intermediate Tm would appear and the scoped expectation would fail
- Formulation depends on the availability of the named tool (SimpleDSFviewer requires Matlab or Matlab Runtime)

## Evidence

- [other] Expected outcome of the scoped sub-task: two Tm species for His-FGF10 at 0.625 μM heparin: [evidence span withheld — anchor status: not_found]
