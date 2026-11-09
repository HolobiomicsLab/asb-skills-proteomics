---
name: de-novo-spectrum-graph-modeling
description: Use when raw MS/MS peak lists (m/z + intensity per fragment peak) must
  be converted into a structured spectrum-graph representation before matching theoretical
  peptide sequences from a database, as Popitam does for mutation/unexpected-PTM characterization.
when_to_use_negative:
- Input is already a structured spectrum graph or a parsed/annotated graph file rather
  than a raw MS/MS peak list
- The goal is plain database searching with fixed expected modifications rather than
  characterization of mutations or unexpected post-translational modifications
- Only a single peptide sequence needs scoring against a spectrum without requiring
  a graph-level representation for combinatorial parsing
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Popitam
  role: 'Consumer of the spectrum graph: matches theoretical peptide sequences from
    a database against the structured representation of the source MS/MS spectrum,
    parsing the graph with an Ant Colony Optimization algorithm and ranking peptides
    by correlation score'
  repo: github.com/chernan/popitam
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.200300402
    title: 'Popitam: Towards new heuristic strategies to improve protein identification
      from tandem mass spectrometry data'
  source_content_hash: sha256:b35806eded14a8b89c502a476ab5811bddf4afbf574d462d1e9b8f22e263d3fb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Popitam
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# de-novo-spectrum-graph-modeling

## Summary

Represent an MS/MS spectrum as a directed spectrum graph (one node per fragment peak with mass and intensity; edges annotated with mass differences), mirroring the de novo sequencing graph structure that Popitam's theoretical-peptide matching relies on.

## When to use

Use when raw MS/MS peak lists (m/z + intensity per fragment peak) must be converted into a structured spectrum-graph representation before matching theoretical peptide sequences from a database, as Popitam does for mutation/unexpected-PTM characterization.

## When NOT to use

- Input is already a structured spectrum graph or a parsed/annotated graph file rather than a raw MS/MS peak list
- The goal is plain database searching with fixed expected modifications rather than characterization of mutations or unexpected post-translational modifications
- Only a single peptide sequence needs scoring against a spectrum without requiring a graph-level representation for combinatorial parsing

## Inputs

- MS/MS peak list for one spectrum (m/z and intensity per fragment peak), e.g. sample spectra shipped in the github.com/chernan/popitam deposit or a PRIDE/MassIVE peak list

## Outputs

- One structured graph file per input spectrum (JSON or TSV) with per-node mass and intensity fields and per-edge mass-difference fields

## How to apply

Load the source MS/MS peak list (sample spectra from the github.com/chernan/popitam deposit or a public PRIDE/MassIVE peak list) and keep both m/z and intensity per fragment peak, since both are node attributes. Create one node per peak storing its mass (m/z) and intensity. Connect pairs of peaks with directed edges and annotate each edge with the mass difference between the two peaks, exactly as in the spectrum graphs used by de novo sequencing software. Write one structured graph file per spectrum (JSON or TSV with explicit mass, intensity, and mass-difference fields); downstream, Popitam parses this graph with an Ant Colony Optimization algorithm to emphasize graph sections relevant to each theoretical sequence, and peptides are ranked by a correlation score.

## Related tools

- **Popitam** (Consumer of the spectrum graph: matches theoretical peptide sequences from a database against the structured representation of the source MS/MS spectrum, parsing the graph with an Ant Colony Optimization algorithm and ranking peptides by correlation score) — github.com/chernan/popitam

## Examples

```
python build_spectrum_graph.py --peaks spectrum.peaks --mz-field mz --intensity-field intensity --edge-field mass_diff --out spectrum_001.graph.json
```

## Evaluation signals

- One structured graph file exists per input spectrum
- Every graph file parses in the expected format (JSON or TSV with explicit fields)
- Every node carries both mass (m/z) and intensity fields
- Every edge carries a mass-difference field equal to the m/z difference of its two endpoint peaks
- Node and edge counts are consistent with the number of fragment peaks in the source peak list

## Limitations

- Popitam is an archived tool: the maintainer 'will not make any new development nor will I perform any maintenance/support on the tool'
- The original repo was migrated from code.google.com purely 'for the sake of archiving'
- The improved version (by Florent Glück) is '1.4 times more efficient' than the original, but results should be compared against the original software behavior
- Graph parsing is 'a highly combinatorial task', handled heuristically by an Ant Colony Optimization algorithm rather than exhaustively
- No discussion-section guidance is available for this article (no changelog content found)

## Evidence

- [abstract] It consists of matching theoretical peptide sequences issued from a database with a structured representation of the source MS/MS spectrum.: "It consists of matching theoretical peptide sequences issued from a database with a structured representation of the source MS/MS spectrum."
- [abstract] The representation is similar to the spectrum graphs commonly used by de novo sequencing software.: "The representation is similar to the spectrum graphs commonly used by de novo sequencing software."
- [abstract] The parsing of the graph, which can be a highly combinatorial task, is performed by a bio-inspired algorithm called Ant Colony Optimization algorithm.: "The parsing of the graph, which can be a highly combinatorial task, is performed by a bio-inspired algorithm called Ant Colony Optimization algorithm."
- [readme] This improved version was made by Florent Glück (github user: florentgluck), and is 1.4 times more efficient than the original software.: "This improved version was made by Florent Glück (github user: florentgluck), and is 1.4 times more efficient than the original software."
