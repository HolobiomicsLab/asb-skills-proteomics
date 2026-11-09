---
name: secondary-structure-hbond-annotation
description: Use when you have atomic coordinates (PDB or mmCIF, optionally gzipped)
  for backbone N, C, O, CA atoms and need to classify donor–acceptor pairs as hydrogen
  bonds via the DSSP E < -0.5 kcal/mol energy cutoff, e.g. to annotate secondary structure.
when_to_use_negative:
- When the goal is to predict secondary structure from sequence alone — the DSSP program
  explicitly 'does not predict secondary structure'; it assigns from coordinates
- When the structural input contains no backbone amide hydrogens and only proline-like
  residues of interest, since prolines lack an amide H and are skipped in the geometric
  H reconstruction
- When a hydrogen-bond definition other than the DSSP Coulomb-type energy cutoff (E
  < -0.5 kcal/mol) is required
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: DSSP
  role: Original Kabsch–Sander algorithm defining hydrogen-bonded secondary-structure
    features; computes the electrostatic H-bond energy used for assignment
  repo: https://github.com/PDB-REDO/dssp
- name: mkdssp
  role: Modern C++20 rewrite (DSSP 4.6) with full mmCIF support; runs the assignment
    and writes annotated mmCIF with _struct_conf secondary structure
  repo: https://github.com/PDB-REDO/dssp
- name: Python
  role: Python module (since DSSP 4.5.2) to load gzipped mmCIF content and iterate
    residues/asym_id/seq_id/compound_id/type directly in scripts
  repo: https://github.com/PDB-REDO/dssp
- name: CMake
  role: Configures and builds mkdssp and its Python module (-DBUILD_PYTHON_MODULE=ON)
  repo: https://github.com/PDB-REDO/dssp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/bip.360221211
    title: 'Dictionary of protein secondary structure: Pattern recognition of hydrogen‐bonded
      and geometrical features'
  source_content_hash: sha256:761c099d451c6e46c1c30876e9a94fe76ac3c06e8bf4c85427bf8f29eef95efa
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# secondary-structure-hbond-annotation

## Summary

Computes the DSSP electrostatic hydrogen-bond energy E between backbone C=O acceptors and N-H donors from atomic coordinates and assigns a hydrogen bond when E < -0.5 kcal/mol, forming the elementary 'turn'/'bridge' patterns that Kabsch & Sander's DSSP dictionary uses to define secondary structure.

## When to use

Use when you have atomic coordinates (PDB or mmCIF, optionally gzipped) for backbone N, C, O, CA atoms and need to classify donor–acceptor pairs as hydrogen bonds via the DSSP E < -0.5 kcal/mol energy cutoff, e.g. to annotate secondary structure.

## When NOT to use

- When the goal is to predict secondary structure from sequence alone — the DSSP program explicitly 'does not predict secondary structure'; it assigns from coordinates
- When the structural input contains no backbone amide hydrogens and only proline-like residues of interest, since prolines lack an amide H and are skipped in the geometric H reconstruction
- When a hydrogen-bond definition other than the DSSP Coulomb-type energy cutoff (E < -0.5 kcal/mol) is required

## Inputs

- Compressed PDB or mmCIF coordinate files (e.g. .pdb.gz, .cif.gz) containing backbone N, C, O, CA atom coordinates per residue
- Partial-charge parameters q1=0.42e, q2=0.20e and f=332 kcal·Å·mol⁻¹·e⁻²
- Energy cutoff of -0.5 kcal/mol for the assignment rule

## Outputs

- Per-pair donor residue, acceptor residue, and computed energy E (kcal/mol) for each classified C=O/N-H pair
- Combined table of assigned hydrogen bonds across entries, e.g. hbonds.csv
- Secondary-structure assignment (annotated mmCIF _struct_conf categories) when run through mkdssp

## How to apply

Load the compressed PDB/mmCIF coordinate files (e.g. 3PTI, 155C, 2ADK) and extract backbone N, C, O, and CA coordinates for every residue. Reconstruct each amide hydrogen position geometrically from N, C, and CA (skipping prolines, which lack an amide H). For every C=O/N-H donor–acceptor pair compute the Coulomb-type energy E = q1*q2*f*(1/r(ON) + 1/r(CH) - 1/r(OH) - 1/r(CN)) with q1=0.42e, q2=0.20e, and f=332 kcal·Å·mol⁻¹·e⁻², then apply the rule Hbond(i,j) := E < -0.5 kcal/mol. Alternatively, build the PDB-REDO DSSP rewrite with CMake (enabling -DBUILD_PYTHON_MODULE=ON) and run mkdssp/dssp directly on the file content, since repeating 'turn' and 'bridge' hydrogen-bonding patterns are what DSSP recognizes as helices, ladders, and sheets.

## Related tools

- **DSSP** (Original Kabsch–Sander algorithm defining hydrogen-bonded secondary-structure features; computes the electrostatic H-bond energy used for assignment) — https://github.com/PDB-REDO/dssp
- **mkdssp** (Modern C++20 rewrite (DSSP 4.6) with full mmCIF support; runs the assignment and writes annotated mmCIF with _struct_conf secondary structure) — https://github.com/PDB-REDO/dssp
- **Python** (Python module (since DSSP 4.5.2) to load gzipped mmCIF content and iterate residues/asym_id/seq_id/compound_id/type directly in scripts) — https://github.com/PDB-REDO/dssp
- **CMake** (Configures and builds mkdssp and its Python module (-DBUILD_PYTHON_MODULE=ON)) — https://github.com/PDB-REDO/dssp

## Examples

```
with gzip.open('3ptb.cif.gz', 'rt') as f: dssp = dssp(f.read())
for res in dssp: print(res.asym_id, res.seq_id, res.compound_id, res.type)
```

## Evaluation signals

- Every reported hydrogen bond satisfies E < -0.5 kcal/mol under E = q1*q2*f*(1/r(ON) + 1/r(CH) - 1/r(OH) - 1/r(CN)) with the stated charges and f factor
- Proline residues are excluded as amide-H donors (no geometrically reconstructed H), so no donor entries should reference proline amide H
- Secondary-structure labels derived from the bonds follow the dictionary logic: repeating turns are helices, repeating bridges are ladders, connected ladders are sheets
- When using mkdssp 4.6 on mmCIF input, the output annotated mmCIF contains secondary structure in the _struct_conf category
- Per-residue iteration via the Python module returns residues with non-empty asym_id, seq_id, compound_id, and type fields consistent with the input PDB/mmCIF

## Limitations

- Proline residues lack an amide H and must be skipped in the geometric H reconstruction
- DSSP requires a good, modern compiler supporting C++20 features; older toolchains may fail to build mkdssp
- DSSP 4.x default output is annotated mmCIF (secondary structure in _struct_conf), which may differ from legacy .dssp text output consumers
- The energy uses reconstructed amide H positions, so H coordinates are geometry-derived, not experimentally observed

## Evidence

- [methods] Hbond(i,j) := [E < -0.5 kcal/mol]: [evidence span withheld — anchor status: not_found]
- [abstract] Cooperative secondary structure is recognized as repeats of the elementary hydrogen‐bonding patterns: [evidence span withheld — anchor status: not_found]
- [readme] DSSP does not predict secondary structure: "The DSSP program was designed by _Wolfgang Kabsch_ and _Chris Sander_ to standardize secondary structure assignment. DSSP is a database of secondary structure assignments"
- [readme] Python module for DSSP: [evidence span withheld — anchor status: external_doc]
- [intro] mmCIF _struct_conf output: "it now writes out an annotated mmCIF file by default, storing the secondary structure information in the `_struct_conf` category."
