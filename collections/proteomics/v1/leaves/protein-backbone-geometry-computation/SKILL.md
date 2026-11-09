---
name: protein-backbone-geometry-computation
description: Use when a per-residue table of backbone geometry annotations (bend flags,
  signed Cα chirality, disulfide-bond assignments) must be derived from PDB/mmCIF
  coordinates rather than from hydrogen-bond patterns.
when_to_use_negative:
- The goal is hydrogen-bond-based secondary structure assignment (helix/ladder/sheet
  recognition) — use DSSP's primary H-bond pattern feature instead; DSSP does not
  predict secondary structure
- Input is already a per-residue feature/annotation table rather than raw 3D coordinates
- Coordinates lack consecutive Cα positions (e.g. fragments shorter than four residues
  or CA atoms missing), making the four-Cα chirality dihedral undefined
edam_operation: http://edamontology.org/operation_2416
edam_topics:
- http://edamontology.org/topic_0078
- http://edamontology.org/topic_2814
tools:
- name: DSSP
  role: Defines the geometric structural features (bend, chirality) and computes per-residue
    annotations from coordinates
  repo: https://github.com/PDB-REDO/dssp
- name: mkdssp
  role: The rewritten DSSP 4.6 executable/Python entry point ('dssp' object) run on
    file content to obtain the parsed structure and residue iterator
  repo: https://github.com/PDB-REDO/dssp
- name: Python
  role: Hosts the DSSP Python module (built since version 4.5.2); reads gzip-compressed
    files, iterates residues, and writes the annotation CSV
  repo: https://github.com/PDB-REDO/dssp
- name: CMake
  role: Configures the build with -DBUILD_PYTHON_MODULE=ON, builds, and installs the
    Python module
  repo: https://github.com/PDB-REDO/dssp
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/bip.360221211
    title: 'Dictionary of protein secondary structure: Pattern recognition of hydrogen‐bonded
      and geometrical features'
  source_content_hash: sha256:ccd8346984dc2be58acfb12d61763567b297e3c76aeb68906e26442337fc75c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PDB-REDO/dssp
  tools:
  - CMake
  - DSSP
  - Python
  - mkdssp
  edam_operation: http://edamontology.org/operation_2416
  edam_topics:
  - http://edamontology.org/topic_0078
  - http://edamontology.org/topic_2814
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# protein-backbone-geometry-computation

## Summary

Computes DSSP's geometric (non-hydrogen-bonded) per-residue structural features — bend (curvature angle > 70°), local chain chirality (sign of the dihedral over four consecutive Cα positions), and SSBOND-based disulfide annotation — from x-ray coordinate files using the modern mkdssp/DSSP 4.6 Python module.

## When to use

Use when a per-residue table of backbone geometry annotations (bend flags, signed Cα chirality, disulfide-bond assignments) must be derived from PDB/mmCIF coordinates rather than from hydrogen-bond patterns.

## When NOT to use

- The goal is hydrogen-bond-based secondary structure assignment (helix/ladder/sheet recognition) — use DSSP's primary H-bond pattern feature instead; DSSP does not predict secondary structure
- Input is already a per-residue feature/annotation table rather than raw 3D coordinates
- Coordinates lack consecutive Cα positions (e.g. fragments shorter than four residues or CA atoms missing), making the four-Cα chirality dihedral undefined

## Inputs

- Protein coordinate file in mmCIF or PDB format (optionally gzip-compressed, e.g. 1cbs.cif.gz); PDB entry 2TLN is the left-handed helix test case
- SSBOND header records from the PDB entry for disulfide annotation

## Outputs

- Per-residue CSV table (dssp_geometry_annotations.csv) with residue identifiers and bend (boolean), chirality (signed), and SS-bond columns
- Parsed DSSP structure with residue iteration interface (asym_id, seq_id, compound_id, type)

## How to apply

Build the DSSP Python module from the PDB-REDO/dssp repository with 'cmake -S . -B build -DBUILD_PYTHON_MODULE=ON', 'cmake --build build', then 'sudo cmake --install build' (the rewrite needs a C++20-capable compiler). Load the coordinate file (mmCIF or PDB, gzip-decompressed if needed) in Python, run the mkdssp 'dssp' object on the file content, and iterate residues via the returned interface. For each residue i, flag bend(i) when the angle between the Cα(i-2)→Cα(i) and Cα(i)→Cα(i+2) virtual-bond vectors exceeds 70 degrees, and compute chirality as the sign of the dihedral formed by Cα positions i-1, i, i+1, i+2 (positive for right-handed helices, negative for ideal twisted β-sheets). Annotate disulfides per residue from SSBOND header records, cross-checking Cys S–S distances against the 3.0 Å cutoff. Write the combined per-residue table (residue identifiers plus bend, chirality, SS-bond columns) to a CSV for downstream use.

## Related tools

- **DSSP** (Defines the geometric structural features (bend, chirality) and computes per-residue annotations from coordinates) — https://github.com/PDB-REDO/dssp
- **mkdssp** (The rewritten DSSP 4.6 executable/Python entry point ('dssp' object) run on file content to obtain the parsed structure and residue iterator) — https://github.com/PDB-REDO/dssp
- **Python** (Hosts the DSSP Python module (built since version 4.5.2); reads gzip-compressed files, iterates residues, and writes the annotation CSV) — https://github.com/PDB-REDO/dssp
- **CMake** (Configures the build with -DBUILD_PYTHON_MODULE=ON, builds, and installs the Python module) — https://github.com/PDB-REDO/dssp

## Examples

```
cmake -S . -B build -DBUILD_PYTHON_MODULE=ON; cmake --build build; sudo cmake --install build; python -c "from mkdssp import dssp; d = dssp(open('1cbs.cif').read()); [print(r.asym_id, r.seq_id) for r in d]"
```

## Evaluation signals

- dssp_geometry_annotations.csv exists and contains bend, chirality, and SS-bond fields for every residue
- Chirality values are strictly signed (+/-), positive for right-handed helices and negative for ideal twisted β-sheets
- Bend assignments are boolean flags consistent with the >70 degree curvature-angle criterion
- Disulfide annotations agree with SSBOND records and all annotated Cys pairs satisfy the 3.0 Å S–S distance cutoff
- Residue iteration counts match dssp.statistics.residues for the input entry

## Limitations

- The bend/chirality definitions are geometric and non-hydrogen-bonded; they capture curvature and handedness only, not H-bond-based helix/bridge assignments
- A modern compiler is required to build mkdssp because it uses C++20 features; older toolchains will fail
- The rewritten DSSP (4.6) writes an annotated mmCIF by default (secondary structure in _struct_conf) and adds Poly-Proline helices, so output may differ from classic DSSP versions

## Evidence

- [abstract] Local chain chirality is defined as torsional handedness of four consecutive Cα positions, positive for right-handed helices and negative for twisted β-sheets: [evidence span withheld — anchor status: not_found]
- [readme] Python module build and usage are documented in the README: [evidence span withheld — anchor status: external_doc]
- [methods] CMake configure command enabling the Python module: "cmake -S . -B build -DBUILD_PYTHON_MODULE=ON"
- [methods] DSSP run on file content within a Python script: "dssp = dssp(file_content)"
- [readme] DSSP does not predict secondary structure — scope statement: "DSSP does **not** predict secondary structure."
