---
name: structure-based-drug-design-b-factor-annotation
description: Use when X-ray crystal structures of protein-ligand complexes (with raw
  B-factors) are available and you need to assess ligand-induced rigidification or
  flexibility gain at the binding site for drug optimization decisions.
when_to_use_negative:
- Input structures lack X-ray B-factors (e.g. NMR or predicted models), since the
  ΔB' analysis is grounded in B-factor profiles from X-ray protein structures.
- The goal is quantitative ligand-affinity prediction rather than qualitative interpretation
  of protein mobility changes at the binding site.
- No reference/unbound structure or normalized baseline is available, so ligand-bound
  vs. reference ΔB' comparison cannot be computed.
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_0081
tools:
- name: BANΔIT
  role: JavaScript-based browser application providing a graphical user interface
    for the normalization and analysis (ΔB' interpretation) of B'-factor profiles
    from protein-ligand complexes.
  repo: https://github.com/FBarthels/BANDIT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/minf.202000144
    title: 'BANΔIT: B’‐Factor Analysis for Drug Design and Structural Biology'
  source_content_hash: sha256:d5a0d1e5be236557a0ecea6e786e1c134db3f4a2725a961c4485f43dd8c395e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FBarthels/BANDIT
  tools:
  - BANΔIT
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# structure-based-drug-design-b-factor-annotation

## Summary

A skill for normalizing and interpreting B'-factor profiles from X-ray protein-ligand complexes using the BANΔIT toolkit, deriving per-residue ΔB' mobility classifications that support structure-based drug design (e.g. pharmacophore model refinement).

## When to use

Use when X-ray crystal structures of protein-ligand complexes (with raw B-factors) are available and you need to assess ligand-induced rigidification or flexibility gain at the binding site for drug optimization decisions.

## When NOT to use

- Input structures lack X-ray B-factors (e.g. NMR or predicted models), since the ΔB' analysis is grounded in B-factor profiles from X-ray protein structures.
- The goal is quantitative ligand-affinity prediction rather than qualitative interpretation of protein mobility changes at the binding site.
- No reference/unbound structure or normalized baseline is available, so ligand-bound vs. reference ΔB' comparison cannot be computed.

## Inputs

- X-ray protein-ligand complex structures retrieved from the PDB (SARS-CoV-2 main protease inhibitor complexes among others)
- Raw per-residue B-factor profiles of ligand-bound structures
- Reference (unbound/normalized) B'-factor profiles of the same protein chain

## Outputs

- Normalized B'-factor profiles per protein chain
- Per-residue ΔB' values at the ligand-binding site
- Per-residue mobility-change classifications (rigidification vs. flexibility gain), assembled as delta_bprime_classification.csv with one row per complex and residue
- Qualitative drug-optimization conclusions such as a B'-factor-supported pharmacophore model for SARS-CoV-2 main protease inhibitors

## How to apply

Retrieve the crystallographic protein-ligand complex structures (e.g. SARS-CoV-2 main protease inhibitor complexes) from the PDB and load each into the BANΔIT JavaScript-based browser application, which provides a graphical user interface for the normalization and analysis of B'-factor profiles. Run the BANΔIT normalization step to convert raw B-factors into normalized B' profiles for the protein chain. Compute per-residue ΔB' values at the ligand-binding site by comparing ligand-bound and reference (unbound/normalized) B'-factor profiles. Classify each binding-site residue by its ΔB'-derived mobility change (rigidification vs. flexibility gain) using the BANΔIT interpretation scheme, then assemble the classifications into a tabular CSV (e.g. delta_bprime_classification.csv) with one row per complex and residue. The rationale is that protein mobility changes have been associated with the quality of protein-ligand interactions, so ΔB' patterns feed directly into drug design artifacts such as a B'-factor-supported pharmacophore model.

## Related tools

- **BANΔIT** (JavaScript-based browser application providing a graphical user interface for the normalization and analysis (ΔB' interpretation) of B'-factor profiles from protein-ligand complexes.) — https://github.com/FBarthels/BANDIT

## Evaluation signals

- Output table delta_bprime_classification.csv exists and is parseable CSV with one row per complex and residue.
- Each binding-site residue carries a ΔB'-derived mobility classification (rigidification vs. flexibility gain) consistent with the BANΔIT interpretation scheme.
- Per-complex ΔB' classifications qualitatively reproduce the drug-optimization conclusions the paper reports (consistency check against the reported B'-factor-supported interpretation).
- A B'-factor-supported pharmacophore model for SARS-CoV-2 main protease inhibitors can be assembled from the classification results.
- Normalized B' profiles (not raw B-factors) are used as the basis of the ΔB' comparison, confirming the normalization step was run.

## Limitations

- The tool is a JavaScript-based browser application with a graphical user interface, so it is interactive rather than scriptable/CLI-driven.
- Interpretation is qualitative: ΔB' profiles support drug design (e.g. pharmacophore modeling) but do not directly yield quantitative affinity or potency predictions.
- Analysis depends on the quality and availability of X-ray B-factors and a suitable reference (unbound/normalized) profile for comparison.

## Evidence

- [abstract] a JavaScript‐based browser application that provides a graphical user interface for the normalization and analysis of B’‐factor profiles.: "a JavaScript‐based browser application that provides a graphical user interface for the normalization and analysis of B’‐factor profiles."
- [abstract] The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design since protein mobility changes have been associated with the quality of protein‐ligand: "The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design since protein mobility changes have been associated with the quality of protein‐ligand"
- [abstract] including the development of a B’‐factor‐supported pharmacophore model for SARS CoV‐2 main protease inhibitors.: "including the development of a B’‐factor‐supported pharmacophore model for SARS CoV‐2 main protease inhibitors."
- [other] The source code can be downloaded from https://github.com/FBarthels/BANDIT.: "The source code can be downloaded from https://github.com/FBarthels/BANDIT."
