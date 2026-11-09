---
name: common-spectra-interface-reconstruction
description: Use when the input is a text-based peak list file (DTA, MGF, MS2, PKL,
  mzXML, mzData, mzML) that must be converted into structured per-spectrum records,
  or when mzIdentML spectrum references must be resolved against external peak list
  files.
when_to_use_negative:
- The input is raw vendor instrument output (not one of the DTA, MS2, MGF, PKL, mzXML,
  mzData, mzML, PRIDE XML peak list formats) — jmzReader only parses the listed text
  and XML-based formats.
- The spectra already exist as structured records (e.g., converted MGF or mzML handled
  through jmzML) and no line-by-line peak list parsing is required.
- The task is exporting peak lists to a format other than mgf via jmzReader-gui, which
  only exports the supported file formats into the commonly used mgf format.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: jmzReader
  role: Java API library whose common parser interface and line-by-line text parsing
    underpin the DTA peak list parsing and structured spectrum record emission
  repo: http://code.google.com/p/jmzreader/
- name: Java
  role: Implementation language; the skill's parser is built as Java code against
    the jmzReader API semantics
- name: jmzReader-gui
  role: Optional visualization tool demonstrating the jmzReader interface and exporting
    supported formats (including DTA-parsed data) to mgf
  repo: https://github.com/PRIDE-Utilities/jmzReader/actions/workflows/maven.yml
- name: Maven
  role: Build/dependency mechanism for pulling the jmzReader parsers into the project
    via the EBI Nexus repositories
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:0868331acc02dfdc1353e26b3759bd96be2c9966fab3212df3ce9f8e489408f7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - Maven
  - jmzReader
  - jmzReader-gui
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# common-spectra-interface-reconstruction

## Summary

Reconstruct a parser for DTA-style peak list files (and sibling formats MS2, MGF, PKL, mzXML, mzData, mzML) behind the single common jmzReader interface, emitting structured spectrum records with charge state, precursor mass, and m/z–intensity peak pairs.

## When to use

Use when the input is a text-based peak list file (DTA, MGF, MS2, PKL, mzXML, mzData, mzML) that must be converted into structured per-spectrum records, or when mzIdentML spectrum references must be resolved against external peak list files.

## When NOT to use

- The input is raw vendor instrument output (not one of the DTA, MS2, MGF, PKL, mzXML, mzData, mzML, PRIDE XML peak list formats) — jmzReader only parses the listed text and XML-based formats.
- The spectra already exist as structured records (e.g., converted MGF or mzML handled through jmzML) and no line-by-line peak list parsing is required.
- The task is exporting peak lists to a format other than mgf via jmzReader-gui, which only exports the supported file formats into the commonly used mgf format.

## Inputs

- DTA-format peak list text file (line-based spectra with precursor mass, charge state, and whitespace-separated m/z intensity pairs)
- Other jmzReader-supported peak list formats: MS2, MGF, PKL, mzXML, mzData, mzML, PRIDE XML
- mzIdentML file referencing external peak list files (parser must support its spectrum-reference methods)

## Outputs

- One structured record (JSON) per input spectrum containing charge state, precursor mass, and a list of peak m/z / intensity pairs
- Spectrum objects exposed through the common jmzReader parser interface, consumable uniformly across DTA, MS2, MGF, PKL, mzXML, mzData, and mzML inputs

## How to apply

Load the peak list file and parse it line by line with a custom text-file reader, mirroring jmzReader's approach so arbitrarily large files are handled in minimal memory. For DTA specifically, read the first whitespace-separated line of each spectrum as the precursor mass and charge state, then treat each subsequent line as a whitespace-separated m/z/intensity peak pair. Emit one structured record per spectrum (e.g., JSON) carrying the charge state, precursor mass, and the peak m/z and intensity lists, exposing the record through the common jmzReader parser interface so downstream code (e.g., mzIdentML consumers) needs to support only one interface instead of multiple peak list formats. When wiring the library directly, add jmzReader as a Maven dependency (groupId uk.ac.ebi.pride.tools, artifactId jmzreader, jar) and replace the version placeholder with the latest version. Validate by running the parser on the sample DTA file from the repository test resources and checking record count, charge-state fields, and non-empty peak pairs.

## Related tools

- **jmzReader** (Java API library whose common parser interface and line-by-line text parsing underpin the DTA peak list parsing and structured spectrum record emission) — http://code.google.com/p/jmzreader/
- **Java** (Implementation language; the skill's parser is built as Java code against the jmzReader API semantics)
- **jmzReader-gui** (Optional visualization tool demonstrating the jmzReader interface and exporting supported formats (including DTA-parsed data) to mgf) — https://github.com/PRIDE-Utilities/jmzReader/actions/workflows/maven.yml
- **Maven** (Build/dependency mechanism for pulling the jmzReader parsers into the project via the EBI Nexus repositories)

## Examples

```
<dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><type>jar</type></dependency>  <!-- then load a sample DTA file and parse it line by line via the jmzReader interface -->
```

## Evaluation signals

- Running the parser on the sample DTA file yields a dta_parsed_records structure that exists and contains exactly one record per spectrum in the input file.
- Every emitted record carries a charge state field parsed from the first whitespace-separated line of each DTA spectrum.
- Every record contains non-empty peak pairs, i.e., each peak line contributes both an m/z and an intensity value.
- The same downstream code path works unchanged for DTA, MS2, MGF, PKL, mzXML, mzData, and mzML inputs because all parsers implement the common interface supporting mzIdentML spectrum-reference methods.
- Memory footprint stays flat when parsing an arbitrarily large peak list file, reflecting the line-by-line custom parsing class.

## Limitations

- The Maven version in the documented dependency snippet is a placeholder — you must change the version number to the latest version, and the EBI Nexus repository URLs must resolve.
- The library is still evolving; the authors note they are committed to expand it and add more classes, so APIs may change.
- jmzReader-gui can only export supported file formats into mgf; no other export target is provided.
- mzIdentML files do not contain spectra data — the parser must be used in conjunction with the external peak list files that mzIdentML references.

## Evidence

- [abstract] We here present the jmzReader library: a collection of Java application programming interfaces (APIs) to parse the most commonly used peak list and XML-based mass spectrometry (MS) data formats: "We here present the jmzReader library: a collection of Java application programming interfaces (APIs) to parse the most commonly used peak list and XML-based mass spectrometry (MS) data formats"
- [intro] Based on a custom build class to efficiently parse text files line by line all parsers can handle arbitrary large files in minimal memory: "Based on a custom build class to efficiently parse text files line by line all parsers can handle arbitrary large files in minimal memory"
- [intro] All peak list parsers support the methods used by mzIdentML to reference external spectra and implement a common interface: "All peak list parsers support the methods used by mzIdentML to reference external spectra and implement a common interface"
- [readme] The jmzReader library is a collection of Java APIs to parse the most commonly used MS peak list formats.: "The jmzReader library is a collection of Java APIs to parse the most commonly used MS peak list formats."
