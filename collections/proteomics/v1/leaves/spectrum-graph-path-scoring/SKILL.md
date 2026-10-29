---
name: spectrum-graph-path-scoring
description: Use when characterizing peptides with mutations or unexpected post-translational
  modifications from MS/MS data by searching a peptide sequence database, especially
  with low-quality spectra or large databases.
when_to_use_negative:
- The goal is purely de novo sequencing rather than matching theoretical sequences
  from a peptide database — Popitam's graph parsing is driven by database sequences.
- The input is a processed identification result (e.g., an existing peptide score
  table) rather than raw MS/MS spectra plus a sequence database.
- The target problem requires no tolerance for mutations or unexpected post-translational
  modifications and a simpler exhaustive search would suffice.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Popitam
  role: Implements the spectrum-graph construction, ACO-based graph parsing, and correlation-score
    ranking of candidate peptides against a sequence database
  repo: github.com/chernan/popitam
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1002/pmic.200300402
    title: 'Popitam: Towards new heuristic strategies to improve protein identification
      from tandem mass spectrometry data'
  source_content_hash: sha256:2477f13569c26b493840cf3a3523ce4598514d641fd878fe8fb194ba6c52098b
schema_version: 0.2.0
metadata:
  license_tier: unknown
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
yes_rate: 0.9090909090909091
n_claims: 11
---

# spectrum-graph-path-scoring

## Summary

Popitam's identification pipeline: build a spectrum-graph representation of an MS/MS spectrum, parse its combinatorial paths with an Ant Colony Optimization algorithm to emphasize sections relevant to each theoretical peptide sequence from a database, and rank the resulting peptides by a correlation score.

## When to use

Use when characterizing peptides with mutations or unexpected post-translational modifications from MS/MS data by searching a peptide sequence database, especially with low-quality spectra or large databases.

## When NOT to use

- The goal is purely de novo sequencing rather than matching theoretical sequences from a peptide database — Popitam's graph parsing is driven by database sequences.
- The input is a processed identification result (e.g., an existing peptide score table) rather than raw MS/MS spectra plus a sequence database.
- The target problem requires no tolerance for mutations or unexpected post-translational modifications and a simpler exhaustive search would suffice.

## Inputs

- Tandem mass spectrometry (MS/MS) source spectrum
- Theoretical peptide sequences issued from a peptide sequence database (e.g., tryptic peptides from a UniProt FASTA)
- Structured spectrum-graph representation of the MS/MS spectrum

## Outputs

- ACO-selected graph path and pheromone/score information per theoretical sequence (e.g., aco_selected_paths.csv)
- List of candidate peptides ranked in descending order by correlation score (e.g., ranked_peptides.csv)

## How to apply

First build a structured representation (spectrum graph) of the source MS/MS spectrum, similar to the spectrum graphs used by de novo sequencing software. Then, for each theoretical peptide sequence issued from the peptide sequence database, run the Ant Colony Optimization algorithm over the highly combinatorial graph, distributing artificial ants to explore candidate paths and updating pheromone trails so graph sections matching each theoretical sequence are progressively emphasized. Collect each per-sequence ACO-selected path and compute the correlation score quantifying how well its emphasized graph sections align with the spectrum structure. Finally, sort the peptides in descending order of correlation score to obtain the ranked identification list. The heuristics are designed to remain tractable for handling modifications/mutations, large databases, and low quality data.

## Related tools

- **Popitam** (Implements the spectrum-graph construction, ACO-based graph parsing, and correlation-score ranking of candidate peptides against a sequence database) — github.com/chernan/popitam

## Evaluation signals

- aco_selected_paths.csv exists and parses as a table with explicit path, pheromone and score fields for every theoretical sequence row.
- Every theoretical sequence row in aco_selected_paths.csv carries a numeric path score consistent with Popitam's correlation-score output.
- ranked_peptides.csv contains a correlation score field for every candidate peptide.
- The correlation-score column in ranked_peptides.csv is ordered in non-increasing (descending) order.
- Script-runs check confirms the ACO execution completed over the spectrum graph without failure.

## Limitations

- The ACO parsing addresses a highly combinatorial task, so results depend on heuristic pheromone-trail updates rather than exhaustive enumeration of graph paths.
- The tool is archived: the README states no new development nor maintenance/support will be performed by the original author.
- No discussion-section content was available in the article, limiting documented caveats.

## Evidence
