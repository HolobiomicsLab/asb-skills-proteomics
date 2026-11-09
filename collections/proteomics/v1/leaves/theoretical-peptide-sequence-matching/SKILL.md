---
name: theoretical-peptide-sequence-matching
description: Use when identifying peptides from tandem mass spectrometry (MS/MS) data
  by database search, especially when mutations or unexpected PTMs are expected or
  the spectrum is of low quality.
when_to_use_negative:
- The input is raw MS/MS spectra that have not yet been converted into a spectrum-graph
  representation — build the graph representation first.
- The goal is purely de novo sequencing without a peptide sequence database — Popitam's
  matching is database-search based, though its graph representation resembles de
  novo spectrum graphs.
- No candidate theoretical sequences are available; the ACO parsing emphasizes graph
  sections relative to each theoretical sequence and cannot run without them.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Popitam
  role: Performs the ACO parsing of the spectrum graph against theoretical peptide
    sequences and ranks peptides by correlation score; characterizes peptides with
    mutations or unexpected PTMs from MS/MS data
  repo: https://github.com/chernan/popitam
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.200300402
    title: 'Popitam: Towards new heuristic strategies to improve protein identification
      from tandem mass spectrometry data'
  source_content_hash: sha256:a198c8d07c9d215a11010bd1c7a1d5ef2895c887b320dafc1e2792a2bdb88285
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chernan/popitam
  tools:
  - Popitam
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# theoretical-peptide-sequence-matching

## Summary

Matches theoretical peptide sequences from a sequence database against a spectrum-graph representation of an MS/MS spectrum using Popitam's Ant Colony Optimization parsing, producing peptides ranked by a correlation score. This handles the combinatorial complexity of the graph, including peptides carrying mutations or unexpected post-translational modifications.

## When to use

Use when identifying peptides from tandem mass spectrometry (MS/MS) data by database search, especially when mutations or unexpected PTMs are expected or the spectrum is of low quality.

## When NOT to use

- The input is raw MS/MS spectra that have not yet been converted into a spectrum-graph representation — build the graph representation first.
- The goal is purely de novo sequencing without a peptide sequence database — Popitam's matching is database-search based, though its graph representation resembles de novo spectrum graphs.
- No candidate theoretical sequences are available; the ACO parsing emphasizes graph sections relative to each theoretical sequence and cannot run without them.

## Inputs

- Structured spectrum-graph representation of the source MS/MS spectrum (as produced by Popitam, de novo-style spectrum graph)
- Theoretical peptide sequences from a peptide sequence database (e.g., tryptic peptides from a public UniProt FASTA)

## Outputs

- aco_selected_paths.csv: per-theoretical-sequence selected graph paths with explicit path, pheromone and score fields
- List of peptides ranked by a correlation score

## How to apply

First build a structured spectrum-graph representation of the source MS/MS spectrum, in the style of the spectrum graphs used by de novo sequencing software. Then load theoretical peptide sequences issued from a peptide sequence database (e.g., tryptic peptides derived from a public UniProt FASTA) and match them against the graph. Because the graph is highly combinatorial, parse it with Popitam's bio-inspired Ant Colony Optimization algorithm: artificial ants explore candidate paths, and pheromone trails are updated so that graph sections relevant to each theoretical sequence are progressively emphasized. Collect, for each theoretical sequence, the ACO-selected path and its correlation score, then rank peptides by that score. Output the selected paths, pheromone and score fields (e.g., aco_selected_paths.csv) for downstream ranking and validation.

## Related tools

- **Popitam** (Performs the ACO parsing of the spectrum graph against theoretical peptide sequences and ranks peptides by correlation score; characterizes peptides with mutations or unexpected PTMs from MS/MS data) — https://github.com/chernan/popitam

## Evaluation signals

- aco_selected_paths.csv exists and parses as a table with the expected path, pheromone and score fields.
- Every theoretical sequence row carries a numeric path score consistent with Popitam's correlation-score output.
- A script_runs check passes on the ACO execution over the combinatorial spectrum graph.
- Output peptides are ranked (ordered) by their correlation score, as Popitam's identification process produces.
- The selected paths correspond to graph sections emphasized for each theoretical sequence, as reflected in pheromone-trail updates during ACO iterations.

## Limitations

- No new development or maintenance/support is provided for the tool; the codebase is archived (migrated from Google Code to GitHub for archiving).
- The spectrum graph is a highly combinatorial structure; the ACO heuristic parsing may not exhaustively explore all candidate paths.
- The identification process targets peptides with mutations or unexpected post-translational modifications; the matching depends on the quality of the MS/MS spectrum (the paper reports promising properties for handling low quality data, but quality still matters).

## Evidence
