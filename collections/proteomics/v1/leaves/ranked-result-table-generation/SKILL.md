---
name: ranked-result-table-generation
description: Use when you have candidate theoretical peptide sequences from a database
  search plus a parsed spectrum graph of an MS/MS spectrum, and need to produce a
  final peptide ranking table by correlation score.
when_to_use_negative:
- You only need de novo sequencing output rather than a database-search peptide ranking
  — Popitam's design goal is characterization of peptides with mutations or unexpected
  post-translational modifications via database search.
- The spectrum graph has not yet been built/parsed (no ACO-emphasized graph sections
  exist) — the correlation score depends on the parsed graph representation.
- No candidate theoretical peptide sequences are available from a database search
  to match against the spectrum.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0092
tools:
- name: Popitam
  role: 'Performs the identification process: matches theoretical peptide sequences
    against the spectrum graph and produces the list of peptides ranked by correlation
    score.'
  repo: https://github.com/chernan/popitam
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.200300402
    title: 'Popitam: Towards new heuristic strategies to improve protein identification
      from tandem mass spectrometry data'
  source_content_hash: sha256:3bcbde60eb1f90dea9ebd2fcfa5faba64bacab5f5b15fb36e168673f71c87bdb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chernan/popitam
  tools:
  - Popitam
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# ranked-result-table-generation

## Summary

Generates a final ranked result table for peptide identification by scoring each candidate theoretical peptide against a structured MS/MS spectrum representation (spectrum graph parsed via Ant Colony Optimization) and sorting by correlation score, as done in Popitam.

## When to use

Use when you have candidate theoretical peptide sequences from a database search plus a parsed spectrum graph of an MS/MS spectrum, and need to produce a final peptide ranking table by correlation score.

## When NOT to use

- You only need de novo sequencing output rather than a database-search peptide ranking — Popitam's design goal is characterization of peptides with mutations or unexpected post-translational modifications via database search.
- The spectrum graph has not yet been built/parsed (no ACO-emphasized graph sections exist) — the correlation score depends on the parsed graph representation.
- No candidate theoretical peptide sequences are available from a database search to match against the spectrum.

## Inputs

- ACO parsing results: pheromone-emphasized sections of the spectrum graph produced for each candidate theoretical sequence
- Candidate theoretical peptide sequences from the peptide database search
- Structured representation (spectrum graph) of the source MS/MS spectrum

## Outputs

- ranked_peptides.csv: list of peptides (identifier/sequence) with correlation score, sorted in descending order of correlation score

## How to apply

First build the spectrum graph of the source MS/MS spectrum and parse it with the Ant Colony Optimization algorithm so that graph sections relevant to each theoretical sequence are emphasized. Match each theoretical peptide sequence from the database search against this structured representation, computing a correlation score that quantifies how well each peptide's emphasized graph sections align with the spectrum structure. Sort the scored peptides in descending order of correlation score and write the ranked list (peptide identifier/sequence plus correlation score) to ranked_peptides.csv. In Popitam, this ranking step is the end product of the identification process after ACO-based graph parsing.

## Related tools

- **Popitam** (Performs the identification process: matches theoretical peptide sequences against the spectrum graph and produces the list of peptides ranked by correlation score.) — https://github.com/chernan/popitam

## Evaluation signals

- ranked_peptides.csv exists and contains a correlation score field for every peptide.
- The correlation score column is ordered in non-increasing (descending) order.
- Every candidate theoretical peptide from the database search appears in the output (one scored row per peptide).
- Top-ranked peptides correspond to theoretical sequences whose emphasized graph sections align best with the MS/MS spectrum structure.
- The output schema includes peptide identifier/sequence alongside the correlation score.

## Limitations

- The original tool is archived: the maintainer states she 'will not make any new development nor will I perform any maintenance/support on the tool'.
- Graph parsing is a highly combinatorial task, handled heuristically by ACO, so the ranking depends on heuristic parsing quality.
- Designed for peptides with mutations or unexpected post-translational modifications in MS/MS data; not a general-purpose scoring framework.

## Evidence
