---
name: peptide-spectrum-matching
description: Use when candidate theoretical peptides from a database search must be
  scored against an MS/MS spectrum graph to rank identification hypotheses, especially
  with mutations or unexpected post-translational modifications.
when_to_use_negative:
- When the input is a raw MS/MS spectrum that has not yet been converted into a spectrum
  graph, since the graph-building step must precede matching
- When no candidate theoretical peptide sequences from a database search are available,
  as the matching operates on database-issued candidates
- When de novo sequencing without a peptide database is the goal, since this skill
  scores database-issued theoretical peptides
edam_operation: http://edamontology.org/operation_3647
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Popitam
  role: Runs the ACO-based spectrum-graph parsing and matches theoretical peptide
    sequences from the database against it, producing peptides ranked by correlation
    score
  repo: github.com/chernan/popitam
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.200300402
    title: 'Popitam: Towards new heuristic strategies to improve protein identification
      from tandem mass spectrometry data'
  source_content_hash: sha256:b7aaef04c78850353599c28c11ee5a626dab4d29e337acf94ac45e6211f0f23a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Popitam
  edam_operation: http://edamontology.org/operation_3647
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# peptide-spectrum-matching

## Summary

Match theoretical peptide sequences issued from a peptide database against a structured representation of the source MS/MS spectrum (spectrum graph), computing a correlation score for each match, to produce a ranked list of candidate peptides as in Popitam's identification process.

## When to use

Use when candidate theoretical peptides from a database search must be scored against an MS/MS spectrum graph to rank identification hypotheses, especially with mutations or unexpected post-translational modifications.

## When NOT to use

- When the input is a raw MS/MS spectrum that has not yet been converted into a spectrum graph, since the graph-building step must precede matching
- When no candidate theoretical peptide sequences from a database search are available, as the matching operates on database-issued candidates
- When de novo sequencing without a peptide database is the goal, since this skill scores database-issued theoretical peptides

## Inputs

- Pheromone-emphasized sections of the spectrum graph produced by ACO parsing for each candidate theoretical sequence (output of the ACO graph-parsing step)
- Candidate theoretical peptide sequences issued from the peptide database search
- Structured representation (spectrum graph) of the source MS/MS spectrum

## Outputs

- A ranked list of peptides (peptide identifier/sequence, correlation score) sorted in descending correlation score, e.g. ranked_peptides.csv

## How to apply

First build a structured representation (spectrum graph) of the source MS/MS spectrum, then parse the graph with the Ant Colony Optimization algorithm to emphasize sections relevant to each theoretical sequence from the database. Next, load the candidate theoretical peptide sequences issued from the peptide database search and match each one against the emphasized graph structure. For every theoretical peptide, compute a correlation score quantifying how well its emphasized graph sections align with the spectrum structure. Finally, sort the scored peptides in descending correlation score and write the ranked list (peptide identifier/sequence, correlation score) to a CSV file such as ranked_peptides.csv. The correlation score, rather than raw peak matches, is the ranking criterion because the ACO-parsed graph representation handles modifications and mutations within the combinatorial path space.

## Related tools

- **Popitam** (Runs the ACO-based spectrum-graph parsing and matches theoretical peptide sequences from the database against it, producing peptides ranked by correlation score) — github.com/chernan/popitam

## Evaluation signals

- ranked_peptides.csv exists and contains a correlation score field for every peptide
- The correlation score column in ranked_peptides.csv is ordered in non-increasing (descending) order
- Every candidate theoretical peptide issued from the database search appears in the ranked output with a peptide identifier/sequence
- Each correlation score reflects alignment between the theoretical peptide's ACO-emphasized graph sections and the spectrum graph structure

## Limitations

- The original Popitam tool is archived; the maintainer states no new development, maintenance, or support will be provided (an improved version by Florent Glück is ~1.4x more efficient)
- The graph parsing is a highly combinatorial task, hence the reliance on a bio-inspired heuristic (Ant Colony Optimization) rather than exhaustive path enumeration
- No discussion section content was available in the extracted article text

## Evidence
