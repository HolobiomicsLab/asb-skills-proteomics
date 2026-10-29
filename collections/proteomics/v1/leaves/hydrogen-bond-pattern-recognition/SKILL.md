---
name: hydrogen-bond-pattern-recognition
description: Use when per-residue secondary structure assignments (helices, ladders,
  sheets, turns, bends) must be derived from backbone hydrogen-bond patterns in an
  mmCIF/PDB structure using DSSP.
when_to_use_negative:
- When the goal is to predict secondary structure from sequence alone — the DSSP program
  does not predict secondary structure
- When input is not a protein coordinate file with resolvable backbone atoms (mmCIF
  or PDB), since H-bond detection requires calculated backbone CO–NH energies
edam_operation: http://edamontology.org/operation_2418
edam_topics:
- http://edamontology.org/topic_0736
- http://edamontology.org/topic_2814
tools:
- name: DSSP
  role: Computes backbone hydrogen bonds and defines the standard secondary structure
    assignment from elementary H-bonding patterns
  repo: https://github.com/PDB-REDO/dssp
- name: mkdssp
  role: Python module exposing the dssp(file_content) constructor so per-residue H-bonds,
    assignments, and statistics can be iterated in a Python script
  repo: https://github.com/PDB-REDO/dssp
- name: CMake
  role: Configures the build with -DBUILD_PYTHON_MODULE=ON to enable the Python module
  repo: https://github.com/PDB-REDO/dssp
- name: Python
  role: 'Hosts the pattern-recognition workflow: gzip-reading mmCIF, instantiating
    dssp, assembling turns/bridges into helices/ladders/sheets, and writing outputs'
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1002/bip.360221211
    title: 'Dictionary of protein secondary structure: Pattern recognition of hydrogen‐bonded
      and geometrical features'
  source_content_hash: sha256:527b069141161cd0e099811c88df88a2bf9c87df2e819a523e9bc03e1bc40368
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PDB-REDO/dssp
  tools:
  - CMake
  - DSSP
  - Python
  - mkdssp
  edam_operation: http://edamontology.org/operation_2418
  edam_topics:
  - http://edamontology.org/topic_0736
  - http://edamontology.org/topic_2814
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# hydrogen-bond-pattern-recognition

## Summary

Reproduces the Kabsch–Sander hierarchical recognition of secondary structure from backbone hydrogen bonds: elementary n-turns and bridges are detected from DSSP-computed H-bonds, then assembled into helices, ladders, and sheets for per-residue SUMMARY assignments.

## When to use

Use when per-residue secondary structure assignments (helices, ladders, sheets, turns, bends) must be derived from backbone hydrogen-bond patterns in an mmCIF/PDB structure using DSSP.

## When NOT to use

- When the goal is to predict secondary structure from sequence alone — the DSSP program does not predict secondary structure
- When input is not a protein coordinate file with resolvable backbone atoms (mmCIF or PDB), since H-bond detection requires calculated backbone CO–NH energies

## Inputs

- Gzipped mmCIF protein coordinate file (e.g. ../test/1cbs.cif.gz)
- DSSP-computed per-residue backbone hydrogen-bond records from the mkdssp Python module

## Outputs

- Per-residue SUMMARY assignments using priority order H, B, E, G, I, T, S
- Segment lists of elementary n-turns (n = 3, 4, 5), minimal 3-/4-/5-helices, parallel/antiparallel bridges, ladders, and sheets
- Per-residue records with asym_id, seq_id, compound_id, and type
- Helix/ladder length distributions and (i,i+n) H-bond type histograms (e.g. secondary_structure_statistics.csv)

## How to apply

Build DSSP 4.6 from github.com/PDB-REDO/dssp with CMake and -DBUILD_PYTHON_MODULE=ON, then install. In Python, load a gzipped mmCIF file (e.g. ../test/1cbs.cif.gz) with gzip.open in text mode and instantiate the mkdssp dssp() object on the file content. Identify elementary n-turns as Hbond(i, i+n) for n = 3, 4, 5 and classify inter-strand hydrogen bonds as parallel or antiparallel bridges; assemble consecutive turns into minimal 3-, 4-, and 5-helices, consecutive same-type bridges into ladders, and connect ladders into sheets. Emit the per-residue SUMMARY line using the priority order H, B, E, G, I, T, S, and augment with chirality (torsional handedness of four consecutive C-alpha positions), bend, and solvent exposure (number of water molecules in possible contact). Verify by printing dssp.statistics.residues and iterating residues printing asym_id, seq_id, compound_id, and type.

## Related tools

- **DSSP** (Computes backbone hydrogen bonds and defines the standard secondary structure assignment from elementary H-bonding patterns) — https://github.com/PDB-REDO/dssp
- **mkdssp** (Python module exposing the dssp(file_content) constructor so per-residue H-bonds, assignments, and statistics can be iterated in a Python script) — https://github.com/PDB-REDO/dssp
- **CMake** (Configures the build with -DBUILD_PYTHON_MODULE=ON to enable the Python module) — https://github.com/PDB-REDO/dssp
- **Python** (Hosts the pattern-recognition workflow: gzip-reading mmCIF, instantiating dssp, assembling turns/bridges into helices/ladders/sheets, and writing outputs)

## Examples

```
from mkdssp import dssp; import gzip; d = dssp(gzip.open('../test/1cbs.cif.gz', 'rt').read()); print(d.statistics.residues)
```

## Evaluation signals

- dssp.statistics.residues matches the expected residue count (e.g. cross-check against the 10,925-residue compilation over 62 globular proteins)
- Iterating over residues yields coherent asym_id, seq_id, compound_id, and type fields for every residue
- For 3PTI, H-bond recovery reaches 18 of the 18 H bonds listed by the original authors (with 11 additional assigned)
- The (i,i+n) H-bond histogram shows a dominant n = 4 peak with a surplus of weak (i,i+2) and (i,i+3) bonds
- Every emitted segment boundary is annotated with KS, AU, and LG labels in the comparison output

## Limitations

- DSSP does not predict secondary structure; assignments depend on the actual coordinate file
- Requires a modern compiler because the rewrite uses many new C++20 features
- The Python module must be explicitly enabled at build time with BUILD_PYTHON_MODULE=ON; default builds produce only the mkdssp executable
- Modern DSSP 4.6 adds Poly-Proline helices and full mmCIF support, which can differ from historical 1983-style strip-table output

## Evidence

- [abstract] Repeating turns are 'helices,' repeating bridges are 'ladders,' connected: [evidence span withheld — anchor status: not_found]
- [abstract] Local chain chirality is defined as torsional handedness of four consecutive Cα positions: [evidence span withheld — anchor status: not_found]
- [abstract] Solvent exposure is given as number of water molecules in possible contact: [evidence span withheld — anchor status: not_found]
- [readme] DSSP does not predict secondary structure: "The DSSP program was designed by _Wolfgang Kabsch_ and _Chris Sander_ to standardize secondary structure assignment."
- [readme] Build the Python module and instantiate dssp on file content: "cmake -S . -B build -DBUILD_PYTHON_MODULE=ON"
- [readme] Iterate residues printing asym_id, seq_id, compound_id, type: "for res in dssp:
    print(res.asym_id, res.seq_id, res.compound_id, res.type)"
