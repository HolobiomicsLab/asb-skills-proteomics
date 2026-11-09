---
name: local-window-deviation-calculation
description: Use when you have an X-ray protein structure in PDB format and need to
  detect residues whose mobility (Cα B-factor) deviates from their local sequence
  neighborhood, e.g. for protein-ligand interaction analysis.
when_to_use_negative:
- The input is NMR or cryo-EM data rather than an X-ray protein structure, since B'-factor
  profile analysis is grounded in X-ray B-factors
- You need raw B-factor normalization only, not the local-window ΔB' deviation values
- The structure lacks per-residue Cα B-factor information in its ATOM records
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_1317
tools:
- name: BANΔIT
  role: JavaScript-based browser application with a graphical user interface that
    performs the B'-factor normalization and ΔB' local-window deviation calculation
  repo: https://github.com/FBarthels/BANDIT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/minf.202000144
    title: 'BANΔIT: B’‐Factor Analysis for Drug Design and Structural Biology'
  source_content_hash: sha256:6ac84d9eee95116d240d8b4fda85e4c696bc4c596164edcabf628f71fba00062
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FBarthels/BANDIT
  tools:
  - BANΔIT
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_1317
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# local-window-deviation-calculation

## Summary

Computes per-residue ΔB' values — the deviation of each residue's normalized B' factor from the local mean over a surrounding sequence window — as implemented in the BANΔIT JavaScript browser application for structure-based drug design.

## When to use

Use when you have an X-ray protein structure in PDB format and need to detect residues whose mobility (Cα B-factor) deviates from their local sequence neighborhood, e.g. for protein-ligand interaction analysis.

## When NOT to use

- The input is NMR or cryo-EM data rather than an X-ray protein structure, since B'-factor profile analysis is grounded in X-ray B-factors
- You need raw B-factor normalization only, not the local-window ΔB' deviation values
- The structure lacks per-residue Cα B-factor information in its ATOM records

## Inputs

- PDB-format coordinate file of an X-ray protein structure (ATOM records with per-atom B-factors)

## Outputs

- bfactor_normalized_profile.csv containing per-residue residue-identifier, B', and ΔB' values

## How to apply

Parse the PDB-format coordinate file and extract the per-residue B-factor profile from ATOM records, taking the Cα B-factor for each residue. Normalize these raw B-factor values to obtain the B' profile according to the BANΔIT normalization scheme. For each residue, compute ΔB' as the deviation of its B' value from the local mean calculated over the surrounding sequence window, so that residues embedded in rigid vs. flexible regions are compared against their own neighborhood rather than global statistics. Assemble the per-residue residue-identifier, B', and ΔB' values into a table and write it to bfactor_normalized_profile.csv. The rationale is that protein mobility changes captured by B-factor profiles have been associated with the quality of protein-ligand interactions, making ΔB' peaks actionable for structure-based drug design (e.g. B'-factor-supported pharmacophore modeling of SARS-CoV-2 main protease inhibitors).

## Related tools

- **BANΔIT** (JavaScript-based browser application with a graphical user interface that performs the B'-factor normalization and ΔB' local-window deviation calculation) — https://github.com/FBarthels/BANDIT

## Evaluation signals

- Output file bfactor_normalized_profile.csv exists and contains one row per residue with residue-identifier, B', and ΔB' columns
- B' values are normalized (B-factor scale removed) rather than raw Cα B-factors copied through
- ΔB' values are approximately zero for residues whose B' matches the local window mean, positive for locally over-flexible and negative for locally over-rigid residues
- Number of rows matches the number of residues with Cα ATOM records in the input PDB file
- Re-running the browser application on the same PDB input reproduces identical B' and ΔB' values

## Limitations

- Restricted to B-factor profiles from X-ray protein structures; B-factors from other structure determination methods are not covered by the article's scope
- BANΔIT is a JavaScript browser application, so it is operated via its graphical user interface rather than scripted CLI automation
- Requires complete per-residue Cα B-factor information; residues missing ATOM records cannot contribute to the profile

## Evidence

- [abstract] With the BANΔIT (B’-factor analysis and ΔB’ interpretation toolkit), we have developed a JavaScript‐based browser application that provides a graphical user interface for the normalization and: [evidence span withheld — anchor status: not_found]
- [abstract] The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design since protein mobility changes have been associated with the quality of protein‐ligand: "The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design since protein mobility changes have been associated with the quality of protein‐ligand"
