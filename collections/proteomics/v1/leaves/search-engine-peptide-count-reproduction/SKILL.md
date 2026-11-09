---
name: search-engine-peptide-count-reproduction
description: Use when reproducing or validating published immunopeptidomics peptide
  identification gains — e.g., comparing per-engine unique peptide counts before and
  after deep learning-based rescoring on datasets like PRIDE PXD004894 or MassIVE
  MSV000084442.
when_to_use_negative:
- Data comes from tryptic-digest shotgun proteomics rather than MHC-bound peptide
  immunopeptidomics — the unspecific-cleavage search space and DeepRescore feature
  design target HLA peptide data
- You need per-peptide binding-affinity annotation or neoantigen prediction rather
  than a reproduction of identification counts — that requires additional steps (netMHCpan
  percentile rank <= 2%) outside this skill
- Input identifications are already rescored feature-level outputs (e.g., existing
  Percolator q-value tables) rather than raw search-engine PSM results
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Comet
  role: 'Search engine #1; produces PSMs rescored by DeepRescore; baseline D1 count
    23400 unique peptides'
- name: MS-GF+
  role: 'Search engine #2; Evalue-based PSM output filtered via PGA target-decoy at
    1% FDR'
  repo: https://github.com/MSGFPlus/msgfplus
- name: X!Tandem
  role: 'Search engine #3 producing one of four parallel PSM sets'
- name: MaxQuant
  role: 'Search engine #4; PEP-scored PSMs used for FDR filtering and rescoring'
- name: ProteoWizard
  role: Converts raw MS/MS files to MGF with peak picking before searching
- name: Customprodbj
  role: Builds the D1 customized database (RefSeq + variant proteins + contaminants),
    automated via NeoFlow
  repo: https://github.com/bzhanglab/customprodbj
- name: NeoFlow
  role: Automates variant annotation and customized database construction for D1
  repo: https://github.com/bzhanglab/neoflow
- name: PGA
  role: Implements target-decoy FDR filtering at 1% at PSM and peptide levels
- name: DeepRescore
  role: Rescores PSMs from all four engines, adding RTdelta (AutoRT) and spectral
    angle (pDeep2) features via Percolator
  repo: https://github.com/bzhanglab/DeepRescore
- name: AutoRT
  role: Transfer-learning RT prediction per LC-MS/MS run yielding the RTdelta feature
  repo: https://github.com/bzhanglab/AutoRT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:8645c1b482ec8f3e857ae09508b8cfd4875c13b4adb6c0288316d5ae9bb80e19
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MSGFPlus/msgfplus
  tools:
  - ANNOVAR
  - AutoRT
  - Comet
  - Customprodbj
  - DeepRescore
  - Docker
  - MS-GF+
  - MaxQuant
  - NeoFlow
  - NextFlow
  - PGA
  - ProteoWizard
  - X!Tandem
  - pDeep2
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# search-engine-peptide-count-reproduction

## Summary

A benchmark-reproduction skill for immunopeptidomics: search MHC peptidomics MS/MS data with four engines (Comet, MS-GF+, X!Tandem, MaxQuant), filter at 1% FDR, then rescore with DeepRescore and compare unique peptide counts against the published D1/D2 baseline and post-rescoring numbers.

## When to use

Use when reproducing or validating published immunopeptidomics peptide identification gains — e.g., comparing per-engine unique peptide counts before and after deep learning-based rescoring on datasets like PRIDE PXD004894 or MassIVE MSV000084442.

## When NOT to use

- Data comes from tryptic-digest shotgun proteomics rather than MHC-bound peptide immunopeptidomics — the unspecific-cleavage search space and DeepRescore feature design target HLA peptide data
- You need per-peptide binding-affinity annotation or neoantigen prediction rather than a reproduction of identification counts — that requires additional steps (netMHCpan percentile rank <= 2%) outside this skill
- Input identifications are already rescored feature-level outputs (e.g., existing Percolator q-value tables) rather than raw search-engine PSM results

## Inputs

- Raw MS/MS data for D1 (Mel15 HLA class I peptides, 938,131 spectra, PRIDE PXD004894) and D2 (A*11:01 mono-allelic cell line, 279,700 spectra, MassIVE MSV000084442)
- MGF-format converted MS/MS files (ProteoWizard, peak picking, m/z 1-2)
- Customized protein databases: RefSeq + variant proteins + 245 contaminants (D1) or UCSC human database + 245 contaminants (D2), with decoys appended
- Somatic mutation VCFs annotated with ANNOVAR (for D1 Customprodbj database)
- Reported reference counts: D1 pre-rescoring 23400/22789/20727/17122 for Comet/MS-GF+/X!Tandem/MaxQuant; D1 post-rescoring 35683/34204/28104/32776; D2 pre-rescoring 3255/4935/4048/2635

