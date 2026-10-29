---
name: mass-difference-edge-annotation
description: Use when a raw MS/MS peak list must be transformed into a de novo-style
  spectrum graph whose nodes carry mass/intensity and edges carry mass differences,
  as required before Popitam-style graph parsing.
when_to_use_negative:
- Input is already a spectrum graph or structured spectrum representation rather than
  a raw peak list
- The goal is direct database-search scoring of raw spectra without an intermediate
  graph representation (e.g. straightforward database search tools that do not need
  a spectrum graph)
- Data are intact-protein or non-MS/MS spectra where fragment-peak mass differences
  are not meaningful
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0080
tools:
- name: Popitam
  role: Downstream tool that consumes the spectrum-graph representation; it matches
    theoretical peptide sequences from a database against this structured representation
    of the source MS/MS spectrum
  repo: https://github.com/chernan/popitam
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.200300402
    title: 'Popitam: Towards new heuristic strategies to improve protein identification
      from tandem mass spectrometry data'
  source_content_hash: sha256:b8eec178527cfd6d792a34c153a2a5ed4d1dc5e5182ca44bb3f39374fad46dc1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chernan/popitam
  tools:
  - Popitam
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mass-difference-edge-annotation

## Summary

Converts an MS/MS peak list into a spectrum graph — one node per fragment peak carrying m/z and intensity, with directed edges annotated by the mass difference between peak pairs — the structured representation Popitam matches theoretical peptide sequences against.

## When to use

Use when a raw MS/MS peak list must be transformed into a de novo-style spectrum graph whose nodes carry mass/intensity and edges carry mass differences, as required before Popitam-style graph parsing.

## When NOT to use

- Input is already a spectrum graph or structured spectrum representation rather than a raw peak list
- The goal is direct database-search scoring of raw spectra without an intermediate graph representation (e.g. straightforward database search tools that do not need a spectrum graph)
- Data are intact-protein or non-MS/MS spectra where fragment-peak mass differences are not meaningful

## Inputs

- MS/MS peak list (m/z and intensity per fragment peak), e.g. from sample spectra in the Popitam repository or PRIDE/MassIVE
- One peak list per spectrum to be converted

## Outputs

- One structured spectrum-graph file per input spectrum (JSON or TSV)
- Nodes with mass (m/z) and intensity fields
- Directed edges with mass-difference fields

## How to apply

Load the source MS/MS peak list (e.g. the sample spectra in github.com/chernan/popitam or a public PRIDE/MassIVE peak list) and retain m/z and intensity for every fragment peak, since these become node attributes. Create one node per peak storing its mass and intensity. Connect pairs of peaks with directed edges and annotate each edge with the mass difference between the two peaks, mirroring the spectrum-graph structure commonly used by de novo sequencing software. Emit one structured graph file per input spectrum (JSON or TSV with explicit mass, intensity, and mass-difference fields). The output graph is the structured representation of the source MS/MS spectrum that Popitam subsequently parses with an Ant Colony Optimization algorithm.

## Related tools

- **Popitam** (Downstream tool that consumes the spectrum-graph representation; it matches theoretical peptide sequences from a database against this structured representation of the source MS/MS spectrum) — https://github.com/chernan/popitam

## Examples

```
python build_spectrum_graph.py --peak_list sample.mgf --output graph.json --node_fields mass,intensity --edge_field mass_difference
```

## Evaluation signals

- One graph file exists per input spectrum
- Each output file parses in the expected format (JSON or TSV)
- Every node carries explicit mass and intensity fields
- Every edge carries a mass-difference field consistent with the m/z values of its two endpoint nodes
- Node and edge counts are consistent with the number of peaks and peak pairs in the source peak list

## Limitations

- Popitam itself is archived: the maintainer 'will not make any new development nor will I perform any maintenance/support on the tool'
- The representation must stay similar to de novo sequencing spectrum graphs for Popitam's ACO parsing to work; nonstandard node/edge attributes may break downstream parsing
- Graph parsing is a highly combinatorial task, so graph size (number of peaks/pairs) affects tractability

## Evidence

- [abstract] It consists of matching theoretical peptide sequences issued from a database with a structured representation of the source MS/MS spectrum.: "It consists of matching theoretical peptide sequences issued from a database with a structured representation of the source MS/MS spectrum."
- [abstract] The representation is similar to the spectrum graphs commonly used by de novo sequencing software.: "The representation is similar to the spectrum graphs commonly used by de novo sequencing software."
