---
name: peak-node-representation
description: Use when MS/MS peak lists (m/z and intensity per fragment peak) must
  be converted into a spectrum-graph (one node per peak, edges labeled with mass differences)
  before peptide identification or modification-tolerant matching, e.g. as input to
  Popitam.
when_to_use_negative:
- The input is already a spectrum graph or a scored peptide list — this skill only
  converts raw peak lists into graph form.
- The goal is full peptide identification/ranking by correlation score; this skill
  produces only the intermediate graph representation that Popitam parses, not the
  ranked peptides themselves.
- The data are MS1 profiles or non-tandem spectra — the representation is specific
  to MS/MS fragment peak data.
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0080
tools:
- name: Popitam
  role: Target tool whose identification process consumes the structured spectrum-graph
    representation of the source MS/MS spectrum; this skill builds that representation
    from MS/MS peak data.
  repo: https://github.com/chernan/popitam
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.200300402
    title: 'Popitam: Towards new heuristic strategies to improve protein identification
      from tandem mass spectrometry data'
  source_content_hash: sha256:ad33122263c5201b2614ebfce3a85ced01be05fb983e7b4d2d9b4117c649dbc8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chernan/popitam
  tools:
  - Popitam
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# peak-node-representation

## Summary

Builds a structured spectrum-graph representation of a source MS/MS spectrum, where each fragment peak becomes a node carrying its mass (m/z) and intensity, and directed edges between nodes are annotated with mass differences. This mirrors the spectrum graphs commonly used by de novo sequencing software and is the representation Popitam matches theoretical peptide sequences against.

## When to use

Use when MS/MS peak lists (m/z and intensity per fragment peak) must be converted into a spectrum-graph (one node per peak, edges labeled with mass differences) before peptide identification or modification-tolerant matching, e.g. as input to Popitam.

## When NOT to use

- The input is already a spectrum graph or a scored peptide list — this skill only converts raw peak lists into graph form.
- The goal is full peptide identification/ranking by correlation score; this skill produces only the intermediate graph representation that Popitam parses, not the ranked peptides themselves.
- The data are MS1 profiles or non-tandem spectra — the representation is specific to MS/MS fragment peak data.

## Inputs

- Source MS/MS peak list (per-spectrum fragment peaks with m/z and intensity; sample spectra from the Popitam repository or PRIDE/MassIVE peak lists)

## Outputs

- One structured spectrum-graph file per input spectrum (JSON or TSV), where each node carries mass and intensity fields and each directed edge carries a mass-difference field

## How to apply

Load the source MS/MS peak list (sample spectra shipped in the github.com/chernan/popitam deposit, or a public PRIDE/MassIVE peak list) keeping m/z and intensity values for each fragment peak, since these are required as node attributes. Create one node per peak, storing its mass (m/z) and intensity. Connect pairs of peaks with directed edges and annotate each edge with the mass difference between the two peaks, mirroring the spectrum-graph structure used by de novo sequencing software. Write the resulting nodes and edges to one structured graph file per input spectrum (JSON or TSV with explicit mass and intensity fields). The rationale is that Popitam's identification process matches theoretical peptide sequences from a database against this structured representation, whose parsing is a combinatorial task later handled by an Ant Colony Optimization algorithm.

## Related tools

- **Popitam** (Target tool whose identification process consumes the structured spectrum-graph representation of the source MS/MS spectrum; this skill builds that representation from MS/MS peak data.) — https://github.com/chernan/popitam

## Examples

```
python build_spectrum_graph.py --peaks spectrum_001.tsv --fields mz,intensity --output spectrum_001_graph.
```

## Evaluation signals

- One graph file exists per input spectrum.
- Every graph file parses in the expected format (JSON or TSV).
- Every node carries both a mass (m/z) field and an intensity field.
- Every directed edge carries a mass-difference field, and spot-check that each edge's mass difference equals the difference between the m/z values of its two endpoint nodes.
- The number of nodes equals the number of peaks in the source peak list for that spectrum.

## Limitations

- The spectrum graph can be a highly combinatorial structure; Popitam relies on an Ant Colony Optimization algorithm to parse it, so the graph representation alone does not resolve which sections are relevant.
- The Popitam repository is archived: the maintainer states she 'will not make any new development nor will I perform any maintenance/support on the tool', so expect no fixes for edge cases.
- The paper's scope is characterizing peptides with mutations or unexpected post-translational modifications from MS/MS data; low-quality data handling is claimed as a property of the overall algorithm but no concrete quality thresholds are given in the provided context.

## Evidence