## Outputs

- Per-engine, per-dataset unique peptide counts at 1% FDR (pre-rescoring) and at q-value <= 0.01 (post-DeepRescore rescoring)
- Summary table of baseline vs. rescored counts with percent gains (e.g., Comet/D1 +56%, MaxQuant/D1 +109%)
- DeepRescore-rescored PSM files with q-value column for filtering

## How to apply

Convert raw MS/MS files to MGF with ProteoWizard (v3.0.19014) using peak picking, then build customized databases: for D1 (Mel15, 938,131 spectra) combine human RefSeq (56,659 sequences), ANNOVAR-annotated variant proteins, and 245 contaminants via Customprodbj/NeoFlow; for D2 (A*11:01, 279,700 spectra) use the UCSC human protein database (63,691 sequences) plus 245 contaminants; append decoys to both. Search with MS-GF+, Comet, X!Tandem, and MaxQuant using unspecific cleavage (immunopeptidomics has no enzymatic digestion), oxidation of methionine as variable and carbamidomethyl of cysteine as fixed modification, 10 ppm precursor and 0.02 Da MS/MS tolerance, peptide length 8-25. Filter identifications with the target-decoy strategy implemented in PGA at 1% FDR at both PSM and peptide levels using each engine's Evalue (or PEP for MaxQuant), then rescore with DeepRescore (NextFlow/Docker), which integrates search-engine-specific, engine-independent, and deep learning features (RTdelta from AutoRT transfer-learning models, spectral angle SA from pDeep2) via Percolator v3.4. Filter rescored results at q-value <= 0.01 and tabulate unique peptide counts per engine per dataset for direct comparison with reported values (e.g., D1 pre-rescoring: Comet 23400, MS-GF+ 22789, X!Tandem 20727, MaxQuant 17122).

## Related tools

- **Comet** (Search engine #1; produces PSMs rescored by DeepRescore; baseline D1 count 23400 unique peptides)
- **MS-GF+** (Search engine #2; Evalue-based PSM output filtered via PGA target-decoy at 1% FDR) — https://github.com/MSGFPlus/msgfplus
- **X!Tandem** (Search engine #3 producing one of four parallel PSM sets)
- **MaxQuant** (Search engine #4; PEP-scored PSMs used for FDR filtering and rescoring)
- **ProteoWizard** (Converts raw MS/MS files to MGF with peak picking before searching)
- **Customprodbj** (Builds the D1 customized database (RefSeq + variant proteins + contaminants), automated via NeoFlow) — https://github.com/bzhanglab/customprodbj
- **NeoFlow** (Automates variant annotation and customized database construction for D1) — https://github.com/bzhanglab/neoflow
- **PGA** (Implements target-decoy FDR filtering at 1% at PSM and peptide levels)
- **DeepRescore** (Rescores PSMs from all four engines, adding RTdelta (AutoRT) and spectral angle (pDeep2) features via Percolator) — https://github.com/bzhanglab/DeepRescore
- **AutoRT** (Transfer-learning RT prediction per LC-MS/MS run yielding the RTdelta feature) — https://github.com/bzhanglab/AutoRT

## Examples

```
git clone https://github.com/bzhanglab/DeepRescore && nextflow run DeepRescore/main.nf --input D1_comet.mgf --engine comet --db D1_custom_decoy.fasta
```

## Evaluation signals

- Summary table contains all four engines for both D1 and D2, with pre-rescoring counts near reported values (D1: 23400/22789/20727/17122; D2: 3255/4935/4048/2635)
- Post-rescoring counts match reported DeepRescore results within expected range (D1: Comet 35683, MS-GF+ 34204, X!Tandem 28104, MaxQuant 32776, i.e., +56%/+109%/+27%/+61%)
- All pre-rescoring counts are filtered at 1% FDR at both PSM and peptide levels via PGA target-decoy; all rescored counts use q-value <= 0.01
- Peptide length distribution restricted to 8-25 residues in the search and counting step
- Four-engine overlap on D1 reproduces the reported union of 31447 unique peptides with ~33% common to all four and ~24% unique to one engine

## Limitations

- DeepRescore currently supports only four search engines (MS-GF+, Comet, X!Tandem, MaxQuant) and takes MGF input plus engine identification results
- Rescoring performance depends on deep learning features (RTdelta from AutoRT transfer learning, spectral angle from pDeep2), which require per-run fine-tuning and are training-data dependent
- Counts are sensitive to the exact database composition (RefSeq vs. UCSC, 245 appended contaminants, decoy handling) and engine versions (e.g., Comet v2018.01 rev. 4, MS-GF+ v2018.10.15, MaxQuant v1.6.5.0)

## Evidence

- [results] D1 post-rescoring counts: [evidence span withheld — anchor status: not_found]
