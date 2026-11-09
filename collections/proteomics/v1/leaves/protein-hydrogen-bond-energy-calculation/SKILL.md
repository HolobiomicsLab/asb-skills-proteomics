---
name: protein-hydrogen-bond-energy-calculation
description: Use when backbone N, C, O, CA coordinates are available (e.g. PDB/mmCIF
  files like 3PTI, 155C, 2ADK) and donor-acceptor hydrogen bonds must be classified
  quantitatively by the DSSP energy rule.
when_to_use_negative:
- When the goal is secondary structure *prediction* from sequence — DSSP does not
  predict, it assigns from coordinates ('DSSP does not predict secondary structure')
- When solvent-side-chain or ligand-mediated hydrogen bonds are needed — this skill
  covers only backbone C=O / N-H pairs from the energy rule
- When proline residues are involved as donors — prolines lack an amide H and must
  be skipped
edam_topics:
- http://edamontology.org/topic_0082
tools:
- name: DSSP
  role: Original algorithm and program that defines the electrostatic hydrogen-bond
    energy rule and secondary structure assignment
  repo: https://github.com/PDB-REDO/dssp
- name: mkdssp
  role: Modern DSSP 4.6 rewrite with full mmCIF support; built via CMake and usable
    as a Python module to obtain DSSP info in scripts
  repo: https://github.com/PDB-REDO/dssp
- name: Python
  role: Loads gzipped coordinate files, reads DSSP output via the mkdssp module, and
    computes per-pair Coulomb energies into hbonds.csv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/bip.360221211
    title: 'Dictionary of protein secondary structure: Pattern recognition of hydrogen‐bonded
      and geometrical features'
  source_content_hash: sha256:2bde6e7e031522fa1a896277bbea24bca40f9d6316d97245642abd11cfe6237b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PDB-REDO/dssp
  tools:
  - DSSP
  - Python
  - mkdssp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0082
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# protein-hydrogen-bond-energy-calculation

## Summary

Implements the DSSP electrostatic hydrogen-bond definition: a Coulomb-type interaction energy E is computed for every backbone C=O/N-H pair from partial charges and a hydrogen bond is assigned when E < -0.5 kcal/mol. This energy rule is the basis of all DSSP secondary structure assignment.

## When to use

Use when backbone N, C, O, CA coordinates are available (e.g. PDB/mmCIF files like 3PTI, 155C, 2ADK) and donor-acceptor hydrogen bonds must be classified quantitatively by the DSSP energy rule.

## When NOT to use

- When the goal is secondary structure *prediction* from sequence — DSSP does not predict, it assigns from coordinates ('DSSP does not predict secondary structure')
- When solvent-side-chain or ligand-mediated hydrogen bonds are needed — this skill covers only backbone C=O / N-H pairs from the energy rule
- When proline residues are involved as donors — prolines lack an amide H and must be skipped

## Inputs

- Compressed PDB coordinate files (e.g. 3PTI, 155C, 2ADK) or gzipped mmCIF files
- Backbone atom coordinates (N, C, O, CA) per residue

## Outputs

- List of assigned hydrogen bonds with donor residue, acceptor residue, and computed electrostatic energy E (kcal/mol)
- Combined tabular file such as hbonds.csv
- Derived secondary structure elements (turns, bridges, helices, ladders, sheets)

## How to apply

Load the compressed PDB/mmCIF coordinate files and extract backbone N, C, O, and CA atom coordinates for every residue. Reconstruct the amide hydrogen geometrically from the N, C, and CA positions for each backbone amide, skipping prolines which lack an amide H. For every C=O (residue i) / N-H (residue j) pair compute E = q1*q2*f*(1/r(ON) + 1/r(CH) - 1/r(OH) - 1/r(CN)) with donor charge q1 = 0.42e, acceptor charge q2 = 0.20e, and f = 332 kcal·Å·mol⁻¹·e⁻². Assign Hbond(i,j) := [E < -0.5 kcal/mol]; the rationale is that DSSP defines secondary structure ('turns' and 'bridges') purely from these hydrogen-bond patterns. Compile donor residue, acceptor residue, and computed energy per entry into a combined table (e.g. hbonds.csv).

## Related tools

- **DSSP** (Original algorithm and program that defines the electrostatic hydrogen-bond energy rule and secondary structure assignment) — https://github.com/PDB-REDO/dssp
- **mkdssp** (Modern DSSP 4.6 rewrite with full mmCIF support; built via CMake and usable as a Python module to obtain DSSP info in scripts) — https://github.com/PDB-REDO/dssp
- **Python** (Loads gzipped coordinate files, reads DSSP output via the mkdssp module, and computes per-pair Coulomb energies into hbonds.csv)

## Examples

```
from mkdssp import dssp; import gzip; file_content = gzip.open('test/1cbs.cif.gz','rt').read(); d = dssp(file_content)
```

## Evaluation signals

- Every assigned bond satisfies E < -0.5 kcal/mol; no pair above the cutoff is labeled a hydrogen bond
- Energies computed with q1=0.42e, q2=0.20e, f=332 kcal·Å·mol⁻¹·e⁻² reproduce the DSSP Coulomb expression on a known test structure (e.g. PDB 1CBS via the mkdssp test file)
- No proline residue appears as a hydrogen-bond donor in the output table
- hbonds.csv contains donor residue, acceptor residue, and energy columns for each entry (3PTI, 155C, 2ADK)
- Aggregated hydrogen-bond patterns recapitulate DSSP 'turn'/'bridge' elements, i.e. helices from repeating turns and ladders from repeating bridges

## Limitations

- Proline residues must be skipped as donors because they lack an amide H
- The rule depends on reconstructed amide hydrogen positions, so coordinate errors or missing backbone atoms (N, C, CA) invalidate the geometric reconstruction
- DSSP 4.x (the rewrite) has no changelog and stores assignments in the _struct_conf category of annotated mmCIF output; older DSSP-style output must be handled separately

## Evidence

- [abstract] Cooperative secondary structure is recognized as repeats of the elementary hydrogen-bonding patterns "turn" and "bridge.": [evidence span withheld — anchor status: not_found]
- [abstract] The computer program DSSP (Define Secondary Structure of Proteins) written in standard PASCAL will be available from the Protein Data Bank: "The computer program DSSP (Define Secondary Structure of Proteins) written in standard PASCAL will be available from the Protein Data Bank"
- [readme] The DSSP program was designed by Wolfgang Kabsch and Chris Sander to standardize secondary structure assignment.: "The DSSP program was designed by _Wolfgang Kabsch_ and _Chris Sander_ to standardize secondary structure assignment."
- [readme] DSSP does not predict secondary structure.: "DSSP does **not** predict secondary structure."
- [methods] Read gzipped mmCIF content and run DSSP in Python: "with gzip.open(file_path, "rt") as f:
    file_content = f.read()"
