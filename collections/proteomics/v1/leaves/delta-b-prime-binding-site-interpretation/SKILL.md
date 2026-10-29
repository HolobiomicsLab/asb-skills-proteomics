---
name: delta-b-prime-binding-site-interpretation
description: Use when analyzing X-ray protein-ligand complex structures (e.g. PDB
  entries) to derive ligand-optimization hypotheses from binding-site mobility changes,
  such as building B'-factor-supported pharmacophore models.
when_to_use_negative:
- Input data are not X-ray structures with B-factors (e.g. NMR ensembles or cryo-EM
  maps without per-residue B-factors), so no B'-factor profile can be normalized
- No reference (unbound/normalized) structure is available, so ΔB' differences at
  the binding site cannot be computed
- The goal is general structural visualization rather than interpreting protein-ligand
  interaction quality via mobility changes
edam_topics:
- http://edamontology.org/topic_1317
- http://edamontology.org/topic_3336
tools:
- name: BANΔIT
  role: JavaScript-based browser application used to normalize raw B-factors into
    B' profiles, compute ΔB' values, and interpret binding-site mobility changes via
    its graphical user interface
  repo: https://github.com/FBarthels/BANDIT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/minf.202000144
    title: 'BANΔIT: B’‐Factor Analysis for Drug Design and Structural Biology'
  source_content_hash: sha256:ec47a7c0ad57f1636ac50e72fb45f5d0d8c083abacb89a40c3b454dd37263c61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FBarthels/BANDIT
  tools:
  - BANΔIT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_1317
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# delta-b-prime-binding-site-interpretation

## Summary

Interpret protein-ligand binding quality from crystallographic B'-factor profiles by normalizing raw B-factors with BANΔIT, computing per-residue ΔB' values at the binding site, and classifying residues as rigidified or flexibility-gained to support structure-based drug design decisions.

## When to use

Use when analyzing X-ray protein-ligand complex structures (e.g. PDB entries) to derive ligand-optimization hypotheses from binding-site mobility changes, such as building B'-factor-supported pharmacophore models.

## When NOT to use

- Input data are not X-ray structures with B-factors (e.g. NMR ensembles or cryo-EM maps without per-residue B-factors), so no B'-factor profile can be normalized
- No reference (unbound/normalized) structure is available, so ΔB' differences at the binding site cannot be computed
- The goal is general structural visualization rather than interpreting protein-ligand interaction quality via mobility changes

## Inputs

- Crystallographic protein-ligand complex structures from the PDB (e.g. SARS-CoV-2 main protease inhibitor complexes)
- Reference unbound/normalized B'-factor profiles for the same protein chain

## Outputs

- Normalized B'-factor profiles per protein chain
- Per-residue ΔB' classification table (delta_bprime_classification.csv; one row per complex and residue, tagging rigidification vs. flexibility gain)
- B'-factor-supported drug-optimization conclusions (e.g. a pharmacophore model)

## How to apply

Retrieve the crystallographic protein-ligand complex structures from the PDB and load each into BANΔIT, the JavaScript-based browser application providing a graphical user interface for the normalization and analysis of B'-factor profiles. Run the BANΔIT normalization step to convert raw B-factors into normalized B' profiles for the protein chain. Compute per-residue ΔB' values at the ligand-binding site by comparing ligand-bound and reference (unbound/normalized) B'-factor profiles. Classify each binding-site residue by its ΔB'-derived mobility change — rigidification vs. flexibility gain — following the BANΔIT interpretation scheme, since protein mobility changes have been associated with the quality of protein-ligand interactions. Assemble per-residue classifications across all complexes (e.g. SARS-CoV-2 main protease inhibitor complexes) into a tabular file such as delta_bprime_classification.csv with one row per complex and residue, and use the rigidification/flexibility pattern to derive drug-optimization conclusions such as a B'-factor-supported pharmacophore model.

## Related tools

- **BANΔIT** (JavaScript-based browser application used to normalize raw B-factors into B' profiles, compute ΔB' values, and interpret binding-site mobility changes via its graphical user interface) — https://github.com/FBarthels/BANDIT

## Examples

```
git clone https://github.com/FBarthels/BANDIT
```

## Evaluation signals

- Output file delta_bprime_classification.csv exists and is parseable CSV with one row per complex and residue
- Every binding-site residue carries a ΔB'-derived classification (rigidification vs. flexibility gain) following the BANΔIT interpretation scheme
- Per-complex ΔB' classifications are qualitatively consistent with the exemplary drug-optimization conclusions reported in the paper
- Raw B-factors have been replaced by normalized B' profiles via the BANΔIT normalization step before comparison
- Derived conclusions assemble into a coherent B'-factor-supported pharmacophore model for the analyzed target (e.g. SARS-CoV-2 main protease)

## Limitations

- Analysis is restricted to X-ray protein structures with B-factor profiles; the association between mobility changes and interaction quality is qualitative, supporting interpretation rather than quantitative affinity prediction
- BANΔIT is a browser-based GUI application; the source code must be downloaded from the GitHub repository to run it locally

## Evidence

- [abstract] With the BANΔIT (B’-factor analysis and ΔB’ interpretation toolkit), we have developed a JavaScript‐based browser application that provides a graphical user interface for the normalization and: [evidence span withheld — anchor status: not_found]
- [abstract] The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design since protein mobility changes have been associated with the quality of protein‐ligand: "The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design since protein mobility changes have been associated with the quality of protein‐ligand"
- [abstract] including the development of a B’‐factor‐supported pharmacophore model for SARS CoV‐2 main protease inhibitors.: "including the development of a B’‐factor‐supported pharmacophore model for SARS CoV‐2 main protease inhibitors."
- [other] The source code can be downloaded from https://github.com/FBarthels/BANDIT.: "The source code can be downloaded from https://github.com/FBarthels/BANDIT."
