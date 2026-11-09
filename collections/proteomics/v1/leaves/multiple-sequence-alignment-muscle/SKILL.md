---
name: multiple-sequence-alignment-muscle
description: Use when a FASTA set of sequence variants (or UniProt/Ensembl variant
  data fetched by UniProtID) must be aligned per-position to compute consensus, conservation,
  entropy, or SNP annotations for 3D structure mapping.
when_to_use_negative:
- When the goal is a direct structure-only annotation without any sequence-variant
  analysis — scop3D's structure step still requires the preceding sequence step to
  have produced consensus.fasta.
- When the input is a single protein structure with no variant/sequence set available
  and no UniProtID from which to download variant data.
- When the analysis targets SNP calling from raw sequencing reads rather than aligning
  already-collected variant sequences.
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: MUSCLE
  role: Aligns the collected variant sequences into a multiple sequence alignment
    from which the consensus and per-position statistics are derived
  repo: github.com/compomics/scop3d
- name: scop3D
  role: Wraps the MUSCLE alignment within its sequence conservation analysis ('./scop3d
    sequence') and sequence SNP variants analysis ('./scop3d sequence -var') commands
  repo: github.com/compomics/scop3d
- name: Biopython
  role: Runtime dependency used by scop3D for sequence handling during the alignment/consensus
    steps
  repo: github.com/compomics/scop3d
- name: Bioperl
  role: Runtime dependency (with Ensembl Perl API) supporting retrieval of UniProt/Ensembl
    variant data feeding the alignment
  repo: github.com/compomics/scop3d
- name: WebLogo
  role: Generates the sequence logo from the aligned variant positions after MUSCLE
    alignment
  repo: github.com/compomics/scop3d
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201400354
    title: 'Scop3D: Three‐dimensional visualization of sequence conservation'
  source_content_hash: sha256:2296d93a7258c5f90d3d98fce6265f68037ae565fe89f8d9a953f77618e8ae56
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bioperl
  - Biopython
  - MUSCLE
  - Scop3D
  - WebLogo
  - scop3D
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 21
---

# multiple-sequence-alignment-muscle

## Summary

Align protein or DNA sequence variants with MUSCLE as the core alignment step of scop3D's sequence conservation and SNP variants analyses, enabling per-position conservation/entropy or SNP frequency/type annotation to be mapped back onto a PDB structure.

## When to use

Use when a FASTA set of sequence variants (or UniProt/Ensembl variant data fetched by UniProtID) must be aligned per-position to compute consensus, conservation, entropy, or SNP annotations for 3D structure mapping.

## When NOT to use

- When the goal is a direct structure-only annotation without any sequence-variant analysis — scop3D's structure step still requires the preceding sequence step to have produced consensus.fasta.
- When the input is a single protein structure with no variant/sequence set available and no UniProtID from which to download variant data.
- When the analysis targets SNP calling from raw sequencing reads rather than aligning already-collected variant sequences.

## Inputs

- FASTA file containing all sequence variants of interest (e.g. sequences.fasta)
- Variant data downloaded from UniProt and Ensembl databases using a UniProtID (e.g. P06858)
- User-set conservation threshold and, for BLAST-retrieved variants, an identity cutoff (e.g. -t 50, -t 70)

## Outputs

- Multiple sequence alignment of variant sequences
- consensus.fasta written to the output directory (usable as BLAST input)
- Text files with absolute and relative amino acid occurrences per position
- Matrix annotating the protein sequence with frequency, variants, and type of each variant
- Sequence logo created with WebLogo

## How to apply

Feed scop3D a FASTA file of all sequence variants of interest (or retrieve variants from UniProt and Ensembl using the protein's UniProtID, e.g. P06858), then align the collected sequences with MUSCLE. From the resulting multiple sequence alignment, construct the consensus by retaining the most frequently found residue per position, kept only when its abundance exceeds the user-set conservation threshold (e.g. supplied via -t alongside an identity cutoff used when BLAST-retrieved variants are used). In the SNP variants mode, align the sequences and compare them to spot SNPs, and additionally translate and align them to obtain a consensus protein sequence. Build absolute and relative amino-acid occurrence matrices and a matrix annotating the protein sequence with the frequency, variants, and type of each variant, and write the consensus to consensus.fasta as BLAST input for the subsequent structure-annotation step.

## Related tools

- **MUSCLE** (Aligns the collected variant sequences into a multiple sequence alignment from which the consensus and per-position statistics are derived) — github.com/compomics/scop3d
- **scop3D** (Wraps the MUSCLE alignment within its sequence conservation analysis ('./scop3d sequence') and sequence SNP variants analysis ('./scop3d sequence -var') commands) — github.com/compomics/scop3d
- **Biopython** (Runtime dependency used by scop3D for sequence handling during the alignment/consensus steps) — github.com/compomics/scop3d
- **Bioperl** (Runtime dependency (with Ensembl Perl API) supporting retrieval of UniProt/Ensembl variant data feeding the alignment) — github.com/compomics/scop3d
- **WebLogo** (Generates the sequence logo from the aligned variant positions after MUSCLE alignment) — github.com/compomics/scop3d

## Examples

```
./scop3d sequence -ent -w ~/outputdir -f sequences.fasta
```

## Evaluation signals

- consensus.fasta is present in the output directory after the sequence step, containing the most-frequently-found residue per position where its abundance exceeds the user-defined conservation threshold.
- For SNP variants mode, the variant matrix annotates the protein sequence with the frequency, variants, and type of each variant.
- Two matrices exist: one with absolute amino-acid abundance per position, one with relative abundances in percent.
- The consensus sequence BLASTs successfully against the PDB to identify a matching structure (e.g. 4F15 or 4ACQ), confirming the alignment produced a usable consensus.
- The subsequent structure step produces exactly two modified PDB files whose B-factor columns carry percent-variation/entropy (conservation mode) or SNP frequency/SNP type (variants mode) for every annotated residue.

## Limitations

- The consensus residue is only retained when found more than the user-set conservation threshold, so positions below threshold yield no consensus residue.
- SNP variants analysis from a single DNA sequence requires BLAST against NCBI with an identity cutoff to obtain multiple sequences; a lone unblasted DNA sequence is insufficient.
- Subsequent scop3D runs search for input files in the -w output directory, so all inputs/outputs must be bundled in one directory.
- Python >= 2.7, Perl >= 5, Biopython, Bioperl, Ensembl Perl API, Weblogo, Muscle, and Emboss are required system dependencies.

## Evidence
