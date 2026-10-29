---
name: spectrum-graph-construction
description: Use when raw MS/MS peak lists must be converted into a de novo-style
  spectrum graph (nodes = peaks with mass/intensity, edges = mass differences) as
  the input representation for peptide identification by matching against a sequence
  database.
when_to_use_negative:
- The goal is straightforward database searching of high-quality spectra without modification
  or mutation characterization — Popitam's spectrum-graph approach is aimed at peptides
  with mutations or unexpected post-translational modifications
- The input is already a structured spectrum graph or a tool-ready feature representation
  — building it again duplicates existing preprocessing
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Popitam
  role: 'Consumes the constructed spectrum-graph representation: matches theoretical
    peptide sequences issued from a database against it, with the Ant Colony Optimization
    algorithm parsing the graph and producing peptides ranked by a correlation score'
  repo: github.com/chernan/popitam
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1002/pmic.200300402
    title: 'Popitam: Towards new heuristic strategies to improve protein identification
      from tandem mass spectrometry data'
  source_content_hash: sha256:3ba2c1a5569ea6927e0db42bbe79d11f20aab1cf22bb6c3b5bec91fa749ed8c7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Popitam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# spectrum-graph-construction

## Summary

Construct a structured spectrum-graph representation of a source MS/MS spectrum — one node per fragment peak (mass and intensity) with directed edges annotated by mass differences — so that theoretical peptide sequences from a database can be matched against it, as done by Popitam prior to Ant Colony Optimization graph parsing.

## When to use

Use when raw MS/MS peak lists must be converted into a de novo-style spectrum graph (nodes = peaks with mass/intensity, edges = mass differences) as the input representation for peptide identification by matching against a sequence database.

## When NOT to use

- The goal is straightforward database searching of high-quality spectra without modification or mutation characterization — Popitam's spectrum-graph approach is aimed at peptides with mutations or unexpected post-translational modifications
- The input is already a structured spectrum graph or a tool-ready feature representation — building it again duplicates existing preprocessing

## Inputs

- MS/MS fragment peak list (m/z and intensity per peak; repository sample spectra or PRIDE/MassIVE peak lists)
- Per-spectrum grouping of peaks (one graph per source MS/MS spectrum)

## Outputs

- One structured spectrum graph file per input spectrum (JSON or TSV) with node fields mass and intensity and edge field mass difference
- Graph files ready to be consumed by Popitam for theoretical peptide sequence matching

## How to apply

Load the source MS/MS peak list (e.g. sample spectra shipped in github.com/chernan/popitam, or a public PRIDE/MassIVE peak list) keeping m/z and intensity for each fragment peak, since these are the node attributes. Create one node per peak storing its mass and intensity. Connect pairs of peaks with directed edges, annotating each edge with the mass difference between the two peaks — mirroring the spectrum-graph structure Popitam uses, which the authors note is 'similar to the spectrum graphs commonly used by de novo sequencing software'. Write one structured graph file per input spectrum (JSON or TSV with explicit mass, intensity, and mass-difference fields). The resulting graph is the representation against which Popitam matches theoretical peptide sequences before the Ant Colony Optimization algorithm parses it to emphasize relevant sections per sequence.

## Related tools

- **Popitam** (Consumes the constructed spectrum-graph representation: matches theoretical peptide sequences issued from a database against it, with the Ant Colony Optimization algorithm parsing the graph and producing peptides ranked by a correlation score) — github.com/chernan/popitam

## Evaluation signals

- Exactly one graph file exists per input MS/MS spectrum
- Every graph file parses in the expected JSON/TSV format
- Every node in each graph carries explicit mass (m/z) and intensity fields
- Every directed edge carries a mass-difference field equal to the m/z difference of the two connected peaks
- For the full identification run, the resulting ranked output contains a correlation-score column and, ideally, matches the repository's shipped reference results

## Limitations

- Popitam maintenance status: the README states no new development, maintenance, or support is performed — the improved version by Florent Glück is archived on GitHub for posterity
- Graph parsing is described as 'a highly combinatorial task', which is why the bio-inspired Ant Colony Optimization heuristic is required; naive parsing of large graphs may be intractable
- The representation inherits the quality limitations of low-quality MS/MS data, a scenario the method explicitly targets but which affects graph content

## Evidence
