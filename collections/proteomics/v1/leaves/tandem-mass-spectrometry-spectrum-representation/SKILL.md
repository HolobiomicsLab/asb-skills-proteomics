---
name: tandem-mass-spectrometry-spectrum-representation
description: Use when you need to convert raw tandem mass spectrometry (MS/MS) peak
  lists into a de novo-style spectrum graph that a sequence-database search or ACO-based
  parser can traverse, especially for peptides with mutations or unexpected PTMs.
when_to_use_negative:
- When the goal is straightforward unmodified peptide identification where a standard
  sequence-database search tool suffices — Popitam's design target is peptides with
  mutations or unexpected post-translational modifications
- When the input is already a parsed spectrum graph or a scored peptide ranking —
  this skill covers the spectrum-to-graph representation step only
- When only de novo sequencing (no sequence database) is required — this skill depends
  on theoretical sequences issued from a database
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0092
tools:
- name: Popitam
  role: Builds the spectrum-graph representation of the source MS/MS spectrum and
    parses it with an Ant Colony Optimization algorithm to match theoretical peptide
    sequences and rank peptides by correlation score
  repo: https://github.com/chernan/popitam
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.200300402
    title: 'Popitam: Towards new heuristic strategies to improve protein identification
      from tandem mass spectrometry data'
  source_content_hash: sha256:4a34f314fac423c7c501d92f2610a9fab3e108a10dfacde24318b51525dd7177
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chernan/popitam
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

# tandem-mass-spectrometry-spectrum-representation

## Summary

Builds a structured spectrum-graph representation of a source MS/MS spectrum, in the style of the spectrum graphs used by de novo sequencing software, so that theoretical peptide sequences from a database can be matched against it. This graph is the substrate Popitam parses with an Ant Colony Optimization algorithm to produce peptides ranked by a correlation score.

## When to use

Use when you need to convert raw tandem mass spectrometry (MS/MS) peak lists into a de novo-style spectrum graph that a sequence-database search or ACO-based parser can traverse, especially for peptides with mutations or unexpected PTMs.

## When NOT to use

- When the goal is straightforward unmodified peptide identification where a standard sequence-database search tool suffices — Popitam's design target is peptides with mutations or unexpected post-translational modifications
- When the input is already a parsed spectrum graph or a scored peptide ranking — this skill covers the spectrum-to-graph representation step only
- When only de novo sequencing (no sequence database) is required — this skill depends on theoretical sequences issued from a database

## Inputs

- Source MS/MS spectrum (tandem mass spectrometry peak list)
- Theoretical peptide sequences from a peptide sequence database (e.g., tryptic peptides from a public UniProt FASTA)

## Outputs

- Structured spectrum-graph representation of the source MS/MS spectrum
- List of peptides ranked by a correlation score (with per-sequence ACO-selected graph paths)

## How to apply

First take the source MS/MS spectrum (peak list from tandem mass spectrometry data) and build a structured representation similar to the spectrum graphs commonly used by de novo sequencing software, where nodes correspond to fragment ions (e.g., b/y ion masses) and edges encode mass differences between peaks. Then load theoretical peptide sequences issued from a sequence database (e.g., tryptic peptides from a UniProt FASTA) and match them against this graph rather than against raw spectra directly. Because the graph can be a highly combinatorial structure, parse it with Popitam's bio-inspired Ant Colony Optimization algorithm: artificial ants explore candidate paths, and pheromone trails are updated so that graph sections relevant to each theoretical sequence are progressively emphasized. Collect, for each theoretical sequence, the selected graph path and its correlation score, yielding a list of peptides ranked by that score. Note that the improved Popitam version by Florent Glück runs 1.4 times faster than the original software, which matters when searching large databases.

## Related tools

- **Popitam** (Builds the spectrum-graph representation of the source MS/MS spectrum and parses it with an Ant Colony Optimization algorithm to match theoretical peptide sequences and rank peptides by correlation score) — https://github.com/chernan/popitam

## Evaluation signals

- The spectrum-graph representation is structurally similar to spectrum graphs used by de novo sequencing software (nodes/edges encode fragment-ion mass differences)
- For each theoretical sequence from the database, the ACO parse yields a selected graph path with a numeric correlation score
- The final output is a list of peptides ranked by correlation score, and scores are consistent across ranked peptides
- Running the improved Popitam version should be about 1.4 times more efficient (faster) than the original software on the same input
- Peptides carrying mutations or unexpected post-translational modifications are still matched via emphasized graph sections rather than dropped entirely

## Limitations

- The original software is unmaintained: the README states the migrator 'will not make any new development nor will I perform any maintenance/support on the tool'
- The graph parsing is a highly combinatorial task, which is why a heuristic (ACO) rather than exhaustive search is required; results depend on pheromone-driven convergence
- No discussion-section content is available from the article to detail failure modes or edge cases

## Evidence
