---
name: protein-binding-site-analysis
description: Use when a published X-ray protein structure with B-factor data is available
  and you want to relate residue-level mobility changes to protein-ligand interaction
  quality, e.g. to build or validate a pharmacophore model for binding-site inhibitors.
when_to_use_negative:
- The input structure lacks per-residue B-factor data or is not an X-ray structure,
  since B'-factor/ΔB' interpretation depends on X-ray crystallographic B-factors
- The goal is generic binding-site pocket detection rather than interpreting residue
  mobility (B'-factor/ΔB' patterns) in relation to protein-ligand interaction quality
- A pharmacophore model already exists and does not need B'-factor support annotation;
  BANΔIT normalization alone would suffice
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0166
tools:
- name: BANΔIT
  role: JavaScript-based browser application used to normalize and analyze B'-factor
    profiles and to identify binding-site regions where B'-factor/ΔB' patterns support
    protein-ligand interaction quality for the pharmacophore model
  repo: https://github.com/FBarthels/BANDIT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/minf.202000144
    title: 'BANΔIT: B’‐Factor Analysis for Drug Design and Structural Biology'
  source_content_hash: sha256:f4108b02045ed8e2bb9e357f5e7ef03fd5f061b8ba3de8038fc2ca2b523e79ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FBarthels/BANDIT
  tools:
  - BANΔIT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0166
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# protein-binding-site-analysis

## Summary

Skill for analyzing X-ray protein structures via B'-factor profiles: normalize B-factors and compute ΔB' residue profiles with BANΔIT, then interpret mobility patterns at binding-site residues to support structure-based drug design, e.g. a B'-factor-supported pharmacophore model for SARS-CoV-2 main protease inhibitors.

## When to use

Use when a published X-ray protein structure with B-factor data is available and you want to relate residue-level mobility changes to protein-ligand interaction quality, e.g. to build or validate a pharmacophore model for binding-site inhibitors.

## When NOT to use

- The input structure lacks per-residue B-factor data or is not an X-ray structure, since B'-factor/ΔB' interpretation depends on X-ray crystallographic B-factors
- The goal is generic binding-site pocket detection rather than interpreting residue mobility (B'-factor/ΔB' patterns) in relation to protein-ligand interaction quality
- A pharmacophore model already exists and does not need B'-factor support annotation; BANΔIT normalization alone would suffice

## Inputs

- Published X-ray protein structure of the target (e.g. SARS CoV-2 main protease) with per-residue B-factor values
- Binding-site residue definitions for the target structure
- Optional: previously normalized B'-factor and ΔB' residue profiles from the BANΔIT normalization step

## Outputs

- Normalized B'-factor and ΔB' residue profiles for the protein structure
- Binding-site annotations mapping pharmacophore feature types (hydrogen-bond donor/acceptor, hydrophobic, aromatic) to their B'-factor support
- Structured pharmacophore_model file listing each pharmacophore feature with its B'-factor support

## How to apply

Load the raw B-factor data of the X-ray structure into the BANΔIT JavaScript browser application, which performs normalization and analysis of B'-factor profiles and produces normalized B'-factor and ΔB' residue profiles. Map these normalized values onto the residues of the binding site in the published structure (for the exemplary application, the SARS CoV-2 main protease). Identify binding-site regions where the B'-factor/ΔB' patterns indicate favorable protein-ligand interaction quality, and annotate the corresponding pharmacophore feature types (e.g. hydrogen-bond donor/acceptor, hydrophobic, aromatic) with their B'-factor support. Assemble the annotated features into a B'-factor-supported pharmacophore model specification and export it as a structured pharmacophore_model file, then validate that it parses and is consistent with the reported application.

## Related tools

- **BANΔIT** (JavaScript-based browser application used to normalize and analyze B'-factor profiles and to identify binding-site regions where B'-factor/ΔB' patterns support protein-ligand interaction quality for the pharmacophore model) — https://github.com/FBarthels/BANDIT

## Evaluation signals

- A pharmacophore_model file is produced and exists at the expected location
- The pharmacophore_model file parses as valid structured pharmacophore data listing each feature with its B'-factor support
- The model is consistent with the exemplary SARS CoV-2 main protease pharmacophore application reported in the article
- Each annotated binding-site pharmacophore feature carries an explicit B'-factor/ΔB' support value from the normalized profiles
- The normalized B'-factor and ΔB' profiles cover the binding-site residues of the loaded X-ray structure

## Limitations

- The method relies on B-factor profiles from X-ray protein structures; it presumes that protein mobility changes are associated with the quality of protein-ligand interactions
- The SARS CoV-2 main protease pharmacophore model is presented as an exemplary/demonstrative drug design application, not a validated discovery pipeline
- BANΔIT is a JavaScript-based browser application, so the workflow is interactive via a GUI rather than a scriptable command-line pipeline

## Evidence

- [abstract] With the BANΔIT (B’-factor analysis and ΔB’ interpretation toolkit), we have developed a JavaScript‐based browser application that provides a graphical user interface for the normalization and: [evidence span withheld — anchor status: not_found]
- [abstract] The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design since protein mobility changes have been associated with the quality of protein‐ligand: "The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design since protein mobility changes have been associated with the quality of protein‐ligand"
- [abstract] including the development of a B’‐factor‐supported pharmacophore model for SARS CoV‐2 main protease inhibitors.: "including the development of a B’‐factor‐supported pharmacophore model for SARS CoV‐2 main protease inhibitors."
- [abstract] provides a graphical user interface for the normalization and analysis of B’‐factor profiles.: "provides a graphical user interface for the normalization and analysis of B’‐factor profiles."
- [other] The source code can be downloaded from https://github.com/FBarthels/BANDIT.: "The source code can be downloaded from https://github.com/FBarthels/BANDIT."
