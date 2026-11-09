---
name: disulfide-bond-detection-from-structures
description: Use when you have a PDB or mmCIF coordinate file (optionally gzipped)
  and need per-residue disulfide-bond annotations plus geometric features (bend, chirality)
  written to a structured table such as dssp_geometry_annotations.csv.
when_to_use_negative:
- DSSP does not predict secondary structure — do not use for prediction from sequence
  alone; a 3D coordinate file is required
- Input is already an annotated feature table or mmCIF with all required annotations
  present — re-deriving geometry is redundant
- Analysis targets hydrogen-bond-based secondary structure assignment only, with no
  need for SS-bond or chirality annotation
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: mkdssp
  role: Runs DSSP on file content to obtain the parsed structure and residue iteration
    interface used to compute bend, chirality, and SS-bond annotations
  repo: https://github.com/PDB-REDO/dssp
- name: Python
  role: 'Holds the annotation workflow: reads gzip/mmCIF content, computes S-S distances
    against the 3.0 Å cutoff, and writes the CSV table'
- name: CMake
  role: Configures and builds the DSSP Python module (BUILD_PYTHON_MODULE=ON) needed
    for in-script DSSP access
  repo: https://github.com/PDB-REDO/dssp
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/bip.360221211
    title: 'Dictionary of protein secondary structure: Pattern recognition of hydrogen‐bonded
      and geometrical features'
  source_content_hash: sha256:2e53c5e4f6dce03a8816c577c38a29061b208e721e9e18f8ba25ca99ebc862df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PDB-REDO/dssp
  tools:
  - CMake
  - Python
  - mkdssp
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# disulfide-bond-detection-from-structures

## Summary

A DSSP-based method for annotating per-residue geometric structural features of a protein coordinate file — disulfide (SS) bonds, bend, and Cα chirality — into a combined table. It grounds the annotation in the Kabsch & Sander dictionary of geometric features (SS bonds as part of primary structure, chirality of four consecutive Cα positions).

## When to use

Use when you have a PDB or mmCIF coordinate file (optionally gzipped) and need per-residue disulfide-bond annotations plus geometric features (bend, chirality) written to a structured table such as dssp_geometry_annotations.csv.

## When NOT to use

- DSSP does not predict secondary structure — do not use for prediction from sequence alone; a 3D coordinate file is required
- Input is already an annotated feature table or mmCIF with all required annotations present — re-deriving geometry is redundant
- Analysis targets hydrogen-bond-based secondary structure assignment only, with no need for SS-bond or chirality annotation

## Inputs

- Protein coordinate file in PDB or mmCIF format (optionally gzip-compressed, e.g. 1cbs.cif.gz)
- SSBOND header records from the PDB entry
- Parsed DSSP structure object with residue iteration interface

## Outputs

- dssp_geometry_annotations.csv: per-residue table with residue identifiers and bend, chirality, and SS-bond columns
- Per-residue signed chirality values (+/-) and boolean bend flags

## How to apply

Build the DSSP Python module from the PDB-REDO/dssp repository with CMake using -DBUILD_PYTHON_MODULE=ON, then 'cmake --build build' and 'sudo cmake --install build'. Read the coordinate file (e.g. PDB entry 2TLN as a left-handed helix test case, mmCIF or PDB format) in Python, decompressing with gzip if needed, and run the mkdssp 'dssp' object on the file content to obtain the parsed structure and residue iteration interface. For each residue i, compute bend(i) from the five-residue curvature angle between the C-alpha(i-2)->C-alpha(i) and C-alpha(i)->C-alpha(i+2) virtual bonds, flagging when the angle exceeds 70 degrees, and compute chirality as the sign of the dihedral over Cα i-1, i, i+1, i+2 (positive for right-handed helices, negative for ideal twisted β-sheets). Annotate disulfide bonds per residue from SSBOND header records, cross-checking Cys S-S distances against the 3.0 Å cutoff. Write residue identifiers plus bend, chirality, and SS-bond columns to dssp_geometry_annotations.csv.

## Related tools

- **mkdssp** (Runs DSSP on file content to obtain the parsed structure and residue iteration interface used to compute bend, chirality, and SS-bond annotations) — https://github.com/PDB-REDO/dssp
- **Python** (Holds the annotation workflow: reads gzip/mmCIF content, computes S-S distances against the 3.0 Å cutoff, and writes the CSV table)
- **CMake** (Configures and builds the DSSP Python module (BUILD_PYTHON_MODULE=ON) needed for in-script DSSP access) — https://github.com/PDB-REDO/dssp

## Examples

```
from mkdssp import dssp; dssp = dssp(file_content)
```

## Evaluation signals

- dssp_geometry_annotations.csv exists and contains bend, chirality, and SS-bond fields for every residue
- Chirality values are signed (+/-): positive for right-handed helices, negative for ideal twisted β-sheets
- Bend assignments are boolean flags consistent with the >70 degree curvature-angle criterion
- SS-bond assignments agree with SSBOND header records and Cys S-S distances pass the 3.0 Å cutoff cross-check
- Residue identifiers match the input structure's asym_id/seq_id iteration from the dssp object

## Limitations

- Chirality requires four consecutive Cα positions (i-1..i+2), so terminal residues and chain breaks cannot be assigned
- Bend detection requires the i-2 and i+2 Cα positions, so residues near chain termini or gaps are unassignable
- DSSP is designed for standardizing assignment from coordinates, not prediction; a C++20-capable compiler is required to build mkdssp

## Evidence

- [abstract] The end result is a compilation of the primary structure, including SS bonds, secondary structure, and solvent exposure of 62 different globular proteins.: "The end result is a compilation of the primary structure, including SS bonds, secondary structure, and solvent exposure of 62 different globular proteins."
- [abstract] Local chain chirality is defined as torsional handedness of four consecutive Cα positions, positive for right-handed helices and negative for twisted β-sheets: [evidence span withheld — anchor status: not_found]
- [methods] cmake -S . -B build -DBUILD_PYTHON_MODULE=ON: "cmake -S . -B build -DBUILD_PYTHON_MODULE=ON"
- [readme] After that you can use dssp in a python script, like this: "After that you can use dssp in a python script, like this:"
