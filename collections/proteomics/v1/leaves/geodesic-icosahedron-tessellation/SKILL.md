---
name: geodesic-icosahedron-tessellation
description: Use when you have atomic coordinates of a protein (e.g. a PDB entry such
  as 1MBN) and need per-residue static solvent accessibility validated against published
  Lee and Richards values.
when_to_use_negative:
- Per-residue solvent accessibility is already provided in a DSSP annotation (DSSP
  output exposes it directly) — re-implementing the tessellation is redundant
- The goal is secondary structure assignment from hydrogen-bond patterns — that is
  DSSP's core function, not solvent exposure computation
- 'Predicting structure from sequence: DSSP does not predict secondary structure,
  and this skill is likewise purely coordinate-based'
edam_topics:
- http://edamontology.org/topic_0081
tools:
- name: Python
  role: Parses atomic coordinates, generates the 80/320-point geodesic test sets by
    icosahedron subdivision, performs occlusion tests, and writes asa_validation.csv
- name: DSSP
  role: The reference program whose solvent-exposure output (number of water molecules
    in possible contact per residue) this computation reproduces and validates
  repo: github.com/PDB-REDO/dssp
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/bip.360221211
    title: 'Dictionary of protein secondary structure: Pattern recognition of hydrogen‐bonded
      and geometrical features'
  source_content_hash: sha256:7cd30db2eccee39253b329f3e4cade366951e68c4ee1a9e80f1a9eedfe105662
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DSSP
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# geodesic-icosahedron-tessellation

## Summary

Computes static solvent exposure (per-residue water contact area) by rolling-probe occlusion sampling over a geodesic sphere: an icosahedron recursively subdivided into ~80 or ~320 near-equal triangular facets supplies test directions per atom, mirroring the Lee–Richards approach DSSP uses to report the number of water molecules in possible contact with each residue.

## When to use

Use when you have atomic coordinates of a protein (e.g. a PDB entry such as 1MBN) and need per-residue static solvent accessibility validated against published Lee and Richards values.

## When NOT to use

- Per-residue solvent accessibility is already provided in a DSSP annotation (DSSP output exposes it directly) — re-implementing the tessellation is redundant
- The goal is secondary structure assignment from hydrogen-bond patterns — that is DSSP's core function, not solvent exposure computation
- Predicting structure from sequence: DSSP does not predict secondary structure, and this skill is likewise purely coordinate-based

## Inputs

- Protein atomic coordinates and residue grouping (e.g. PDB entry 1MBN)
- van der Waals radii per atom type
- Sphere refinement level (80-point or 320-point icosahedron subdivision)

## Outputs

- Per-atom solvent-accessible areas
- Per-residue solvent accessibility values (number of water molecules in possible contact) for 80- and 320-point integrations
- asa_validation.csv with per-residue areas, deviations from Lee and Richards values, and tolerance checks

## How to apply

Load the coordinate file (PDB 1MBN) and parse atomic positions plus residue grouping in Python. Subdivide an icosahedron once (80 test points) or twice (320 test points) on a unit sphere; these are the probe placement directions. For each atom, place the water-rolling sphere along each test-point direction and test occlusion by neighboring atoms using the stated van der Waals radii; count unobstructed directions to get per-atom accessible area, then sum per-atom areas into per-residue values. Judge correctness by deviation from published Lee and Richards myoglobin areas: tolerance is <=4 A^2 at 80 points and <=1 A^2 at 320 points, recorded per residue in asa_validation.csv.

## Related tools

- **Python** (Parses atomic coordinates, generates the 80/320-point geodesic test sets by icosahedron subdivision, performs occlusion tests, and writes asa_validation.csv)
- **DSSP** (The reference program whose solvent-exposure output (number of water molecules in possible contact per residue) this computation reproduces and validates) — github.com/PDB-REDO/dssp

## Examples

```
from mkdssp import dssp; import gzip; d = dssp(gzip.open('../test/1cbs.cif.gz','rt').read()); print(d.statistics.residues)
```

## Evaluation signals

- Icosahedron subdivision yields exactly 80 test points at one refinement and 320 at two refinements
- Per-residue deviation from published Lee and Richards myoglobin areas is <=4 A^2 at 80 points
- Per-residue deviation from published Lee and Richards areas drops to <=1 A^2 at 320 points
- Per-atom area equals the summed contribution of unobstructed probe directions (occluded directions contribute zero)
- asa_validation.csv contains areas, deviations, and tolerance flags for every residue

## Limitations

- Static exposure only — coordinates are fixed; conformational dynamics of water contacts are not captured
- Accuracy depends on tessellation density: 80 points has a coarser <=4 A^2 tolerance than 320 points (<=1 A^2)
- Results depend on the stated van der Waals radii used for the occlusion test against the water-rolling probe

## Evidence

- [abstract] solvent exposure as water contact count: [evidence span withheld — anchor status: not_found]
- [readme] DSSP standardized assignment role: "The DSSP program was designed by _Wolfgang Kabsch_ and _Chris Sander_ to standardize secondary structure assignment."
- [readme] Python module for working with DSSP info: [evidence span withheld — anchor status: external_doc]
- [readme] DSSP does not predict: "DSSP does **not** predict secondary structure."
