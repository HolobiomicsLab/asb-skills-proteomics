---
name: per-residue-attribute-extraction
description: Use when an X-ray PDB coordinate file must be converted into a per-residue
  normalized B'-factor and ΔB' profile to probe protein flexibility in structure-based
  drug design.
when_to_use_negative:
- The structure is not from X-ray crystallography (e.g., NMR or cryo-EM), so no crystallographic
  B-factor profile exists to normalize.
- The input is already a normalized per-residue feature table rather than raw PDB
  ATOM records.
- The goal is general structural visualization rather than B'-factor normalization
  and ΔB' interpretation for drug design.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: BANΔIT
  role: JavaScript-based browser application with a GUI that performs the normalization
    and analysis of B'-factor profiles, producing the per-residue B'/ΔB' table
  repo: https://github.com/FBarthels/BANDIT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/minf.202000144
    title: 'BANΔIT: B’‐Factor Analysis for Drug Design and Structural Biology'
  source_content_hash: sha256:cbe34e49feffe0838dddb9520c2b8a49f04e1e1f8b8a74eb6c4952f369c066a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FBarthels/BANDIT
  tools:
  - BANΔIT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# per-residue-attribute-extraction

## Summary

Extract and normalize per-residue B-factors from X-ray PDB structures into B' and ΔB' profiles using BANΔIT, a JavaScript-based browser application for structure-based drug design. B'-factor profiles capture protein mobility changes associated with protein–ligand interaction quality.

## When to use

Use when an X-ray PDB coordinate file must be converted into a per-residue normalized B'-factor and ΔB' profile to probe protein flexibility in structure-based drug design.

## When NOT to use

- The structure is not from X-ray crystallography (e.g., NMR or cryo-EM), so no crystallographic B-factor profile exists to normalize.
- The input is already a normalized per-residue feature table rather than raw PDB ATOM records.
- The goal is general structural visualization rather than B'-factor normalization and ΔB' interpretation for drug design.

## Inputs

- PDB-format coordinate file of an X-ray protein structure (ATOM records with Cα B-factors)

## Outputs

- bfactor_normalized_profile.csv containing per-residue residue-identifier, B', and ΔB' values

## How to apply

Parse the PDB-format coordinate file and extract the per-residue B-factor profile from ATOM records, taking Cα B-factors as the per-residue mobility value. Normalize the B-factor values to obtain the B' profile as defined by the BANΔIT normalization scheme. For each residue, compute ΔB' as the deviation of its B' value from the local mean over the surrounding sequence window, which highlights residues whose mobility departs from their sequence neighborhood. Assemble the per-residue residue-identifier, B', and ΔB' values into a table and write it to bfactor_normalized_profile.csv. The downstream ΔB' values can then support analyses such as a B'-factor-supported pharmacophore model (e.g., for SARS-CoV-2 main protease inhibitors).

## Related tools

- **BANΔIT** (JavaScript-based browser application with a GUI that performs the normalization and analysis of B'-factor profiles, producing the per-residue B'/ΔB' table) — https://github.com/FBarthels/BANDIT

## Evaluation signals

- bfactor_normalized_profile.csv exists and contains the three expected columns: residue identifier, B', and ΔB' per residue.
- The number of rows matches the number of residues with Cα ATOM records in the input PDB file.
- ΔB' for each residue equals the deviation of its B' value from the local mean over the surrounding sequence window (recomputable by an independent implementation).
- Normalized B' values follow the BANΔIT normalization scheme and are consistent in scale across residues.
- Re-running the same PDB input through the GUI reproduces an identical B'/ΔB' table (deterministic output).

## Limitations

- Applies to X-ray protein structures, since B-factors derive from crystallographic refinement; other structure sources lack comparable B-factors.
- ΔB' is a local sequence-window deviation, so it reflects neighborhood-relative mobility, not absolute flexibility.
- Extraction is restricted to Cα B-factors from ATOM records; residues missing Cα atoms will not contribute to the profile.

## Evidence

- [other] BANΔIT browser app workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract tool description: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract rationale: "The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design since protein mobility changes have been associated with the quality of protein‐ligand"
