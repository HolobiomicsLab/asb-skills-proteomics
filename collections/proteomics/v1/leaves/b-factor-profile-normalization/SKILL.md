---
name: b-factor-profile-normalization
description: Use when you have X-ray protein structures in PDB format (ATOMIC ATOM
  records with Cα B-factors) and need normalized B'-factor/ΔB' profiles or mobility-based
  (rigidification vs. flexibility gain) interpretation at ligand-binding sites.
when_to_use_negative:
- Input structures are not X-ray structures or lack per-residue B-factors (e.g. NMR
  ensembles or predicted models), since the method normalizes X-ray B-factor profiles
- You need quantitative thermodynamic or kinetic mobility measurements rather than
  a crystallographic B'-factor/ΔB' interpretation
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: BANΔIT
  role: JavaScript-based browser application with a GUI used to normalize raw B-factors
    into B' profiles, compute ΔB', and interpret binding-site mobility changes for
    drug design
  repo: https://github.com/FBarthels/BANDIT
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1002/minf.202000144
    title: 'BANΔIT: B’‐Factor Analysis for Drug Design and Structural Biology'
  source_content_hash: sha256:2a0d399027853ce55e3ea60b4536526aee386076d18c2ecc48c8ebb40d79f6bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FBarthels/BANDIT
  tools:
  - BANΔIT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# b-factor-profile-normalization

## Summary

Normalizes raw per-residue B-factors from X-ray PDB structures into B' profiles and computes per-residue ΔB' deviations using the BANΔIT JavaScript browser toolkit, because protein mobility changes have been associated with protein-ligand interaction quality and structure-based drug design.

## When to use

Use when you have X-ray protein structures in PDB format (ATOMIC ATOM records with Cα B-factors) and need normalized B'-factor/ΔB' profiles or mobility-based (rigidification vs. flexibility gain) interpretation at ligand-binding sites.

## When NOT to use

- Input structures are not X-ray structures or lack per-residue B-factors (e.g. NMR ensembles or predicted models), since the method normalizes X-ray B-factor profiles
- You need quantitative thermodynamic or kinetic mobility measurements rather than a crystallographic B'-factor/ΔB' interpretation

## Inputs

- PDB-format coordinate file of an X-ray protein structure with per-residue (Cα) B-factors in ATOM records
- Optional: ligand-bound and reference (unbound/normalized) structures of the same protein for ΔB'-based binding-site comparison

## Outputs

- Normalized B' profile and per-residue ΔB' values for each residue
- bfactor_normalized_profile.csv — residue-identifier, B', and ΔB' table
- delta_bprime_classification.csv — one row per complex and residue with ΔB'-derived mobility-change classification (rigidification vs. flexibility gain)
- pharmacophore_model file — structured pharmacophore feature specification annotated with B'-factor support

## How to apply

Parse the PDB-format coordinate file and extract the per-residue B-factor profile from ATOM records (Cα B-factors per residue). Apply the BANΔIT normalization scheme to convert raw B-factors into the normalized B' profile. For each residue, compute ΔB' as the deviation of its B' value from the local mean over the surrounding sequence window. Assemble residue-identifier, B', and ΔB' values into a table (bfactor_normalized_profile.csv); for protein-ligand complexes, compare ligand-bound versus reference normalized profiles and classify each binding-site residue by its ΔB'-derived mobility change (rigidification vs. flexibility gain). The rationale is that B'-factor-supported mobility patterns indicate protein-ligand interaction quality and can annotate pharmacophore features, e.g. for the SARS CoV-2 main protease inhibitor model.

## Related tools

- **BANΔIT** (JavaScript-based browser application with a GUI used to normalize raw B-factors into B' profiles, compute ΔB', and interpret binding-site mobility changes for drug design) — https://github.com/FBarthels/BANDIT

## Examples

```
git clone https://github.com/FBarthels/BANDIT && open the BANΔIT browser application to load a PDB file and export the normalized B'/ΔB' profile
```

## Evaluation signals

- bfactor_normalized_profile.csv exists, is parseable CSV, and contains residue-identifier, B', and ΔB' columns for every residue of the chain
- Normalized B' values follow the BANΔIT normalization scheme and ΔB' values equal each residue's deviation from the local sequence-window mean
- Per-complex ΔB' classifications in delta_bprime_classification.csv are qualitatively consistent with the rigidification vs. flexibility-gain interpretations reported in the article
- pharmacophore_model file parses as valid structured pharmacophore data with each feature carrying its B'-factor support, consistent with the reported SARS CoV-2 main protease pharmacophore application
- Loading the same PDB into the BANΔIT GUI reproduces the same B'/ΔB' values as the scripted workflow

## Limitations

- BANΔIT is a JavaScript browser application with a graphical user interface; it is not provided as a command-line or Python library in the article context, so automation requires the browser tool or the source code from GitHub
- Interpretation depends on crystallographic B-factors, so results reflect X-ray-derived mobility information tied to protein-ligand interaction quality, not direct experimental dynamics

## Evidence

- [abstract] BANΔIT browser application: [evidence span withheld — anchor status: not_found]
- [abstract] B-factor profiles for drug design: "The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design since protein mobility changes have been associated with the quality of protein‐ligand"
- [abstract] pharmacophore application: "including the development of a B’‐factor‐supported pharmacophore model for SARS CoV‐2 main protease inhibitors."
- [other] source code availability: "The source code can be downloaded from https://github.com/FBarthels/BANDIT."
