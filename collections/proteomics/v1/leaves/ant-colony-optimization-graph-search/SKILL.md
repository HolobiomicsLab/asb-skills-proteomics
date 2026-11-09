---
name: ant-colony-optimization-graph-search
description: Use when peptide identification from tandem mass spectrometry data must
  match theoretical database peptide sequences against a combinatorial spectrum graph,
  especially with mutations or unexpected post-translational modifications, large
  databases, or low-quality spectra.
when_to_use_negative:
- The task is pure de novo sequencing without a peptide sequence database to match
  against — Popitam's process consists of matching theoretical sequences issued from
  a database against the spectrum representation
- Input is already a ranked peptide identification list or feature table rather than
  raw MS/MS spectra needing spectrum-graph construction
- No compiled Popitam executable or build environment is available and no spectrum-graph
  representation of the MS/MS spectrum can be produced
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Popitam
  role: Implements the spectrum-graph construction, Ant Colony Optimization parsing
    of the graph against theoretical peptide sequences, and correlation-score-ranked
    peptide output
  repo: https://github.com/chernan/popitam
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1002/pmic.200300402
    title: 'Popitam: Towards new heuristic strategies to improve protein identification
      from tandem mass spectrometry data'
  source_content_hash: sha256:a57f6ac834da969f4506abf034956d92c361880b53717b0f04aab8ef3613ac65
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chernan/popitam
  tools:
  - Popitam
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# ant-colony-optimization-graph-search

## Summary

Applies a bio-inspired Ant Colony Optimization algorithm to parse a highly combinatorial spectrum-graph representation of an MS/MS spectrum, emphasizing graph sections relevant to each theoretical peptide sequence from a database and yielding peptides ranked by a correlation score (as implemented in Popitam).

## When to use

Use when peptide identification from tandem mass spectrometry data must match theoretical database peptide sequences against a combinatorial spectrum graph, especially with mutations or unexpected post-translational modifications, large databases, or low-quality spectra.

## When NOT to use

- The task is pure de novo sequencing without a peptide sequence database to match against — Popitam's process consists of matching theoretical sequences issued from a database against the spectrum representation
- Input is already a ranked peptide identification list or feature table rather than raw MS/MS spectra needing spectrum-graph construction
- No compiled Popitam executable or build environment is available and no spectrum-graph representation of the MS/MS spectrum can be produced

## Inputs

- Source MS/MS spectrum (e.g., the repository's shipped example MS/MS spectrum input)
- Theoretical peptide sequences from a peptide sequence database (e.g., tryptic peptides from a public UniProt FASTA)

## Outputs

- ACO-selected graph paths with pheromone and score fields per theoretical sequence (aco_selected_paths.csv)
- List of peptides ranked by a correlation score (popitam_ranked_results.txt)

## How to apply

First build a structured spectrum-graph representation of the source MS/MS spectrum, in the style of the spectrum graphs used by de novo sequencing software. Load theoretical peptide sequences issued from a peptide sequence database (e.g., tryptic peptides from a public UniProt FASTA). Initialize the Ant Colony Optimization search over the highly combinatorial spectrum graph, distributing artificial ants to explore candidate paths for each theoretical sequence; iterate the ACO cycle, updating pheromone trails so that graph sections matching each theoretical peptide sequence are progressively emphasized. For each theoretical sequence, collect the ACO-selected graph path and its per-sequence path (correlation) score, and rank the resulting peptide list by this correlation score. The Popitam tool (improved version by Florent Glück, 1.4x more efficient) implements this; it can be compiled from the chernan/popitam repository and run on shipped example MS/MS spectrum input.

## Related tools

- **Popitam** (Implements the spectrum-graph construction, Ant Colony Optimization parsing of the graph against theoretical peptide sequences, and correlation-score-ranked peptide output) — https://github.com/chernan/popitam

## Examples

```
python scripts/run_aco.py --spectrum example.mgf --fasta uniprot_tryptic.fasta --out aco_selected_paths.csv
```

## Evaluation signals

- aco_selected_paths.csv exists, parses as a table with explicit path, pheromone and score fields
- Every theoretical sequence row in aco_selected_paths.csv carries a numeric path score consistent with Popitam's correlation-score output
- popitam_ranked_results.txt exists and contains a correlation-score column
- The run script completed successfully (script_runs check on the ACO execution)
- Results match the repository's shipped reference results for the example MS/MS spectrum input

## Limitations

- The GitHub repository is archived for maintenance: the maintainer states 'I will not make any new development nor will I perform any maintenance/support on the tool'
- The improved version provides a 1.4x efficiency gain over the original software but exact runtime on very large combinatorial graphs may still be substantial
- Handling of mutations or unexpected post-translational modifications is the method's design goal but is heuristic (ACO-based), not exhaustive

## Evidence

- [abstract] The parsing of the graph, which can be a highly combinatorial task, is performed by a bio-inspired algorithm called Ant Colony Optimization algorithm.: "The parsing of the graph, which can be a highly combinatorial task, is performed by a bio-inspired algorithm called Ant Colony Optimization algorithm."
- [abstract] It consists of matching theoretical peptide sequences issued from a database with a structured representation of the source MS/MS spectrum.: "It consists of matching theoretical peptide sequences issued from a database with a structured representation of the source MS/MS spectrum."
- [abstract] The representation is similar to the spectrum graphs commonly used by de novo sequencing software.: "The representation is similar to the spectrum graphs commonly used by de novo sequencing software."
- [readme] This improved version was made by Florent Glück (github user: florentgluck), and is 1.4 times more efficient than the original software.: "This improved version was made by Florent Glück (github user: florentgluck), and is 1.4 times more efficient than the original software."
- [readme] But I will not make any new development nor will I perform any maintenance/support on the tool.: "But I will not make any new development nor will I perform any maintenance/support on the tool."
