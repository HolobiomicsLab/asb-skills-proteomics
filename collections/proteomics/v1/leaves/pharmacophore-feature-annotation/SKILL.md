---
name: pharmacophore-feature-annotation
description: Use when an X-ray protein–ligand complex structure and its BANΔIT-normalized
  B'-factor/ΔB' profiles are available and a structure-based pharmacophore model should
  be annotated with protein-mobility (B'-factor) support for drug design.
when_to_use_negative:
- No X-ray B'-factor data are available for the protein (e.g. only NMR or predicted
  structures), since the method relies on B'-factor profile normalization from X-ray
  protein structures
- The goal is conventional ligand- or shape-based pharmacophore generation that does
  not require B'-factor/ΔB' interaction-quality annotation
tools:
- name: BANΔIT
  role: Browser application used to normalize B'-factor/ΔB' profiles and identify
    binding-site regions whose B'-factor patterns support favorable protein-ligand
    interactions for pharmacophore feature annotation
  repo: https://github.com/FBarthels/BANDIT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/minf.202000144
    title: 'BANΔIT: B’‐Factor Analysis for Drug Design and Structural Biology'
  source_content_hash: sha256:fb9f1bbaea5196e4d9ecf46892ec85754e7f396be75fa8cf22f01180f3380b46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FBarthels/BANDIT
  tools:
  - BANΔIT
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pharmacophore-feature-annotation

## Summary

Builds a B'-factor-supported pharmacophore model by mapping BANΔIT-normalized B'-factor/ΔB' residue profiles onto binding-site residues of an X-ray structure and annotating pharmacophore feature types (e.g. hydrogen-bond donor/acceptor, hydrophobic, aromatic) with their B'-factor support, as demonstrated for SARS-CoV-2 main protease inhibitors.

## When to use

Use when an X-ray protein–ligand complex structure and its BANΔIT-normalized B'-factor/ΔB' profiles are available and a structure-based pharmacophore model should be annotated with protein-mobility (B'-factor) support for drug design.

## When NOT to use

- No X-ray B'-factor data are available for the protein (e.g. only NMR or predicted structures), since the method relies on B'-factor profile normalization from X-ray protein structures
- The goal is conventional ligand- or shape-based pharmacophore generation that does not require B'-factor/ΔB' interaction-quality annotation

## Inputs

- Normalized B'-factor and ΔB' residue profiles for the target protein produced by the BANΔIT normalization step
- Published X-ray structure of the target protein (e.g. the SARS-CoV-2 main protease)

## Outputs

- Annotated pharmacophore feature set binding-site features (donor/acceptor, hydrophobic, aromatic) tagged with B'-factor support
- Structured pharmacophore_model file listing each pharmacophore feature with its B'-factor support

## How to apply

First run the BANΔIT toolkit to obtain normalized B'-factor and ΔB' residue profiles for the target protein from its X-ray structure. Load the published X-ray structure and map the normalized B'-factor/ΔB' values onto the binding-site residues. Using BANΔIT, identify binding-site regions where the B'-factor/ΔB' patterns indicate favorable protein-ligand interaction quality — the rationale being that protein mobility changes have been associated with the quality of protein-ligand interactions — and annotate the corresponding pharmacophore feature types (hydrogen-bond donor/acceptor, hydrophobic, aromatic) with their B'-factor support. Assemble the annotated features into a B'-factor-supported pharmacophore model specification and export it as a structured pharmacophore_model file listing each feature with its B'-factor support, following the exemplary SARS-CoV-2 main protease inhibitor application.

## Related tools

- **BANΔIT** (Browser application used to normalize B'-factor/ΔB' profiles and identify binding-site regions whose B'-factor patterns support favorable protein-ligand interactions for pharmacophore feature annotation) — https://github.com/FBarthels/BANDIT

## Evaluation signals

- A pharmacophore_model file exists and parses as valid structured pharmacophore data
- The pharmacophore_model lists each feature with its B'-factor support annotation
- The model is consistent with the exemplary SARS-CoV-2 main protease pharmacophore application reported in the article
- Normalized B'-factor/ΔB' values are correctly mapped onto binding-site residues of the published X-ray structure
- Feature types are restricted to the article's annotated categories (hydrogen-bond donor/acceptor, hydrophobic, aromatic)

## Limitations

- Applicability depends on X-ray B'-factor data quality, since protein mobility changes derived from B'-factor profiles are the basis for interaction-quality assessment
- BANΔIT is a JavaScript-based browser application rather than a command-line pipeline, so the workflow is GUI-driven and demonstrated as a single exemplary case (SARS-CoV-2 main protease inhibitors)

## Evidence

- [abstract] abstract: "The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design since protein mobility changes have been associated with the quality of protein‐ligand"
- [other] other: "The source code can be downloaded from https://github.com/FBarthels/BANDIT."
