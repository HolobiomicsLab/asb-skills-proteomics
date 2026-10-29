---
name: coulombic-electrostatic-modeling
description: Use when atomic coordinates (PDB/mmCIF) must be converted into backbone
  hydrogen-bond assignments, e.g. to derive DSSP-style secondary structure, base-pair
  maps, or per-pair bond energies from N, C, O, CA coordinates.
when_to_use_negative:
- Input is already a DSSP output or a precomputed hydrogen-bond/secondary-structure
  table — recompute nothing
- The goal is secondary-structure prediction from sequence alone; DSSP explicitly
  does not predict secondary structure, it assigns from coordinates
- The structure lacks backbone amide hydrogens in positions where proline or non-standard
  residues make the geometric H reconstruction invalid
edam_topics:
- http://edamontology.org/topic_0082
tools:
- name: DSSP
  role: Reference implementation defining the hydrogen-bond electrostatic energy criterion
    and secondary-structure assignment from PDB entries
  repo: https://github.com/PDB-REDO/dssp
- name: mkdssp
  role: Modern rewrite of DSSP with full mmCIF support; can be built as a Python module
    to read coordinate files and iterate over residue assignments directly in Python
  repo: https://github.com/PDB-REDO/dssp
- name: Python
  role: Implements the geometric amide-H reconstruction and per-pair Coulomb energy
    calculation, then compiles results into hbonds.csv
- name: CMake
  role: Configures the mkdssp build with -DBUILD_PYTHON_MODULE=ON so the DSSP engine
    can be called from Python scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/bip.360221211
    title: 'Dictionary of protein secondary structure: Pattern recognition of hydrogen‐bonded
      and geometrical features'
  source_content_hash: sha256:10b5f2056ee7f1436d167d222781828bd9c0843c191be3dfcbdc9819cecfe77f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PDB-REDO/dssp
  tools:
  - CMake
  - DSSP
  - Python
  - mkdssp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0082
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# coulombic-electrostatic-modeling

## Summary

Computes the DSSP Coulomb-type electrostatic interaction energy between backbone C=O and N-H groups from partial charges and interatomic distances, classifying pairs as hydrogen bonds when E < -0.5 kcal/mol. This is the energetic criterion DSSP uses to detect hydrogen bonds that define secondary structure.

## When to use

Use when atomic coordinates (PDB/mmCIF) must be converted into backbone hydrogen-bond assignments, e.g. to derive DSSP-style secondary structure, base-pair maps, or per-pair bond energies from N, C, O, CA coordinates.

## When NOT to use

- Input is already a DSSP output or a precomputed hydrogen-bond/secondary-structure table — recompute nothing
- The goal is secondary-structure prediction from sequence alone; DSSP explicitly does not predict secondary structure, it assigns from coordinates
- The structure lacks backbone amide hydrogens in positions where proline or non-standard residues make the geometric H reconstruction invalid

## Inputs

- Compressed PDB or mmCIF coordinate files (e.g. 3PTI, 155C, 2ADK, 1cbs.cif.gz)
- Per-residue backbone atom coordinates for N, C, O, and CA
- Residue identities (to exclude proline amides)

## Outputs

- Per-pair hydrogen-bond assignments Hbond(i,j) with donor residue, acceptor residue, and computed energy E
- A combined table such as hbonds.csv
- Optionally, DSSP secondary structure annotations derived from the bond patterns

## How to apply

Load compressed PDB/mmCIF coordinate files (e.g. 3PTI, 155C, 2ADK) and extract backbone N, C, O, and CA coordinates for every residue. Reconstruct the amide hydrogen position geometrically from the N, C, and CA atom positions (skip proline residues, which lack an amide H). For every C=O/N-H donor-acceptor pair compute E = q1*q2*f*(1/r(ON) + 1/r(CH) - 1/r(OH) - 1/r(CN)) with q1=0.42e, q2=0.20e, and f=332 kcal·Å·mol⁻¹·e⁻². Apply the assignment rule Hbond(i,j) := [E < -0.5 kcal/mol] to classify each pair, then compile donor residue, acceptor residue, and energy into a combined table (e.g. hbonds.csv). The rationale follows Kabsch & Sander: cooperative secondary structure is recognized as repeats of the elementary hydrogen-bonding patterns 'turn' and 'bridge', so the electrostatic criterion is the foundation for helix and ladder assignment.

## Related tools

- **DSSP** (Reference implementation defining the hydrogen-bond electrostatic energy criterion and secondary-structure assignment from PDB entries) — https://github.com/PDB-REDO/dssp
- **mkdssp** (Modern rewrite of DSSP with full mmCIF support; can be built as a Python module to read coordinate files and iterate over residue assignments directly in Python) — https://github.com/PDB-REDO/dssp
- **Python** (Implements the geometric amide-H reconstruction and per-pair Coulomb energy calculation, then compiles results into hbonds.csv)
- **CMake** (Configures the mkdssp build with -DBUILD_PYTHON_MODULE=ON so the DSSP engine can be called from Python scripts)

## Examples

```
cmake -S . -B build -DBUILD_PYTHON_MODULE=ON && cmake --build build && sudo cmake --install build; from mkdssp import dssp; d = dssp(open('1cbs.cif').read())
```

## Evaluation signals

- Energy for a true C=O(i)/N-H(j) hydrogen bond falls below the -0.5 kcal/mol cutoff; non-bonded pairs stay above it
- Computed distances r(ON), r(CH), r(OH), r(CN) are physically plausible (roughly 2-4 Å for the H-bond O...N contact)
- Proline residues produce no amide-H-based assignments
- Cross-check per-pair energies against mkdssp output on the same mmCIF input; secondary-structure annotations should match
- hbonds.csv schema contains donor residue, acceptor residue, and energy for every entry processed

## Limitations

- Proline residues lack an amide H and must be skipped when reconstructing donor geometry
- The model uses fixed partial charges (q1=0.42e, q2=0.20e) and a single hard cutoff (-0.5 kcal/mol), not a full force field
- DSSP does not predict secondary structure — it requires solved coordinates; the 4.x rewrite stores assignments in the mmCIF _struct_conf category

## Evidence

- [abstract] Hydrogen bond energy assignment rule: [evidence span withheld — anchor status: not_found]
- [readme] DSSP does not predict secondary structure: "The DSSP program was designed by _Wolfgang Kabsch_ and _Chris Sander_ to standardize secondary structure assignment."
- [methods] Run DSSP from Python on coordinate content: "for res in dssp:
    print(res.asym_id, res.seq_id, res.compound_id, res.type)"
- [methods] Build with Python module enabled: "cmake -S . -B build -DBUILD_PYTHON_MODULE=ON"
