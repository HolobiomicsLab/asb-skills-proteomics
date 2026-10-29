---
name: structure-based-drug-design
description: Use when a protein X-ray structure is available and you want to derive
  a pharmacophore model whose features are annotated with B'-factor/ΔB' support from
  BANΔIT to guide inhibitor design.
when_to_use_negative:
- The input structure has no B-factor/temperature-factor data from X-ray crystallography,
  so no B'-factor normalization or ΔB' interpretation is possible
- You only need generic ligand-based pharmacophore features without B'-factor/ΔB'
  support, which does not require the BANΔIT normalization workflow
- The binding-site residues lack mapped normalized B'-factor/ΔB' values because the
  BANΔIT normalization step has not yet been run
edam_topics:
- http://edamontology.org/topic_3369
tools:
- name: BANΔIT
  role: JavaScript-based browser application used to normalize and analyze B'-factor
    profiles, map ΔB' values onto binding-site residues, and assemble the B'-factor-supported
    pharmacophore model
  repo: https://github.com/FBarthels/BANDIT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/minf.202000144
    title: 'BANΔIT: B’‐Factor Analysis for Drug Design and Structural Biology'
  source_content_hash: sha256:93427afb71d1d5198fee1ce17425bb642fcc8cb2f9984dd5639a0474a826ffe3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FBarthels/BANDIT
  tools:
  - BANΔIT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3369
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# structure-based-drug-design

## Summary

Builds B'-factor-supported pharmacophore models by mapping BANΔIT-normalized B'-factor and ΔB' residue profiles onto binding-site residues of an X-ray structure, so that protein mobility changes inform the quality of protein-ligand interactions during rational drug design (exemplified for SARS-CoV-2 main protease inhibitors).

## When to use

Use when a protein X-ray structure is available and you want to derive a pharmacophore model whose features are annotated with B'-factor/ΔB' support from BANΔIT to guide inhibitor design.

## When NOT to use

- The input structure has no B-factor/temperature-factor data from X-ray crystallography, so no B'-factor normalization or ΔB' interpretation is possible
- You only need generic ligand-based pharmacophore features without B'-factor/ΔB' support, which does not require the BANΔIT normalization workflow
- The binding-site residues lack mapped normalized B'-factor/ΔB' values because the BANΔIT normalization step has not yet been run

## Inputs

- Published X-ray protein structure of the target (e.g. SARS CoV-2 main protease)
- Normalized B'-factor residue profile produced by the BANΔIT normalization step
- ΔB' residue profile produced by the BANΔIT normalization step

## Outputs

- Binding-site pharmacophore features (hydrogen-bond donor/acceptor, hydrophobic, aromatic) annotated with their B'-factor support
- Structured pharmacophore_model file listing each pharmacophore feature with its B'-factor support

## How to apply

First run the BANΔIT toolkit's normalization and analysis of B'-factor profiles on the target X-ray structure (e.g. the SARS CoV-2 main protease) to obtain normalized B'-factor and ΔB' residue profiles. Load these profiles into BANΔIT's JavaScript browser application and map the normalized B'-factor/ΔB' values onto the binding-site residues of the published structure. Identify binding-site regions where the B'-factor/ΔB' patterns support favorable protein-ligand interaction quality, and annotate the corresponding pharmacophore feature types (hydrogen-bond donor/acceptor, hydrophobic, aromatic) with their B'-factor support. Assemble the annotated features into a B'-factor-supported pharmacophore model specification and export it as a structured pharmacophore_model file. The rationale is that protein mobility changes captured in B-factor profiles are associated with the quality of protein-ligand interactions and therefore can steer structure-based drug design.

## Related tools

- **BANΔIT** (JavaScript-based browser application used to normalize and analyze B'-factor profiles, map ΔB' values onto binding-site residues, and assemble the B'-factor-supported pharmacophore model) — https://github.com/FBarthels/BANDIT

## Evaluation signals

- A pharmacophore_model file exists after the workflow completes
- The pharmacophore_model file parses as valid structured pharmacophore data
- Each listed pharmacophore feature is paired with its B'-factor support annotation
- The model is consistent with the exemplary SARS CoV-2 main protease pharmacophore application reported in the article
- The underlying B'-factor/ΔB' profiles reproduce the BANΔIT normalization output for the input X-ray structure

## Limitations

- Depends on the quality of B'-factor profiles from X-ray protein structures; the association between protein mobility changes and protein-ligand interaction quality is the basis of the approach and may not hold for every target
- The BANΔIT application is a JavaScript-based browser tool, so automation and batch processing of the pharmacophore-building workflow may be limited
- Demonstrated as an exemplary drug design application (SARS CoV-2 main protease inhibitors); transfer to other targets requires repeating the normalization and feature-annotation workflow

## Evidence

- [abstract] B-factor profiles for drug design rationale: "The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design since protein mobility changes have been associated with the quality of protein‐ligand"
- [abstract] SARS-CoV-2 Mpro pharmacophore application: "including the development of a B’‐factor‐supported pharmacophore model for SARS CoV‐2 main protease inhibitors."
- [abstract] BANΔIT toolkit description: [evidence span withheld — anchor status: not_found]
- [other] Source code availability: "The source code can be downloaded from https://github.com/FBarthels/BANDIT."
