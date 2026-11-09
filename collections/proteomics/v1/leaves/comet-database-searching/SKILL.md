---
name: comet-database-searching
description: Use when raw immunopeptidomics (MHC-bound peptide) MS/MS data has been
  converted to MGF and you need PSM-level peptide-spectrum matches against a target-decoy
  protein database for downstream FDR filtering, rescoring, or validation.
when_to_use_negative:
- You already have PSMs from Comet or another engine and only want to improve confidence
  — use DeepRescore post-processing rescoring instead of re-searching
- The raw data has not yet been converted from vendor format to MGF — convert with
  ProteoWizard first
- The goal is de novo sequencing or open/modifiable search-space analysis rather than
  fixed-parameter database searching against a defined FASTA
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Comet
  role: Database search engine used to identify peptides from MGF spectra against
    a target-decoy database (v2018.01 rev. 4 in the synthetic peptide validation workflow)
  repo: http://comet-ms.sourceforge.net/
- name: ProteoWizard
  role: Converts raw Orbitrap Fusion HCD MS/MS files to MGF format required as Comet
    input
- name: DeepRescore
  role: Consumes Comet identification results plus MGF data to rescore PSMs with deep
    learning-derived RT and MS/MS prediction features
  repo: https://github.com/bzhanglab/DeepRescore
- name: PDV
  role: Visualization of Comet identification results (e.g., spectral comparison of
    detected synthetic peptides)
  repo: https://github.com/wenbostar/PDV
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:d0b87182bad830b31846b2b601e71a2a7a09427bbc9d0bde99205531ed1c5296
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Comet
  - DeepRescore
  - PDV
  - ProteoWizard
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# comet-database-searching

## Summary

Database searching of immunopeptidomics MS/MS spectra with Comet, one of the four search engines benchmarked in the DeepRescore study, producing PSM/peptide identifications that are filtered at 1% FDR and can feed DeepRescore rescoring or synthetic peptide validation.

## When to use

Use when raw immunopeptidomics (MHC-bound peptide) MS/MS data has been converted to MGF and you need PSM-level peptide-spectrum matches against a target-decoy protein database for downstream FDR filtering, rescoring, or validation.

## When NOT to use

- You already have PSMs from Comet or another engine and only want to improve confidence — use DeepRescore post-processing rescoring instead of re-searching
- The raw data has not yet been converted from vendor format to MGF — convert with ProteoWizard first
- The goal is de novo sequencing or open/modifiable search-space analysis rather than fixed-parameter database searching against a defined FASTA

## Inputs

- MGF-format MS/MS peak lists converted from raw vendor files (ProteoWizard v3.0.19014)
- Protein sequence FASTA database (e.g., human RefSeq with 56,659 sequences plus 245 contaminants, or customized variant database from Customprodbj/NeoFlow)
- Comet parameter file (enzyme, 10 ppm precursor tolerance, 0.02 Da MS/MS tolerance, fixed/variable modifications, peptide length 8-25)

## Outputs

- Comet PSM/peptide identification results (TSV) with q-value column
- Peptide identifications filtered at 1% FDR (PSM and peptide levels) for downstream DeepRescore rescoring
- Synthetic peptide detection table (peptide sequence, detected/not-detected status) for validation searches

## How to apply

Convert raw vendor MS/MS files (e.g., Orbitrap Fusion HCD data) to MGF format with ProteoWizard (v3.0.19014) first, since Comet consumes MGF in the DeepRescore workflow. Search the MGF against a protein database (e.g., human RefSeq 56,659 sequences, or a customized target-decoy database built by Customprodbj for variant peptides) using parameters suited to unenzymatic immunopeptidomics: unspecific (no-enzyme) cleavage, oxidation of methionine as a variable modification, carbamidomethyl of cysteine as a fixed modification, 10 ppm precursor tolerance, 0.02 Da MS/MS fragment tolerance, and peptide length 8-25. Filter Comet outputs with the target-decoy strategy implemented in PGA, controlling FDR at 1% at both PSM and peptide levels, or by filtering the DeepRescore output column `q-value` with q-value <= 0.01. For synthetic peptide validation searches, restrict the database to only the selected synthetic peptides (e.g., the 18 peptides chosen from dataset D1) so detection status can be directly tabulated. Comet was one of the strongest engines in the benchmark, identifying 23,400 unique peptides on dataset D1 and 3,255 on D2 at 1% FDR.

## Related tools

- **Comet** (Database search engine used to identify peptides from MGF spectra against a target-decoy database (v2018.01 rev. 4 in the synthetic peptide validation workflow)) — http://comet-ms.sourceforge.net/
- **ProteoWizard** (Converts raw Orbitrap Fusion HCD MS/MS files to MGF format required as Comet input)
- **DeepRescore** (Consumes Comet identification results plus MGF data to rescore PSMs with deep learning-derived RT and MS/MS prediction features) — https://github.com/bzhanglab/DeepRescore
- **PDV** (Visualization of Comet identification results (e.g., spectral comparison of detected synthetic peptides)) — https://github.com/wenbostar/PDV

## Examples

```
comet -P comet.params -D target_decoy.fasta sample.mgf
```

## Evaluation signals

- On dataset D1 (Mel15 HLA class I, 938,131 MS/MS spectra), Comet should identify ~23,400 unique peptide sequences at 1% FDR (23,2789/20,727/17,122 for MS-GF+/X!Tandem/MaxQuant for comparison)
- All reported identifications pass 1% FDR at both PSM and peptide levels via the target-decoy strategy implemented in PGA
- Only peptides of length 8-25 are retained in the result (8-11 for MHC binding affinity evaluation)
- Filtering the DeepRescore output by `q-value <= 0.01` yields the final peptide set
- For synthetic peptide validation searches restricted to the 18 D1-derived peptides, detection status at 1% FDR must match the paper's reported validation outcome

## Limitations

- Comet searches depend on prior MGF conversion; the search space for no-enzyme immunopeptidomics is inflated, which can raise false positives and lower sensitivity without rescoring
- Only peptides with length between 8 and 25 were considered in this workflow
- The four search engines identify largely non-overlapping peptide sets on D1 (only 33% common to all four; 24% unique to one), so Comet alone under-represents the full identifiability

## Evidence

- [methods] Raw MS/MS data files from both datasets were converted to MGF files using ProteoWizard (v3.0.19014): "Raw MS/MS data files from both datasets were converted to MGF files using ProteoWizard (v3.0.19014)"
- [methods] FDR control at 1% at PSM and peptide levels: "FDR was controlled at 1% at both PSM and peptide levels"
- [methods] Peptide length constraint: "Only peptides with length between 8 and 25 were considered"
- [results] Comet benchmark performance on D1: "For D1, Comet, MS-GF+, X!Tandem, and MaxQuant identified 23400, 22789, 20727, and 17122 unique peptide sequences, respectively"
- [other] Synthetic peptide validation search parameters: [evidence span withheld — anchor status: not_found]
