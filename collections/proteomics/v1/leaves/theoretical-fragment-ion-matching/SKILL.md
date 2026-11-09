---
name: theoretical-fragment-ion-matching
description: Use when candidate theoretical peptides from a sequence database must
  be scored against an MS/MS spectrum — especially for peptides carrying mutations
  or unexpected post-translational modifications.
when_to_use_negative:
- The goal is de novo sequencing without a peptide database — Popitam's identification
  process matches theoretical peptide sequences issued from a database, so it is the
  wrong choice when no candidate sequences exist
- You only need the spectrum graph construction or the ACO graph-parsing step and
  have no theoretical sequences to score yet — use those upstream skills instead
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0092
tools:
- name: Popitam
  role: Performs the matching of theoretical peptide sequences from the database against
    the structured MS/MS spectrum representation and produces the list of peptides
    ranked by correlation score
  repo: github.com/chernan/popitam
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.200300402
    title: 'Popitam: Towards new heuristic strategies to improve protein identification
      from tandem mass spectrometry data'
  source_content_hash: sha256:d251c4064072b87abb62800fa36b870b2dd4305611c6d3083acf1348635f96e7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Popitam
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Theoretical fragment ion matching with correlation-score ranking

## Summary

Matches theoretical peptide sequences issued from a peptide database search against a structured representation (spectrum graph) of the source MS/MS spectrum, computing a correlation score of how well ACO-parsed graph sections align with each theoretical peptide, and outputs peptides ranked by that score.

## When to use

Use when candidate theoretical peptides from a sequence database must be scored against an MS/MS spectrum — especially for peptides carrying mutations or unexpected post-translational modifications.

## When NOT to use

- The goal is de novo sequencing without a peptide database — Popitam's identification process matches theoretical peptide sequences issued from a database, so it is the wrong choice when no candidate sequences exist
- You only need the spectrum graph construction or the ACO graph-parsing step and have no theoretical sequences to score yet — use those upstream skills instead

## Inputs

- Pheromone-emphasized sections of the spectrum graph produced by the ACO parsing step for each candidate theoretical sequence
- Candidate theoretical peptide sequences issued from the peptide database search
- Structured representation (spectrum graph) of the source MS/MS spectrum

## Outputs

- List of peptides ranked in descending order by correlation score (e.g. ranked_peptides.csv with peptide identifier/sequence and correlation score fields)

## How to apply

First build a spectrum-graph representation of the source MS/MS spectrum (similar to spectrum graphs used by de novo sequencing software), then parse the graph with an Ant Colony Optimization algorithm to emphasize the sections relevant to each theoretical sequence from the database. Load the candidate theoretical peptide sequences issued from the peptide database search and match each one against this structured representation. Compute a correlation score quantifying how well each theoretical peptide's emphasized graph sections align with the spectrum structure — Popitam handles this combinatorial parsing via ACO rather than exhaustive enumeration, which is what makes it robust to modifications/mutations, large databases, and low quality data. Finally sort the scored peptides in descending order of correlation score and write the ranked list (peptide identifier/sequence plus score) to an output such as ranked_peptides.csv. This matching-plus-ranking step is the end product of Popitam's identification process after graph parsing.

## Related tools

- **Popitam** (Performs the matching of theoretical peptide sequences from the database against the structured MS/MS spectrum representation and produces the list of peptides ranked by correlation score) — github.com/chernan/popitam

## Evaluation signals

- ranked_peptides.csv exists after the run
- Every peptide row in ranked_peptides.csv has a correlation score field populated
- The correlation score column is ordered in non-increasing (descending) order
- Each output peptide identifier/sequence corresponds to a theoretical peptide that was in the candidate database-search input
- Scores reflect alignment of ACO-parsed graph sections with the spectrum structure — top-ranked peptides should correspond to sequences whose emphasized sections coincide with observed spectrum peaks

## Limitations

- Popitam is archived software: the maintainer states she 'will not make any new development nor will I perform any maintenance/support on the tool'
- The parsing of the spectrum graph can be a highly combinatorial task, mitigated by the Ant Colony Optimization algorithm
- Method targets peptides with mutations or unexpected post-translational modifications; characterization quality depends on the source MS/MS data (promising for low quality data per the abstract, but not guaranteed)

## Evidence
