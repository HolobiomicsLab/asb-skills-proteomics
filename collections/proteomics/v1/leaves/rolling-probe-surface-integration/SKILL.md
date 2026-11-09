---
name: rolling-probe-surface-integration
description: Use when reproducing DSSP static solvent exposure — number of water molecules
  in possible contact with each residue — from PDB/mmCIF atomic coordinates, or when
  benchmarking an accessible-surface-area implementation against the published Lee
  and Richards myoglobin values.
when_to_use_negative:
- 'Input is sequence-only (e.g. FASTA): geodesic integration needs 3D atomic coordinates,
  neighboring atoms, and van der Waals radii, and DSSP does not predict structure
  from sequence.'
- The goal is secondary-structure assignment (turns/bridges/helices recognized from
  hydrogen-bond patterns) rather than solvent exposure — that is separate DSSP machinery.
- Trusted per-residue exposure values already exist in the DSSP database output for
  the PDB entry — recompute only when coordinates or radii change or when benchmarking.
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Python
  role: Parses the myoglobin coordinate file and residue grouping, generates the 80/320-point
    icosahedron test sets, performs per-atom occlusion counting with the stated van
    der Waals radii, sums per-residue areas, and writes asa_validation.csv.
- name: DSSP (mkdssp 4.6)
  role: Reference implementation defining solvent exposure as the number of water
    molecules in possible contact with a residue; its Python module (built with -DBUILD_PYTHON_MODULE=ON)
    can be used to cross-check computed per-residue exposure.
  repo: https://github.com/PDB-REDO/dssp
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/bip.360221211
    title: 'Dictionary of protein secondary structure: Pattern recognition of hydrogen‐bonded
      and geometrical features'
  source_content_hash: sha256:bbabe8206b08d3df17990b96dd17e80d6033dcd8c58334c544ac56bfe3b8aae7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PDB-REDO/dssp
  tools:
  - DSSP (mkdssp 4.6)
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# rolling-probe-surface-integration

## Summary

Reconstructs DSSP-style static solvent exposure by rolling a water-probe sphere over geodesic test points (icosahedron subdivided to 80 or 320 points) on each atom, counting unobstructed directions into per-residue accessible area, and validating against the published Lee and Richards myoglobin values (tolerance <=4 A^2 at 80 points, <=1 A^2 at 320 points). DSSP reports this exposure as the number of water molecules in possible contact with a residue.

## When to use

Use when reproducing DSSP static solvent exposure — number of water molecules in possible contact with each residue — from PDB/mmCIF atomic coordinates, or when benchmarking an accessible-surface-area implementation against the published Lee and Richards myoglobin values.

## When NOT to use

- Input is sequence-only (e.g. FASTA): geodesic integration needs 3D atomic coordinates, neighboring atoms, and van der Waals radii, and DSSP does not predict structure from sequence.
- The goal is secondary-structure assignment (turns/bridges/helices recognized from hydrogen-bond patterns) rather than solvent exposure — that is separate DSSP machinery.
- Trusted per-residue exposure values already exist in the DSSP database output for the PDB entry — recompute only when coordinates or radii change or when benchmarking.

## Inputs

- PDB/mmCIF atomic coordinate file with residue grouping (validation case: PDB entry 1MBN, myoglobin)
- Van der Waals radii per atom type, as stated by the reference implementation
- Published Lee and Richards per-residue solvent accessibility values for myoglobin (validation reference)

## Outputs

- Per-atom solvent-accessible areas from the 80-point and 320-point geodesic integrations
- Per-residue solvent exposure values (number of water molecules in possible contact)
- asa_validation.csv listing per-residue areas, deviations from Lee and Richards values, and tolerance check results

## How to apply

Load a coordinate file and parse atomic positions grouped by residue (the validation case is PDB entry 1MBN, myoglobin). Generate test-point sets by recursive subdivision of an icosahedron on a unit sphere: 20 base triangles refined once yield 80 points, refined twice yield 320. For each atom, place the water-rolling sphere along each test-point direction and test occlusion by neighboring atoms using the stated van der Waals radii, counting unobstructed directions to obtain per-atom accessible area. Sum per-atom areas into per-residue accessibility for both the 80-point and 320-point integrations, then compare against the published Lee and Richards values for myoglobin, recording the deviation per residue. Write per-residue areas, deviations, and tolerance checks (<=4 A^2 at 80 points, <=1 A^2 at 320 points) to asa_validation.csv; DSSP's convention expresses exposure as the number of water molecules in possible contact with a residue.

## Related tools

- **Python** (Parses the myoglobin coordinate file and residue grouping, generates the 80/320-point icosahedron test sets, performs per-atom occlusion counting with the stated van der Waals radii, sums per-residue areas, and writes asa_validation.csv.)
- **DSSP (mkdssp 4.6)** (Reference implementation defining solvent exposure as the number of water molecules in possible contact with a residue; its Python module (built with -DBUILD_PYTHON_MODULE=ON) can be used to cross-check computed per-residue exposure.) — https://github.com/PDB-REDO/dssp

## Examples

```
from mkdssp import dssp; import gzip; d = dssp(gzip.open('../test/1cbs.cif.gz','rt').read())
```

## Evaluation signals

- Per-residue deviations from the published Lee and Richards myoglobin areas satisfy <=4 A^2 with the 80-point integration and <=1 A^2 with the 320-point integration.
- Refining the icosahedron from 80 to 320 test points keeps all per-residue areas within the stated tolerances, demonstrating convergence of the geodesic integration.
- asa_validation.csv contains, for every residue, the computed area, the deviation from the reference, and a passing tolerance check for both point counts.
- A fully buried atom returns zero unobstructed test directions (zero accessible area) after occlusion testing with the stated van der Waals radii.
- Reported exposure is expressible in DSSP's units: the number of water molecules in possible contact with a residue.

## Limitations

- Accuracy is discretization-limited: the 80-point (one-refinement) set only guarantees <=4 A^2 per residue; <=1 A^2 requires the 320-point (two-refinement) set.
- Deviations are benchmarked specifically against the Lee and Richards values for myoglobin (PDB entry 1MBN); no other reference structure is validated in this workflow.
- Areas depend on the stated van der Waals radii and the water-rolling probe convention; cross-checking via the DSSP Python module requires building PDB-REDO/dssp from source with a modern C++20 compiler.

## Evidence

- [abstract] Solvent "exposure" is given as the number of water molecules in possible contact with a residue.: [evidence span withheld — anchor status: not_found]
- [abstract] The end result is a compilation of the primary structure, including SS bonds, secondary structure, and solvent exposure of 62 different globular proteins.: "The end result is a compilation of the primary structure, including SS bonds, secondary structure, and solvent exposure of 62 different globular proteins."
- [other] Static solvent exposure is computed by geodesic sphere integration, summing over a polyhedron of 20, 80, 320, or more approximately equal triangles per atom.: [evidence span withheld — anchor status: not_found]
- [other] Write the per-residue areas, deviations, and tolerance checks (<=4 A^2 at 80 points, <=1 A^2 at 320 points) to asa_validation.csv.: [evidence span withheld — anchor status: not_found]
- [readme] Since version 4.5.2 it is possible to build a Python module to directy work with DSSP info inside your Python scripts.: [evidence span withheld — anchor status: external_doc]
- [readme] DSSP does not predict secondary structure.: "DSSP does not predict secondary structure."
