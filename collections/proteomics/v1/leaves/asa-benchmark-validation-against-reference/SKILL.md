---
name: asa-benchmark-validation-against-reference
description: Use when you have reimplemented or modified a solvent-exposure calculation
  (water molecules in possible contact per residue) and need to prove numerical correctness
  against published reference ASA values.
when_to_use_negative:
- The goal is secondary structure assignment, not solvent exposure — DSSP does not
  predict secondary structure, and ASA validation is unrelated to hydrogen-bond pattern
  recognition (turns, bridges, helices, ladders)
- The structure already carries annotated ASA values in a trusted DSSP entry from
  the Protein Data Bank, so recomputation and benchmarking are unnecessary
- You need dynamic/time-averaged solvent exposure from an ensemble or MD trajectory;
  this skill validates static ASA from a single set of atomic coordinates
edam_topics:
- http://edamontology.org/topic_0081
tools:
- name: DSSP
  role: Reference definition of solvent exposure (number of water molecules in possible
    contact with a residue) and the database of DSSP assignments the benchmark values
    come from
  repo: https://github.com/PDB-REDO/dssp
- name: Python
  role: Implements the geodesic sphere integration (icosahedron subdivision, occlusion
    tests, per-residue summation) and writes asa_validation.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/bip.360221211
    title: 'Dictionary of protein secondary structure: Pattern recognition of hydrogen‐bonded
      and geometrical features'
  source_content_hash: sha256:0fd5331984889d5dfa6a2c0ff053435bd2acb89bc9256335d46d115fa3044c0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PDB-REDO/dssp
  tools:
  - DSSP
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# asa-benchmark-validation-against-reference

## Summary

Validates a geodesic-sphere solvent-accessible-surface-area (ASA) implementation by comparing per-residue areas against the published Lee and Richards values for myoglobin, using graded tolerances tied to test-point density (<=4 A^2 at 80 points, <=1 A^2 at 320 points).

## When to use

Use when you have reimplemented or modified a solvent-exposure calculation (water molecules in possible contact per residue) and need to prove numerical correctness against published reference ASA values.

## When NOT to use

- The goal is secondary structure assignment, not solvent exposure — DSSP does not predict secondary structure, and ASA validation is unrelated to hydrogen-bond pattern recognition (turns, bridges, helices, ladders)
- The structure already carries annotated ASA values in a trusted DSSP entry from the Protein Data Bank, so recomputation and benchmarking are unnecessary
- You need dynamic/time-averaged solvent exposure from an ensemble or MD trajectory; this skill validates static ASA from a single set of atomic coordinates

## Inputs

- Atomic coordinate file (e.g., PDB 1MBN myoglobin) with atomic positions and residue grouping
- Set of van der Waals radii for the atom types present
- Published Lee and Richards per-residue ASA reference values for the benchmark protein
- Geodesic sphere test-point sets at 80-point and 320-point refinements (icosahedron subdivision)

## Outputs

- asa_validation.csv with per-residue areas at 80- and 320-point integration, per-residue deviations from reference, and pass/fail tolerance checks (<=4 A^2 at 80 points, <=1 A^2 at 320 points)

## How to apply

Parse a coordinate file (e.g., PDB 1MBN myoglobin) with Python, grouping atoms by residue. For each atom, generate test directions by recursively subdividing an icosahedron to 80 or 320 nearly equal-area triangles on a unit sphere, place the water-rolling sphere along each direction, and count directions unobstructed by neighbors using the stated van der Waals radii to get per-atom accessible area. Sum per-atom areas into per-residue solvent exposure values (DSSP reports exposure as the number of water molecules in possible contact with a residue). Run the integration at both 80-point and 320-point densities and diff the results against the published Lee and Richards residue areas, recording per-residue deviation. Accept the implementation only if all residue deviations fall within <=4 A^2 at 80 points and <=1 A^2 at 320 points; the coarser grid must also converge toward the finer-grid result, since tighter tolerance at higher refinement is the core correctness signal.

## Related tools

- **DSSP** (Reference definition of solvent exposure (number of water molecules in possible contact with a residue) and the database of DSSP assignments the benchmark values come from) — https://github.com/PDB-REDO/dssp
- **Python** (Implements the geodesic sphere integration (icosahedron subdivision, occlusion tests, per-residue summation) and writes asa_validation.csv)

## Examples

```
from mkdssp import dssp; import gzip; d = dssp(gzip.open('1cbs.cif.gz','rt').read()); print(d.statistics.residues)
```

## Evaluation signals

- Every residue's computed area deviates from the published Lee and Richards value by <=4 A^2 at 80 test points and <=1 A^2 at 320 test points
- Per-residue areas from the 320-point integration converge toward (improve upon) the 80-point result rather than diverging
- Per-atom accessible areas are non-negative and bounded by the full sphere area implied by the van der Waals radius
- asa_validation.csv contains all required columns: per-residue areas, deviations, and explicit tolerance pass/fail flags
- Residue totals in the DSSP statistics match the residue count iterated in the Python loop (dssp.statistics.residues)

## Limitations

- Static ASA from a single coordinate set ignores dynamics and alternate conformations; it reports only the number of water molecules in possible contact
- Accuracy is a function of test-point density: the 80-point grid only guarantees <=4 A^2, so it cannot substitute for the 320-point grid when <=1 A^2 is required
- DSSP's rewrite (4.6) changed output format (annotated mmCIF with _struct_conf) and needs a modern C++20 compiler; benchmarking against the original 1983 Lee and Richards values still assumes the original radii conventions

## Evidence

- [abstract] Solvent exposure is defined as water-molecule contact count (abstract): [evidence span withheld — anchor status: not_found]
- [abstract] Benchmark scope: solvent exposure compiled for 62 globular proteins: "The end result is a compilation of the primary structure, including SS bonds, secondary structure, and solvent exposure of 62 different globular proteins."
- [readme] DSSP as the source of assignments and calculated entries for PDB structures: "DSSP is a database of secondary structure assignments (and much more) for all protein entries in the Protein Data Bank (PDB). DSSP is also the program that calculates DSSP entries from PDB entries."
- [readme] Scope boundary: DSSP is not a prediction tool: "DSSP does **not** predict secondary structure."
- [methods] Concrete Python invocation for running DSSP on a structure file: "file_path = os.path.join("..", "test", "1cbs.cif.gz")
with gzip.open(file_path, "rt") as f:
    file_content = f.read()
dssp = dssp(file_content)"
