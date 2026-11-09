---
name: protein-ligand-complex-structure-retrieval-from-pdb
description: Use when a paper's conclusions rest on X-ray protein-ligand complex structures
  (raw B-factor profiles) that must be fetched from the PDB before per-residue ΔB'
  mobility analysis, e.g. for structure-based drug design.
when_to_use_negative:
- The structure is not an X-ray crystallographic protein-ligand complex — B'-factor
  analysis requires crystallographic B-factor profiles from X-ray protein structures
- Only an unbound/apo structure is available, so no ligand-bound vs. reference comparison
  at the binding site is possible
- The goal is already a derived feature table (e.g. normalized B' or ΔB' values) rather
  than the underlying structures themselves
edam_topics:
- http://edamontology.org/topic_0166
tools:
- name: BANΔIT
  role: Browser application into which each retrieved complex is loaded for normalization
    and interpretation of B'-factor profiles via its graphical user interface
  repo: https://github.com/FBarthels/BANDIT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/minf.202000144
    title: 'BANΔIT: B’‐Factor Analysis for Drug Design and Structural Biology'
  source_content_hash: sha256:de9a28da46e8088b6850937cb3c50b6488ac0e0e615320060f40671c0ff5e88d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FBarthels/BANDIT
  tools:
  - BANΔIT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0166
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# protein-ligand-complex-structure-retrieval-from-pdb

## Summary

Retrieves the crystallographic protein-ligand complex structures cited in a study (e.g. the SARS-CoV-2 main protease inhibitor complexes used for the B'-factor-supported pharmacophore model) from the PDB so their raw B-factors can be normalized and interpreted with BANΔIT.

## When to use

Use when a paper's conclusions rest on X-ray protein-ligand complex structures (raw B-factor profiles) that must be fetched from the PDB before per-residue ΔB' mobility analysis, e.g. for structure-based drug design.

## When NOT to use

- The structure is not an X-ray crystallographic protein-ligand complex — B'-factor analysis requires crystallographic B-factor profiles from X-ray protein structures
- Only an unbound/apo structure is available, so no ligand-bound vs. reference comparison at the binding site is possible
- The goal is already a derived feature table (e.g. normalized B' or ΔB' values) rather than the underlying structures themselves

## Inputs

- PDB entry identifiers or titles of crystallographic protein-ligand complexes cited in the source article (e.g. SARS-CoV-2 main protease inhibitor complexes)
- Downloaded PDB/mmCIF coordinate files containing ATOM records (protein chain) and HETATM records (bound ligand) with raw B-factor columns

## Outputs

- A set of retrieved protein-ligand complex structure files loaded into BANΔIT
- Normalized B' profiles per protein chain
- Per-residue ΔB' classifications (rigidification vs. flexibility gain) at ligand-binding sites
- delta_bprime_classification.csv — one row per complex and residue

## How to apply

Identify the PDB entries cited in the paper for each protein-ligand complex of interest, such as the SARS-CoV-2 main protease inhibitor complexes underlying the B'-factor-supported pharmacophore model. Download the corresponding coordinate files from the RCSB PDB so that both the protein chain and the bound ligand (HETATM records at the binding site) are present. Load each structure into the BANΔIT JavaScript-based browser application, which provides a graphical user interface for the normalization and analysis of B'-factor profiles. Run the BANΔIT normalization to convert raw crystallographic B-factors into normalized B' profiles for the protein chain, then compute per-residue ΔB' values at the ligand-binding site by comparing ligand-bound and reference (unbound/normalized) profiles. Classify each binding-site residue as rigidification vs. flexibility gain using the BANΔIT interpretation scheme, and assemble all classifications into a single tabular file, delta_bprime_classification.csv, with one row per complex and residue.

## Related tools

- **BANΔIT** (Browser application into which each retrieved complex is loaded for normalization and interpretation of B'-factor profiles via its graphical user interface) — https://github.com/FBarthels/BANDIT

## Evaluation signals

- Every cited complex structure is retrieved from the PDB and contains both protein ATOM records and ligand HETATM records at the binding site with a populated B-factor column
- BANΔIT normalization completes for each loaded chain, producing a valid normalized B' profile (no raw B-factors remain unnormalized)
- Per-residue ΔB' classification labels use only the BANΔIT interpretation scheme categories (rigidification vs. flexibility gain)
- delta_bprime_classification.csv exists, is parseable as CSV, and contains one row per complex-residue pair covering all analyzed complexes
- The assembled ΔB' classifications are qualitatively consistent with the paper's reported drug-optimization conclusions (e.g. the B'-factor-supported pharmacophore model for SARS-CoV-2 main protease inhibitors)

## Limitations

- Analysis is restricted to X-ray crystallographic structures — NMR or other structure types lack comparable B-factor profiles
- Qualitative comparison against the paper's reported conclusions is a consistency check, not a quantitative validation metric
- BANΔIT is a browser-based JavaScript application, so very large batches of complexes must be processed through its GUI rather than a headless pipeline

## Evidence

- [abstract] BANΔIT GUI for B'-profile normalization: [evidence span withheld — anchor status: not_found]
- [abstract] B-factor profiles for structure-based drug design: "The analysis of B‐factor profiles from X‐ray protein structures can be utilized for structure‐based drug design since protein mobility changes have been associated with the quality of protein‐ligand"
- [other] BANΔIT source code repository: "The source code can be downloaded from https://github.com/FBarthels/BANDIT."
