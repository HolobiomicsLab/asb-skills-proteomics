---
name: proteomics-database-search-and-psm-validation
description: Use when raw MS/MS runs (e.g., mzML converted from instrument raw files)
  need peptide identification and FDR-validated PSMs in a search-engine hits CSV to
  feed downstream isobaric reporter ion quantification such as IsobaricQuant.
when_to_use_negative:
- The input is already a validated PSM hits file or a quantified reporter-ion table
  — searching would duplicate identification work and not alter the IsobaricQuant
  inputs
- The data are label-free or SILAC rather than iTRAQ/TMT/TMTpro isobarically labeled
  peptides, so scan-group-level reporter quantification via IsobaricQuant is not applicable
- Only a small number of scans need manual inspection, where the IsobaricQuant integrated
  viewer (visual assessment of SPS ion selection and isolation window purity) is more
  appropriate than a full re-search
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Comet
  role: Search engine used to identify peptides from the mzML spectra before conversion
    into the IsobaricQuant hits format
  repo: https://uwpr.github.io/Comet/
- name: Mokapot
  role: PSM validation tool applied to Comet results to produce FDR-controlled peptide-spectrum
    matches
- name: msconvert
  role: ProteoWizard tool used to convert instrument raw files to mzML before searching
    and quantification
  repo: http://proteowizard.sourceforge.net/tools/filters.html
- name: IsobaricQuant
  role: Downstream consumer of the validated hits file; defines the required hits
    CSV schema and performs reporter ion quantification
  repo: https://github.com/Villen-Lab/isobaricquant
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:7ff9be0f95cd68d3a927cff5f9c4c1e2205658a4af7c997834441a635f4ba0db
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Comet
  - IsobaricQuant
  - Java
  - Mokapot
  - msconvert
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# proteomics-database-search-and-psm-validation

## Summary

Search tandem mass spectra against a protein database with Comet, validate peptide-spectrum matches (PSMs) with Mokapot, and export the validated PSMs as the comma-separated hits file (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z) required by IsobaricQuant for reporter ion quantification of isobarically labeled (iTRAQ/TMT/TMTpro) data.

## When to use

Use when raw MS/MS runs (e.g., mzML converted from instrument raw files) need peptide identification and FDR-validated PSMs in a search-engine hits CSV to feed downstream isobaric reporter ion quantification such as IsobaricQuant.

## When NOT to use

- The input is already a validated PSM hits file or a quantified reporter-ion table — searching would duplicate identification work and not alter the IsobaricQuant inputs
- The data are label-free or SILAC rather than iTRAQ/TMT/TMTpro isobarically labeled peptides, so scan-group-level reporter quantification via IsobaricQuant is not applicable
- Only a small number of scans need manual inspection, where the IsobaricQuant integrated viewer (visual assessment of SPS ion selection and isolation window purity) is more appropriate than a full re-search

## Inputs

- mzML files converted from instrument raw data (e.g., via msconvert from PRIDE raw files such as PXD020815)
- Protein sequence database (FASTA) for Comet searching
- Comet search output files (per-spectrum peptide matches)
- Mokapot configuration/parameters for PSM-level FDR validation

## Outputs

- Comma-separated hits CSV with columns: Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z
- FDR-validated PSM list from Mokapot
- PSM identifications per scan suitable for downstream reporter ion quantification (isopep.csv/isopep_extra.csv after IsobaricQuant)

## How to apply

Convert instrument raw files to mzML first (e.g., with ProteoWizard msconvert, including peakPicking and zeroSamples filters as recommended for IsobaricQuant), then search the spectra with Comet against the appropriate protein database. Validate the resulting PSMs with Mokapot to control false discovery at the peptide-spectrum level, then convert the validated results into the specific hits file format required by IsobaricQuant: a CSV with columns Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, and M/Z. The Start Scan and M/Z fields are critical because IsobaricQuant uses them to locate the correct scans in the mzML for reporter ion extraction (e.g., 20 ppm reporter-ion tolerance, TMTpro 16-plex isoMethod configuration on Java 17). Verify correctness by checking that validated PSM scan numbers and m/z values align with scans present in the mzML files and that hits coverage matches the acquisition method (MS2, SPS-MS3, or RTS-MS3).

## Related tools

- **Comet** (Search engine used to identify peptides from the mzML spectra before conversion into the IsobaricQuant hits format) — https://uwpr.github.io/Comet/
- **Mokapot** (PSM validation tool applied to Comet results to produce FDR-controlled peptide-spectrum matches)
- **msconvert** (ProteoWizard tool used to convert instrument raw files to mzML before searching and quantification) — http://proteowizard.sourceforge.net/tools/filters.html
- **IsobaricQuant** (Downstream consumer of the validated hits file; defines the required hits CSV schema and performs reporter ion quantification) — https://github.com/Villen-Lab/isobaricquant

## Examples

```
java -jar IsobaricQuant.jar -c config -mzf mzML -h hits -o output
```

## Evaluation signals

- Hits CSV contains exactly the required columns: Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z
- Every Start Scan value in the hits file corresponds to a scan present in the paired mzML file, so IsobaricQuant can extract reporter ions (e.g., with 20 ppm reporter-ion tolerance)
- PSM-level FDR is controlled after Mokapot validation (validated PSMs, not raw Comet output, feed the hits file)
- In the HYpro16 TMTpro 16-plex benchmark, downstream IsobaricQuant quantification reached 100% of shared yeast scan groups fully quantified across all 16 reporter channels at the MS2 level
- log10 reporter intensities from tool-specific search pipelines (Comet/Mokapot vs FragPipe vs MaxQuant) show high pairwise Pearson correlation, confirming quantification is on par with state-of-the-art tools

## Limitations

- IsobaricQuant's hits format requires a specific conversion of the search engine output, so search results from engines other than Comet need an equivalent conversion step
- Cutoffs and downstream filtering performance are dataset-dependent; the optimal QC score and cutoff must be adjusted per dataset rather than reused blindly
- Hits files tie identifications to scan numbers and precursor m/z; mismatched mzML conversion settings (e.g., missing peakPicking or zeroSamples filters) can break scan-level alignment

## Evidence
