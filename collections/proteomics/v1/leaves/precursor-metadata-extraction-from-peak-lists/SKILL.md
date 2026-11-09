---
name: precursor-metadata-extraction-from-peak-lists
description: Use when a peak list file in PKL (or a related text format like DTA,
  MS2, MGF) needs header parsing to recover precursor m/z, charge, and intensity fields,
  and per-spectrum records with peak lists must be assembled.
when_to_use_negative:
- Input is already a structured mzML or mzData XML file where precursor metadata is
  directly queryable via the standard API rather than text header parsing.
- The task is visualizing or converting peak list files to mgf format rather than
  extracting precursor metadata — jmzReader-gui handles that directly.
- The peak list format is not among the supported ones (dta, mgf, ms2, mzData, mzXML,
  pkl, mzML, PRIDE XML) and no custom parser is available.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: jmzReader
  role: Provides the PKL parser among its collection of Java APIs for peak list formats;
    its line-by-line parsing class and common parser interface serve as the reference
    implementation for the extraction workflow.
  repo: https://github.com/PRIDE-Utilities/jmzReader
- name: jmzReader-gui
  role: Viewer/export tool that supports all jmzReader formats and can export them
    to mgf; useful for visual verification of parsed PKL records.
- name: Maven
  role: Used to add jmzReader as a dependency to the project via the EBI Nexus repositories
    (pst-release, pst-snapshots).
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:3def7340561357f8b885b94419f63143296243ed5e06da568bf717111c886990
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PRIDE-Utilities/jmzReader
  tools:
  - Maven
  - jmzReader
  - jmzReader-gui
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# precursor-metadata-extraction-from-peak-lists

## Summary

Extracts precursor metadata (precursor m/z, charge state, precursor intensity) from peak list files — specifically the PKL format — and assembles one structured record per spectrum combining precursor metadata with the parsed m/z–intensity peak list, mirroring jmzReader's memory-efficient parsing approach.

## When to use

Use when a peak list file in PKL (or a related text format like DTA, MS2, MGF) needs header parsing to recover precursor m/z, charge, and intensity fields, and per-spectrum records with peak lists must be assembled.

## When NOT to use

- Input is already a structured mzML or mzData XML file where precursor metadata is directly queryable via the standard API rather than text header parsing.
- The task is visualizing or converting peak list files to mgf format rather than extracting precursor metadata — jmzReader-gui handles that directly.
- The peak list format is not among the supported ones (dta, mgf, ms2, mzData, mzXML, pkl, mzML, PRIDE XML) and no custom parser is available.

## Inputs

- PKL peak list file (text format with precursor m/z, charge, and intensity header per spectrum)
- Java/Maven environment with jmzReader dependency (uk.ac.ebi.pride.tools:jmzreader)

## Outputs

- Structured per-spectrum records each containing precursor m/z, charge state, precursor intensity, and a non-empty m/z–intensity peak list (e.g. pkl_spectra table)

## How to apply

Parse the PKL peak list file line by line: each spectrum header line carries the precursor m/z, charge state, and precursor intensity fields, which are extracted into a precursor metadata record. The subsequent peak lines are read into m/z–intensity peak pairs using a custom line-by-line text file parser, mirroring jmzReader's approach that lets all parsers handle arbitrarily large files in minimal memory. Assemble one structured record per spectrum combining the precursor metadata and the parsed peak list, using a uniform record schema analogous to jmzReader's common parser interface. Write all records to an output table (e.g. pkl_spectra). The rationale is that mzIdentML files do not contain spectra data but refer to external peak list files, so a uniform per-spectrum schema lets downstream software reference spectra through one interface instead of supporting multiple formats.

## Related tools

- **jmzReader** (Provides the PKL parser among its collection of Java APIs for peak list formats; its line-by-line parsing class and common parser interface serve as the reference implementation for the extraction workflow.) — https://github.com/PRIDE-Utilities/jmzReader
- **jmzReader-gui** (Viewer/export tool that supports all jmzReader formats and can export them to mgf; useful for visual verification of parsed PKL records.)
- **Maven** (Used to add jmzReader as a dependency to the project via the EBI Nexus repositories (pst-release, pst-snapshots).)

## Examples

```
import uk.ac.ebi.pride.tools.jmzreader.JmzReader; import uk.ac.ebi.pride.tools.pkl_reader.PklReader; JmzReader pkl = new PklReader(new File("sample.pkl")); for (Spectrum s : pkl.getSpectra()) { Precursor p = s.getPrecursor(); }
```

## Evaluation signals

- The parser runs on the sample PKL file from the jmzReader repository test resources without errors.
- Every emitted spectrum record contains a precursor m/z value, a charge state, and a non-empty peak list.
- Peak list m/z–intensity pairs in each record match the corresponding peak lines in the source PKL file.
- Each PKL spectrum header line contributes exactly one structured record; record count equals spectrum count in the file.
- All records follow a uniform schema (precursor m/z, charge, intensity, peak list), consistent with jmzReader's common parser interface.

## Limitations

- The README notes the library is still evolving: 'the library is still evolving, we are committed to expand this library and add more useful classes.'
- The Maven dependency snippet requires manually updating the version number to the latest version — the README says 'you need to change the version number to the latest version.'
- The README's repository URL snippet contains a malformed URL ('hhttps://ww.ebi.ac.uk/...'), which may need correction when configuring Maven repositories.
- jmzReader handles arbitrarily large files in minimal memory but relies on a custom line-by-line text parsing class; memory behavior depends on that implementation.

## Evidence
