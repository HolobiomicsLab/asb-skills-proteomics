---
name: dssp-strip-table-generation
description: Use when you need per-residue standardized secondary-structure assignments
  (helices, ladders, sheets, turns, bends, chirality, exposure) computed from protein
  coordinate files and rendered as linear strip tables.
when_to_use_negative:
- DSSP does not predict secondary structure — do not use it for prediction from sequence;
  it requires 3D coordinate files with hydrogen-bond-capable geometry
- Input structures are not in mmCIF format and cannot be converted — the rewritten
  DSSP 4.6 offers full mmCIF support and writes annotated mmCIF output
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_0736
- http://edamontology.org/topic_0080
tools:
- name: DSSP
  role: Core algorithm that computes backbone hydrogen-bond patterns, elementary turns
    and bridges, and secondary-structure assignments
  repo: https://github.com/PDB-REDO/dssp
- name: mkdssp
  role: Python module used to run dssp(file_content) on mmCIF file content and iterate
    over per-residue records
  repo: https://github.com/PDB-REDO/dssp
- name: CMake
  role: Build system used to configure and compile DSSP with the Python module enabled
    (-DBUILD_PYTHON_MODULE=ON)
  repo: https://github.com/PDB-REDO/dssp
- name: Python
  role: Scripting language used to read gzipped mmCIF files, invoke mkdssp, and assemble
    the strip tables
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/bip.360221211
    title: 'Dictionary of protein secondary structure: Pattern recognition of hydrogen‐bonded
      and geometrical features'
  source_content_hash: sha256:93085e814310f4df65766d934a51cf287126656bf58bcb16d085473d1d9c6ccf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PDB-REDO/dssp
  tools:
  - CMake
  - DSSP
  - Python
  - mkdssp
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_0736
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# dssp-strip-table-generation

## Summary

Generate DSSP strip tables — linear per-residue records of secondary structure, hydrogen-bonding, chirality, bend, and solvent exposure — for a set of globular proteins, reproducing the format of the Kabsch & Sander dictionary covering 62 proteins / 10,925 residues.

## When to use

Use when you need per-residue standardized secondary-structure assignments (helices, ladders, sheets, turns, bends, chirality, exposure) computed from protein coordinate files and rendered as linear strip tables.

## When NOT to use

- DSSP does not predict secondary structure — do not use it for prediction from sequence; it requires 3D coordinate files with hydrogen-bond-capable geometry
- Input structures are not in mmCIF format and cannot be converted — the rewritten DSSP 4.6 offers full mmCIF support and writes annotated mmCIF output

## Inputs

- mmCIF protein coordinate files (e.g. 1cbs.cif.gz), possibly gzip-compressed
- Set of 62 globular protein structures from the Protein Data Bank

## Outputs

- Strip tables (SHEET/BRIDGE/CHIRALITY/BEND/turn lines/SUMMARY/EXPOSURE/SEQUENCE lines) covering all 10,925 residues of the 62 proteins, written to dssp_strip_tables.txt
- Per-residue records with asym_id, seq_id, compound_id, type, secondary-structure state, chirality, bend, and solvent exposure

## How to apply

Build the PDB-REDO DSSP rewrite (v4.6) with CMake using -DBUILD_PYTHON_MODULE=ON and install with 'cmake --install build' so the mkdssp Python module is available. For each of the 62 mmCIF protein coordinate files, decompress gzipped files with gzip.open in text mode and pass the file content to dssp(), which computes backbone hydrogen-bond patterns, derives elementary turns and bridges, and assembles them into helices, ladders, and sheets. Iterate over the dssp object reading asym_id, seq_id, compound_id, and type per residue, and record local chain chirality (torsional handedness of four consecutive C-alpha positions), bend, and solvent exposure as the number of water molecules in possible contact. Assemble the per-residue records into strip tables (SHEET/BRIDGE/CHIRALITY/BEND/turn lines/SUMMARY/EXPOSURE/SEQUENCE lines) and write them to dssp_strip_tables.txt. Verify by checking dssp.statistics.residues totals against the expected residue count for the protein set.

## Related tools

- **DSSP** (Core algorithm that computes backbone hydrogen-bond patterns, elementary turns and bridges, and secondary-structure assignments) — https://github.com/PDB-REDO/dssp
- **mkdssp** (Python module used to run dssp(file_content) on mmCIF file content and iterate over per-residue records) — https://github.com/PDB-REDO/dssp
- **CMake** (Build system used to configure and compile DSSP with the Python module enabled (-DBUILD_PYTHON_MODULE=ON)) — https://github.com/PDB-REDO/dssp
- **Python** (Scripting language used to read gzipped mmCIF files, invoke mkdssp, and assemble the strip tables)

## Examples

```
from mkdssp import dssp; import gzip; f = gzip.open('../test/1cbs.cif.gz', 'rt'); d = dssp(f.read()); print('residues: ', d.statistics.residues)
```

## Evaluation signals

- dssp.statistics.residues matches the expected total of 10,925 residues across the 62 globular proteins
- Per-residue records contain asym_id, seq_id, compound_id, and type fields for every residue iterated from the dssp object
- Strip tables include all expected line classes: SHEET/BRIDGE, CHIRALITY, BEND, turn lines, SUMMARY, EXPOSURE, and SEQUENCE
- Chirality values follow the defined convention: positive for right-handed helices and negative for ideal twisted beta-sheets
- Solvent exposure values are expressed as the number of water molecules in possible contact with each residue

## Limitations

- DSSP does not predict secondary structure; it only standardizes assignment from experimental coordinates
- A modern compiler is required to build mkdssp because it uses many new C++20 features
- DSSP 4.6 writes an annotated mmCIF file by default (secondary structure in the _struct_conf category); consuming strip-table output requires the Python iteration workflow instead

## Evidence

- [abstract] The dictionary output is presented in linear form, with strip tables giving the detailed assignments for each of the 10,925 residues of the 62 globular proteins.: [evidence span withheld — anchor status: not_found]
- [abstract] Cooperative secondary structure recognized from hydrogen-bonding patterns.: [evidence span withheld — anchor status: not_found]
- [readme] Python module invocation from README.: "with gzip.open(file_path, "rt") as f:
    file_content = f.read()
 
dssp = dssp(file_content)"
- [readme] Build configuration for the Python module.: "cmake -S . -B build -DBUILD_PYTHON_MODULE=ON"
- [abstract] Definition of local chain chirality.: [evidence span withheld — anchor status: not_found]
- [abstract] Definition of solvent exposure.: [evidence span withheld — anchor status: not_found]
