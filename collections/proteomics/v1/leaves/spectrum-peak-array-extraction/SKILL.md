---
name: spectrum-peak-array-extraction
description: Use when an mzML (or another jmzReader-supported peak list/XML format
  such as mzXML, MGF, PRIDE XML) file must be read in a Java project to obtain per-spectrum
  MS level, precursor ion information, and peak m/z/intensity lists.
when_to_use_negative:
- When the input is an mzIdentML file — mzIdentML files do not contain spectra data
  but refer to external peak list files, so this extraction would find no peaks.
- When spectra are already parsed into in-memory Spectrum/peak-list objects and only
  downstream analysis is needed.
- When you need to parse vendor raw formats (e.g. Thermo RAW) that are outside the
  jmzReader peak list/XML format set.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: jmzReader
  role: Library providing the mzML parser and the common parser interface used to
    iterate spectra and extract peak m/z/intensity arrays
  repo: http://code.google.com/p/jmzreader/
- name: jmzML
  role: Already existing Java API on which the jmzReader mzML parser is implemented
- name: Maven
  role: Build/dependency mechanism used to add the uk.ac.ebi.pride.tools:jmzreader
    artifact and EBI Nexus repositories to the project
- name: Java
  role: Programming language in which the parsers and extraction code are written
    and run
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:c9bdd20228eb97ec0fb1fbc6cc0885e3485917c17aa1a82c2e65b543dc627cb2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - Maven
  - jmzML
  - jmzReader
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# spectrum-peak-array-extraction

## Summary

Parses mzML mass spectrometry files in Java using the jmzReader mzML parser (built on the jmzML API) to extract per-spectrum metadata (MS level, precursor m/z and charge) and peak m/z/intensity arrays via a common parser interface.

## When to use

Use when an mzML (or another jmzReader-supported peak list/XML format such as mzXML, MGF, PRIDE XML) file must be read in a Java project to obtain per-spectrum MS level, precursor ion information, and peak m/z/intensity lists.

## When NOT to use

- When the input is an mzIdentML file — mzIdentML files do not contain spectra data but refer to external peak list files, so this extraction would find no peaks.
- When spectra are already parsed into in-memory Spectrum/peak-list objects and only downstream analysis is needed.
- When you need to parse vendor raw formats (e.g. Thermo RAW) that are outside the jmzReader peak list/XML format set.

## Inputs

- mzML file (XML-based MS data format with spectra and peak lists)
- Java/Maven project pom file
- other jmzReader-supported peak list formats (DTA, MS2, MGF, PKL, mzXML, mzData, PRIDE XML)

## Outputs

- mzml_spectra.tsv with one row per spectrum and columns: index, MS level, precursor m/z, precursor charge, peak m/z array, peak intensity array
- In-memory Spectrum objects exposing MS level, precursor ion metadata, and peak lists

## How to apply

Add the jmzReader parsers to a Maven project by declaring the uk.ac.ebi.pride.tools:jmzreader dependency in the pom file, pointing at the EBI Nexus pst-release and pst-snapshots repositories. Load the mzML file through the jmzReader mzML parser, which reuses the already existing jmzML API rather than a new parser implementation. Iterate over the spectra via the common jmzReader parser interface, recording each spectrum's index and MS level, then extract precursor m/z and charge where present and pull the peak m/z and intensity arrays for every spectrum. Write the records to a TSV (e.g. mzml_spectra.tsv) with one row per spectrum and columns for index, MS level, precursor fields, and peak arrays; validate that the row count equals the number of spectra in the input file. Because all jmzReader parsers implement one common interface optimized for mzIdentML cross-referencing, the same extraction code works unchanged for other supported formats.

## Related tools

- **jmzReader** (Library providing the mzML parser and the common parser interface used to iterate spectra and extract peak m/z/intensity arrays) — http://code.google.com/p/jmzreader/
- **jmzML** (Already existing Java API on which the jmzReader mzML parser is implemented)
- **Maven** (Build/dependency mechanism used to add the uk.ac.ebi.pride.tools:jmzreader artifact and EBI Nexus repositories to the project)
- **Java** (Programming language in which the parsers and extraction code are written and run)

## Examples

```
MzMLFile file = new MzMLFile(new File("sample.mzML")); for (Spectrum s : file.getSpectra()) { int msLevel = s.getMsLevel(); PeakList peaks = s.getPeakList(); }
```

## Evaluation signals

- mzml_spectra.tsv exists and is well-formed TSV containing the required fields: index, MS level, precursor m/z, precursor charge, and peak arrays.
- Row count of the output TSV equals the number of spectra in the input mzML file.
- Each spectrum's recorded MS level and precursor ion information (m/z and charge, where present) match the mzML metadata.
- Extracted peak m/z and intensity arrays match the peak lists stored in the input file for each spectrum.
- The extraction code compiles against the common jmzReader interface without format-specific branches, confirming it reuses the shared parser API.

## Limitations

- The README's Maven snippet uses a placeholder version ('version'); you need to change the version number to the latest version of jmzReader.
- The README's EBI release repository URL contains a typo ('hhttps://ww.ebi.ac.uk/...'), so the repository entry must be corrected to resolve the dependency.
- The library is still evolving and the authors commit to expanding it and adding more classes, so interfaces may change between versions.

## Evidence
