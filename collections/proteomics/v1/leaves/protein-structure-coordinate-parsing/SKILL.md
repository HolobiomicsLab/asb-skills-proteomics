---
name: protein-structure-coordinate-parsing
description: Use when the input is a PDB-format coordinate file from an X-ray structure
  and the goal is a per-residue B-factor (mobility) profile for downstream B'/ΔB'
  normalization or pharmacophore support.
when_to_use_negative:
- Input structures are not from X-ray crystallography (e.g. NMR models, AlphaFold
  predictions) — B-factors there do not reflect experimental mobility as analyzed
  by BANΔIT
- The analysis target is raw per-atom B-factors rather than a normalized per-residue
  B'/ΔB' profile
- A normalized profile table (residue identifier, B', ΔB') already exists and only
  downstream interpretation is needed
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_1317
tools:
- name: BANΔIT
  role: Browser-based GUI application that parses the PDB coordinate input, normalizes
    the extracted Cα B-factor profile to B', computes ΔB' against the local mean,
    and writes bfactor_normalized_profile.csv
  repo: https://github.com/FBarthels/BANDIT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/minf.202000144
    title: 'BANΔIT: B’‐Factor Analysis for Drug Design and Structural Biology'
  source_content_hash: sha256:fdd8eb2731fca3d69cbd0a86817875b0f5bcb0a9e6dc8f591eed3b4d51d88120
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FBarthels/BANDIT
  tools:
  - BANΔIT
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_1317
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# protein-structure-coordinate-parsing

## Summary

Parses PDB-format X-ray protein coordinate files to extract per-residue Cα B-factor profiles, the input required for BANΔIT's B' normalization and ΔB' deviation analysis in structure-based drug design.

## When to use

Use when the input is a PDB-format coordinate file from an X-ray structure and the goal is a per-residue B-factor (mobility) profile for downstream B'/ΔB' normalization or pharmacophore support.

## When NOT to use

- Input structures are not from X-ray crystallography (e.g. NMR models, AlphaFold predictions) — B-factors there do not reflect experimental mobility as analyzed by BANΔIT
- The analysis target is raw per-atom B-factors rather than a normalized per-residue B'/ΔB' profile
- A normalized profile table (residue identifier, B', ΔB') already exists and only downstream interpretation is needed

## Inputs

- PDB-format coordinate file of an X-ray protein structure (ATOM records with per-atom B-factors)

## Outputs

- Per-residue B-factor profile extracted from Cα ATOM records
- bfactor_normalized_profile.csv containing per-residue residue-identifier, B', and ΔB' values

## How to apply

Read the PDB-format file and extract, from ATOM records, the Cα B-factor for each residue in sequence order to build the raw per-residue B-factor profile. Because raw B-factors are not comparable between structures (different refinement scales), feed the profile into the BANΔIT normalization scheme to obtain the B' profile, then compute ΔB' per residue as its deviation from the local mean over the surrounding sequence window. Assemble residue identifier, B', and ΔB' into a table and write it to bfactor_normalized_profile.csv. BANΔIT provides a JavaScript-based graphical user interface in the browser for this normalization and analysis, so no scripting library is required; the source code is available at github.com/FBarthels/BANDIT. Validate that the extracted profile covers exactly the Cα records present in the input file before normalizing.

## Related tools

- **BANΔIT** (Browser-based GUI application that parses the PDB coordinate input, normalizes the extracted Cα B-factor profile to B', computes ΔB' against the local mean, and writes bfactor_normalized_profile.csv) — https://github.com/FBarthels/BANDIT

## Evaluation signals

- The number of residues in the output profile equals the number of Cα ATOM records in the input PDB file
- bfactor_normalized_profile.csv contains exactly the columns residue identifier, B', and ΔB'
- ΔB' for each residue matches its B' value minus the local mean over the surrounding sequence window
- B' values fall on the normalized scale rather than the raw refined B-factor scale of the input file (values differ from the raw ATOM-record B-factors)
- Residue identifiers in the output preserve the sequence order of the input coordinate file

## Limitations

- Applicable to X-ray protein structures; B-factor profiles are meaningful only where crystallographic B-factors encode mobility
- BANΔIT is a JavaScript browser application with a GUI, so it cannot be scripted headlessly via the workflow described here

## Evidence

- [abstract] we have developed a JavaScript‐based browser application that provides a graphical user interface for the normalization and: [evidence span withheld — anchor status: not_found]
- [other] Parse the PDB-format coordinate file and extract the per-residue B-factor profile from ATOM records: [evidence span withheld — anchor status: not_found]
- [abstract] The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design: "The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design since protein mobility changes have been associated with the quality of protein‐ligand"
- [abstract] The source code can be downloaded from https://github.com/FBarthels/BANDIT.: "The source code can be downloaded from https://github.com/FBarthels/BANDIT."
