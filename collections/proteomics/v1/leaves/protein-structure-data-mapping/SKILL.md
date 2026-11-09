---
name: protein-structure-data-mapping
description: Use when a published X-ray structure is available and BANΔIT-derived
  B'-factor/ΔB' residue profiles must be transferred onto binding-site residues to
  derive a B'-factor-supported pharmacophore model.
when_to_use_negative:
- The input protein structure is not from X-ray crystallography — B'-factor/ΔB' profiles
  derive from X-ray B-factors and are not meaningful for NMR or cryo-EM-only models.
- The target is to normalize raw B-factors or compute ΔB' profiles for the first time
  — use the BANΔIT normalization/analysis step, not this mapping skill.
- No binding-site residues or ligand interaction hypothesis exist, so there is no
  basis for annotating pharmacophore features with B'-factor support.
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_1317
- http://edamontology.org/topic_0128
tools:
- name: BANΔIT
  role: JavaScript-based browser application used to load normalized B'-factor/ΔB'
    profiles, map them onto X-ray structure residues, identify interaction-quality-supporting
    binding-site regions, and assemble the pharmacophore feature specification.
  repo: https://github.com/FBarthels/BANDIT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/minf.202000144
    title: 'BANΔIT: B’‐Factor Analysis for Drug Design and Structural Biology'
  source_content_hash: sha256:365bd194b17d9ca27f74ddb2d3d517f2dfe686b5665430ea70c1d90cabc8c382
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FBarthels/BANDIT
  tools:
  - BANΔIT
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_1317
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# protein-structure-data-mapping

## Summary

Maps BANΔIT-normalized B'-factor and ΔB' residue profiles onto the residues of a published X-ray protein structure (e.g. the SARS CoV-2 main protease) so that binding-site mobility patterns can be annotated as pharmacophore features with B'-factor support for structure-based drug design.

## When to use

Use when a published X-ray structure is available and BANΔIT-derived B'-factor/ΔB' residue profiles must be transferred onto binding-site residues to derive a B'-factor-supported pharmacophore model.

## When NOT to use

- The input protein structure is not from X-ray crystallography — B'-factor/ΔB' profiles derive from X-ray B-factors and are not meaningful for NMR or cryo-EM-only models.
- The target is to normalize raw B-factors or compute ΔB' profiles for the first time — use the BANΔIT normalization/analysis step, not this mapping skill.
- No binding-site residues or ligand interaction hypothesis exist, so there is no basis for annotating pharmacophore features with B'-factor support.

## Inputs

- Normalized B'-factor and ΔB' residue profiles for the target protein (output of the BANΔIT normalization step)
- Published X-ray structure of the target protein (e.g. SARS CoV-2 main protease PDB structure)
- Binding-site residue definition (binding-site residues to which profile values are mapped)

## Outputs

- Binding-site residues annotated with mapped B'-factor/ΔB' values and pharmacophore feature types (hydrogen-bond donor/acceptor, hydrophobic, aromatic) plus their B'-factor support
- Structured pharmacophore_model. file listing each pharmacophore feature with its B'-factor support

## How to apply

First load the normalized B'-factor and ΔB' residue profiles for the target protein produced by the BANΔIT normalization step. Then load the published X-ray structure (e.g. the SARS CoV-2 main protease) and map the normalized B'-factor/ΔB' values onto the binding-site residues using the BANΔIT browser application. Identify binding-site regions where the B'-factor/ΔB' patterns support favorable protein-ligand interaction quality, and annotate the corresponding pharmacophore feature types (hydrogen-bond donor/acceptor, hydrophobic, aromatic) with their B'-factor support. Finally assemble the annotated features into a B'-factor-supported pharmacophore model specification and export it as a structured pharmacophore_model. file. The rationale is that protein mobility changes (B-factor profiles) are associated with protein-ligand interaction quality and can therefore guide pharmacophore feature placement in structure-based drug design.

## Related tools

- **BANΔIT** (JavaScript-based browser application used to load normalized B'-factor/ΔB' profiles, map them onto X-ray structure residues, identify interaction-quality-supporting binding-site regions, and assemble the pharmacophore feature specification.) — https://github.com/FBarthels/BANDIT

## Examples

```
Open BANΔIT at github.com/FBarthels/BANDIT, load normalized B'-factor/ΔB' profiles + SARS-CoV-2 Mpro PDB, map profiles onto binding-site residues, export pharmacophore_model. file.
```

## Evaluation signals

- pharmacophore_model. file exists and parses as valid structured pharmacophore data.
- Every feature entry in the pharmacophore model lists its B'-factor support value.
- The mapped feature set is consistent with the exemplary SARS CoV-2 main protease pharmacophore application reported in the article.
- Each annotated pharmacophore feature type (hydrogen-bond donor/acceptor, hydrophobic, aromatic) corresponds to a binding-site residue carrying a mapped B'-factor/ΔB' value.
- Mapped B'-factor/ΔB' values match the normalized profiles produced by the upstream BANΔIT normalization step.

## Limitations

- The skill depends on X-ray B-factors, so results inherit the resolution and refinement quality of the published structure.
- Pharmacophore feature annotation rests on the association between B-factor profile changes and protein-ligand interaction quality, an interpretive rather than deterministic link.
- BANΔIT is a JavaScript-based browser application, so mapping is performed interactively through its graphical user interface rather than a scripted pipeline.

## Evidence

- [abstract] B'-factor profiles from X-ray protein structures ... structure-based drug design: "The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design since protein mobility changes have been associated with the quality of protein‐ligand"
- [abstract] B'-factor-supported pharmacophore model: "including the development of a B’‐factor‐supported pharmacophore model for SARS CoV‐2 main protease inhibitors."
- [abstract] BANΔIT browser application for normalization and analysis of B'-factor profiles: [evidence span withheld — anchor status: not_found]
- [abstract] Source code repository: "The source code can be downloaded from https://github.com/FBarthels/BANDIT."
