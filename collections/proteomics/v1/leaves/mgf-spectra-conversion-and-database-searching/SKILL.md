---
name: mgf-spectra-conversion-and-database-searching
description: Use when starting from vendor raw MS/MS files of immunopeptidomics (or
  similar enzymatic-digestion-free) experiments that must be converted to MGF and
  searched with multiple engines before Percolator-based rescoring.
when_to_use_negative:
- Input MS/MS data already exists as MGF peak lists — skip the conversion step and
  go straight to database searching
- The experiment is a standard tryptic digest with in-silico enzymatic specificity
  — unspecific-cleavage searching inflates the search space unnecessarily
- The goal is quantification or variant discovery requiring NeoFlow's full proteogenomic
  pipeline rather than simple identification-level searching
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ProteoWizard
  role: Converts vendor raw MS/MS files to MGF format with peak-picking filter
- name: MS-GF+
  role: One of four search engines used to identify peptide-spectrum matches from
    the MGF files
  repo: https://github.com/MSGFPlus/msgfplus
- name: Comet
  role: Search engine used to identify PSMs from the MGF files
- name: X!Tandem
  role: Search engine used to identify PSMs from the MGF files
- name: MaxQuant
  role: Search engine used to identify PSMs from the MGF files
- name: PGA
  role: Implements the target-decoy strategy for 1% FDR filtering at PSM and peptide
    levels
- name: DeepRescore
  role: Downstream rescoring tool whose input is the MGF files plus per-engine identification
    results
  repo: https://github.com/bzhanglab/DeepRescore
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:a6e9383b67852a23a204ab181fafe7fa22e81e1209ab9b4e0d9a92bfda241b35
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MSGFPlus/msgfplus
  tools:
  - AutoRT
  - Comet
  - DeepRescore
  - Docker
  - GPTime
  - MS-GF+
  - MaxQuant
  - NextFlow
  - PGA
  - ProteoWizard
  - X!Tandem
  - pDeep2
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# MGF spectra conversion and database searching

## Summary

Converts vendor raw MS/MS immunopeptidomics data to MGF with ProteoWizard peak picking, then searches the MGF files against a human protein database with four engines (Comet, MS-GF+, X!Tandem, MaxQuant) using unspecific cleavage settings appropriate for HLA-bound peptides.

## When to use

Use when starting from vendor raw MS/MS files of immunopeptidomics (or similar enzymatic-digestion-free) experiments that must be converted to MGF and searched with multiple engines before Percolator-based rescoring.

## When NOT to use

- Input MS/MS data already exists as MGF peak lists — skip the conversion step and go straight to database searching
- The experiment is a standard tryptic digest with in-silico enzymatic specificity — unspecific-cleavage searching inflates the search space unnecessarily
- The goal is quantification or variant discovery requiring NeoFlow's full proteogenomic pipeline rather than simple identification-level searching

## Inputs

- Vendor raw MS/MS data files (e.g., from PRIDE PXD004894 or MassIVE MSV000084442)
- Human RefSeq protein database (56,659 sequences) or customized target+decoy FASTA database
- Search parameter definitions (unspecific cleavage, Met oxidation variable, Cys carbamidomethyl fixed, 10 ppm precursor tolerance, 0.02 Da MS/MS tolerance)

## Outputs

- MGF-format peak lists for each raw file
- Per-engine PSM identification results (MS-GF+, Comet, X!Tandem, MaxQuant)
- FDR-filtered (1% PSM and peptide level) unique peptide lists per search engine

## How to apply

Convert raw MS/MS files to MGF using ProteoWizard (v3.0.19014 in the paper) applying the peak-picking filter, since downstream rescoring tools such as DeepRescore take MS/MS data in MGF format. Search the MGF files with four engines (MS-GF+, Comet, X!Tandem, MaxQuant) against a human RefSeq protein database (56,659 sequences; a UCSC Genome Browser variant database with 63,691 sequences plus 245 contaminant proteins was used for D2). Because HLA-bound peptides are not enzymatically generated, use unspecific cleavage, oxidation of methionine as a variable modification, and carbamidomethyl of cysteine as a fixed modification, with 10 ppm precursor and 0.02 Da MS/MS tolerances. Restrict results to peptides of length 8-25 and control FDR at 1% at both PSM and peptide levels using the target-decoy strategy implemented in PGA. Running all four engines is deliberate: the union of identifications (e.g., 31,447 unique peptides for D1, only 33% found by all four engines) supplies the high-quality training peptides used for AutoRT transfer learning.

## Related tools

- **ProteoWizard** (Converts vendor raw MS/MS files to MGF format with peak-picking filter)
- **MS-GF+** (One of four search engines used to identify peptide-spectrum matches from the MGF files) — https://github.com/MSGFPlus/msgfplus
- **Comet** (Search engine used to identify PSMs from the MGF files)
- **X!Tandem** (Search engine used to identify PSMs from the MGF files)
- **MaxQuant** (Search engine used to identify PSMs from the MGF files)
- **PGA** (Implements the target-decoy strategy for 1% FDR filtering at PSM and peptide levels)
- **DeepRescore** (Downstream rescoring tool whose input is the MGF files plus per-engine identification results) — https://github.com/bzhanglab/DeepRescore

## Examples

```
msconvert --mgf --filter "peakPicking true" --outdir mgf/ *.raw
```

## Evaluation signals

- Each raw file yields a corresponding MGF file after ProteoWizard conversion; DeepRescore rejects inputs not in MGF format
- Filtration is applied at q-value <= 0.01 (1% FDR) at both PSM and peptide levels via the target-decoy strategy in PGA
- Only peptides with length between 8 and 25 are retained
- Identified peptide counts should be in the same range as reported: D1 gave 23400 (Comet), 22789 (MS-GF+), 20727 (X!Tandem), 17122 (MaxQuant) unique peptides; D2 gave 3255, 4935, 4048, and 2635 respectively
- Low overlap across engines is expected (only 33% of D1 peptides found by all four, 24% unique to one engine) — near-100% overlap signals a configuration error

## Limitations

- Unspecific cleavage in immunopeptidomics enlarges the search space, which can lead to a high false positive rate and low sensitivity in MHC-binding peptide identification
- The four engines disagree substantially; no single engine captures the full peptide set, so single-engine results underestimate identifications
- MaxQuant's non-MGF native workflows differ from the other engines' handling of the same MGF inputs, complicating direct parameter comparison

## Evidence
