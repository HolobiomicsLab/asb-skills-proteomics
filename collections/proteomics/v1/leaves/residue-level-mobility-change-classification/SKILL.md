---
name: residue-level-mobility-change-classification
description: Use when analyzing crystallographic protein-ligand complexes (PDB structures
  with raw B-factors) to derive residue-level mobility-change conclusions that inform
  drug optimization, e.g. building a B'-factor-supported pharmacophore model.
when_to_use_negative:
- Input structures lack B-factor data or are not X-ray protein structures — B'-factor
  profile analysis depends on crystallographic B-factors
- The goal is a generic B-factor visualization rather than ligand-site mobility-change
  interpretation for drug design
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: BANΔIT
  role: JavaScript-based browser application providing a GUI for normalization of
    raw B-factors into B' profiles and ΔB'-based interpretation of residue mobility
    changes
  repo: https://github.com/FBarthels/BANDIT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/minf.202000144
    title: 'BANΔIT: B’‐Factor Analysis for Drug Design and Structural Biology'
  source_content_hash: sha256:66a7c8d8ede87f07962cd2e26f4b08f7a3e51b06d3617a340d338f732a3af61d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FBarthels/BANDIT
  tools:
  - BANΔIT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# residue-level-mobility-change-classification

## Summary

Classifies each protein residue near a ligand-binding site as rigidified or flexibility-gained by computing per-residue ΔB' values from normalized B'-factor profiles, using the BANΔIT toolkit to support structure-based drug design decisions.

## When to use

Use when analyzing crystallographic protein-ligand complexes (PDB structures with raw B-factors) to derive residue-level mobility-change conclusions that inform drug optimization, e.g. building a B'-factor-supported pharmacophore model.

## When NOT to use

- Input structures lack B-factor data or are not X-ray protein structures — B'-factor profile analysis depends on crystallographic B-factors
- The goal is a generic B-factor visualization rather than ligand-site mobility-change interpretation for drug design

## Inputs

- Crystallographic protein-ligand complex structures retrieved from the PDB (with raw B-factors), e.g. SARS-CoV-2 main protease inhibitor complexes
- Reference (unbound/normalized) B'-factor profiles for the same protein chain

## Outputs

- delta_bprime_classification.csv — one row per complex and residue, with per-residue ΔB'-derived mobility classifications (rigidification vs. flexibility gain)
- Per-residue ΔB' values at the ligand-binding site

## How to apply

Retrieve the crystallographic protein-ligand complex structures (e.g. SARS-CoV-2 main protease inhibitor complexes) from the PDB and load each into the BANΔIT browser application, a JavaScript-based GUI for the normalization and analysis of B'-factor profiles. Run the BANΔIT normalization step to convert raw B-factors into normalized B' profiles for the protein chain. Compute per-residue ΔB' values at the ligand-binding site by comparing ligand-bound and reference (unbound/normalized) B'-factor profiles. Classify each binding-site residue by its ΔB'-derived mobility change (rigidification vs. flexibility gain) using the BANΔIT interpretation scheme, then assemble one row per complex and residue into a tabular file. The rationale is that protein mobility changes from B-factor profiles are associated with the quality of protein-ligand interactions and can guide structure-based drug design.

## Related tools

- **BANΔIT** (JavaScript-based browser application providing a GUI for normalization of raw B-factors into B' profiles and ΔB'-based interpretation of residue mobility changes) — https://github.com/FBarthels/BANDIT

## Evaluation signals

- delta_bprime_classification.csv exists and is parseable CSV with one row per complex and residue
- Per-complex ΔB' classifications reproduce the exemplary drug-optimization conclusions reported by the paper (qualitative consistency check)
- The resulting B'-factor-supported interpretation is consistent with the reported SARS-CoV-2 main protease pharmacophore model
- Normalization via BANΔIT successfully converts raw B-factors into normalized B' profiles for the protein chain before ΔB' computation

## Limitations

- Analysis is limited to X-ray protein structures where B-factors are present and comparable between ligand-bound and reference states
- Interpretation is qualitative: classifications follow the BANΔIT interpretation scheme and validation is a qualitative consistency check against published conclusions, not a quantitative benchmark

## Evidence

- [abstract] With the BANΔIT (B’-factor analysis and ΔB’ interpretation toolkit), we have developed a JavaScript‐based browser application that provides a graphical user interface for the normalization and: [evidence span withheld — anchor status: not_found]
- [abstract] protein mobility changes associated with interaction quality: "protein mobility changes have been associated with the quality of protein‐ligand"
- [abstract] pharmacophore model finding: "including the development of a B’‐factor‐supported pharmacophore model for SARS CoV‐2 main protease inhibitors."
- [other] source code repository: "The source code can be downloaded from https://github.com/FBarthels/BANDIT."
