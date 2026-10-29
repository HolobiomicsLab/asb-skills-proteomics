---
name: peptide-spectrum-matching-database-search
description: Use when raw proteomics MS/MS spectra (e.g. mzML converted from instrument
  raw files) must be identified at the PSM level before isobaric-label (iTRAQ/TMT/TMTpro)
  reporter ion quantification with IsobaricQuant, or when preparing its hits file.
when_to_use_negative:
- A validated hits file (PSM table with Start Scan and M/Z columns) already exists
  and only quantification/filtering is needed — go straight to IsobaricQuant.
- The data are not isobarically labeled (label-free or SILAC) — reporter ion quantification
  downstream of the PSM search does not apply.
- Peptides must be identified de novo without a reference protein sequence database.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Comet
  role: Search engine used to assign peptide sequences to MS/MS spectra; its output
    must be converted to the IsobaricQuant hits format
  repo: https://uwpr.github.io/Comet/
- name: Percolator
  role: Validates and weights PSMs after the Comet search prior to hits-file conversion
- name: msconvert
  role: ProteoWizard tool that converts instrument raw files to the mzML spectra files
    searched by Comet (runnable via Docker image chambm/pwiz-skyline-i-agree-to-the-vendor-licenses)
  repo: http://proteowizard.sourceforge.net/tools/filters.html
- name: IsobaricQuant
  role: Downstream consumer of the PSM hits file, mzML file and config file for iTRAQ/TMT/TMTpro
    reporter ion quantification
  repo: https://github.com/Villen-Lab/isobaricquant
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:bc03ce8decbd240b10db8bb48af78b0ce3a3696018a0a5fb4525c66f4dd5c761
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Comet
  - IsobaricQuant
  - Java
  - Percolator
  - msconvert
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# peptide-spectrum-matching-database-search

## Summary

Identify peptide spectral matches (PSMs) by searching MS/MS spectra against a protein sequence database with Comet, validate them with Percolator, and convert the search output into the comma-separated hits file format consumed by IsobaricQuant for TMT/iTRAQ reporter ion quantification.

## When to use

Use when raw proteomics MS/MS spectra (e.g. mzML converted from instrument raw files) must be identified at the PSM level before isobaric-label (iTRAQ/TMT/TMTpro) reporter ion quantification with IsobaricQuant, or when preparing its hits file.

## When NOT to use

- A validated hits file (PSM table with Start Scan and M/Z columns) already exists and only quantification/filtering is needed — go straight to IsobaricQuant.
- The data are not isobarically labeled (label-free or SILAC) — reporter ion quantification downstream of the PSM search does not apply.
- Peptides must be identified de novo without a reference protein sequence database.

## Inputs

- Raw instrument MS/MS files converted to mzML (e.g. via msconvert with --mzML, peakPicking and zeroSamples filters)
- Protein sequence database for the target proteome used by Comet
- User-supplied IsobaricQuant text configuration file (e.g. TMTpro isoMethod, PPMTolerance settings, scanLevel)

## Outputs

- Comet search results with Percolator-validated PSMs
- IsobaricQuant-compatible hits CSV with columns: Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z

## How to apply

Convert instrument raw files to mzML (e.g. with ProteoWizard msconvert, applying peak picking and zero-sample removal filters as recommended in the IsobaricQuant README), then search the spectra with Comet against a protein sequence database and validate/weight the PSMs with Percolator to control false discovery. Because Comet's native output is not directly consumable downstream, convert the validated search results into the IsobaricQuant hits CSV, which requires the columns: Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z. The hits file is then paired with the mzML file and a user-supplied text configuration file to run IsobaricQuant reporter ion quantification at the MS2 or MS3 scan level. The rationale is that IsobaricQuant quantification quality depends entirely on correct peptide identification and scan mapping, so PSM validation (Percolator) precedes any interference filtering or protein-level aggregation.

## Related tools

- **Comet** (Search engine used to assign peptide sequences to MS/MS spectra; its output must be converted to the IsobaricQuant hits format) — https://uwpr.github.io/Comet/
- **Percolator** (Validates and weights PSMs after the Comet search prior to hits-file conversion)
- **msconvert** (ProteoWizard tool that converts instrument raw files to the mzML spectra files searched by Comet (runnable via Docker image chambm/pwiz-skyline-i-agree-to-the-vendor-licenses)) — http://proteowizard.sourceforge.net/tools/filters.html
- **IsobaricQuant** (Downstream consumer of the PSM hits file, mzML file and config file for iTRAQ/TMT/TMTpro reporter ion quantification) — https://github.com/Villen-Lab/isobaricquant

## Evaluation signals

- The hits CSV parses correctly and contains the seven required columns: Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z.
- Start Scan values in the hits file correspond to scan numbers present in the source mzML file.
- PSMs are Percolator-validated (FDR-controlled) before conversion to the hits format.
- IsobaricQuant accepts the hits file, mzML file and config file without input-format errors and emits the expected outputs (isofrag.csv, isolab.csv, isopep.csv, isopep_extra.csv).
- Quantified protein counts are consistent with the expected depth for the acquisition strategy (e.g. ~5,380 proteins for MS2-level PISA data, ~4,033 for RTS-MS3).

## Limitations

- Comet output is not directly consumable by IsobaricQuant — a conversion step to the hits CSV format is mandatory.
- The article specifies no Comet search parameters; users must select their own enzyme, precursor/fragment tolerance (IsobaricQuant itself was run with 20 ppm PPMTolerance/MS1PPMTolerance/MS2PPMTolerance in the PISA workflow) and FDR strategy.
- PSM identification quality bounds downstream accuracy: co-isolated, co-eluting precursors of similar m/z still reduce reporter ion quantification accuracy even when the primary PSM is correct.

## Evidence

- [readme] Raw file conversion to mzML: [evidence span withheld — anchor status: external_doc]
