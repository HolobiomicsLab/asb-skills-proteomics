---
name: pheromone-trail-update-design
description: Use when a highly combinatorial spectrum graph built from MS/MS peak
  data must be parsed against theoretical peptide sequences, and per-sequence candidate
  paths need to be emphasized via ACO pheromone updates before ranking peptides.
when_to_use_negative:
- The input is raw MS/MS peak lists that have not yet been converted into the structured
  spectrum-graph representation — build the spectrum graph first.
- The goal is de novo sequencing without a peptide sequence database, since this skill
  matches theoretical sequences issued from a database against the graph.
- Theoretical peptide sequences cannot be provided, so there is no basis for per-sequence
  pheromone emphasis and correlation-score ranking.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: Popitam
  role: Implements the spectrum-graph parsing by the Ant Colony Optimization algorithm,
    including pheromone-trail updates and correlation-score ranking; the improved
    version by Florent Glück is 1.4x more efficient than the original.
  repo: https://github.com/chernan/popitam
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.200300402
    title: 'Popitam: Towards new heuristic strategies to improve protein identification
      from tandem mass spectrometry data'
  source_content_hash: sha256:3124a0350ba0c3ab3bf749c05f9e00820e73e45362d9fe86bdca3f5b0200cbdf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chernan/popitam
  tools:
  - Popitam
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# pheromone-trail-update-design

## Summary

Designs the Ant Colony Optimization pheromone-update cycle used by Popitam to parse a combinatorial MS/MS spectrum graph, progressively emphasizing graph sections that match each theoretical peptide sequence from a database.

## When to use

Use when a highly combinatorial spectrum graph built from MS/MS peak data must be parsed against theoretical peptide sequences, and per-sequence candidate paths need to be emphasized via ACO pheromone updates before ranking peptides.

## When NOT to use

- The input is raw MS/MS peak lists that have not yet been converted into the structured spectrum-graph representation — build the spectrum graph first.
- The goal is de novo sequencing without a peptide sequence database, since this skill matches theoretical sequences issued from a database against the graph.
- Theoretical peptide sequences cannot be provided, so there is no basis for per-sequence pheromone emphasis and correlation-score ranking.

## Inputs

- Structured spectrum-graph representation of a source MS/MS spectrum (built as in de novo sequencing spectrum graphs)
- Theoretical peptide sequences from a peptide sequence database (e.g., tryptic peptides from a public UniProt FASTA)

## Outputs

- Per-theoretical-sequence ACO-selected graph paths with pheromone fields (e.g., aco_selected_paths.csv)
- Per-sequence numeric path (correlation) scores feeding a list of peptides ranked by correlation score

## How to apply

Build a structured spectrum-graph representation of the source MS/MS spectrum, in the style of spectrum graphs used by de novo sequencing software, and load theoretical peptide sequences issued from a peptide sequence database (e.g., tryptic peptides from a UniProt FASTA). Initialize the Ant Colony Optimization search over the highly combinatorial graph, distributing artificial ants to explore candidate paths for each theoretical sequence. Iterate the ACO cycle, updating pheromone trails so that graph sections matching each theoretical peptide sequence are progressively emphasized — the rationale is that heuristic pheromone reinforcement makes the combinatorial parsing tractable while remaining robust to mutations and unexpected post-translational modifications. Collect, for each theoretical sequence, the ACO-selected graph path and its per-sequence path (correlation) score, then write paths and scores with explicit path, pheromone and score fields for downstream peptide ranking by correlation score.

## Related tools

- **Popitam** (Implements the spectrum-graph parsing by the Ant Colony Optimization algorithm, including pheromone-trail updates and correlation-score ranking; the improved version by Florent Glück is 1.4x more efficient than the original.) — https://github.com/chernan/popitam

## Evaluation signals

- aco_selected_paths.csv exists and parses as a table with the expected path, pheromone and score fields.
- Every theoretical sequence row carries a numeric path score consistent with Popitam's correlation-score output.
- The ACO execution completes without error (script_runs check) and emits one selected path per theoretical sequence.
- Final peptide candidates can be ordered into a list of peptides ranked by correlation score.
- Paths selected after pheromone updates correspond to graph sections matching the target theoretical peptide sequences rather than random graph walks.

## Limitations

- The Celine Hernandez-migrated repository is archived for preservation only — no new development, maintenance or support is provided.
- The spectrum graph can be a highly combinatorial structure, so results depend on the ACO heuristic rather than exhaustive search.
- Popitam was designed for peptides with mutations or unexpected post-translational modifications; it is not a generic de novo sequencing tool.

## Evidence
