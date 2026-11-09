---
name: cv-parameter-extraction
description: Use when an mzIdentML file or downstream tool references an external
  mzData peak list file and you must extract spectrum IDs, cvParam metadata, and peak
  arrays into a structured record per spectrum.
when_to_use_negative:
- The input spectra are already in a parsed, structured form (e.g. an existing MGF
  export or feature table) and no cvParam/mzData re-parsing is needed
- You need to modify or convert peak list files to another format rather than extract
  per-spectrum records — jmzReader-gui's mgf export covers format conversion instead
- The spectra are embedded inside the mzIdentML file itself; mzIdentML files do not
  contain spectra data but refer to external peak list files
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: jmzReader
  role: Java parser library whose mzData reader exposes spectra (identifiers, cvParams,
    peak lists) through the common interface used by mzIdentML-referencing software
  repo: https://github.com/PRIDE-Utilities/jmzReader
- name: jmzML
  role: Underlying mzML API on which the jmzReader collection of parsers builds
- name: Maven
  role: Build/dependency manager used to pull the jmzReader parser dependencies from
    the EBI Nexus repository
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:0743238290872d66d5f179a05f8ffa381497adf8803164ebb0355ccd97c02b48
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PRIDE-Utilities/jmzReader
  tools:
  - Maven
  - jmzML
  - jmzReader
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cv-parameter-extraction

## Summary

A method for parsing mzData XML peak list files with the jmzReader library to extract per-spectrum identifiers, controlled-vocabulary (cvParam) parameters, and decoded m/z-intensity peak lists. It supports mzIdentML-centric proteomics software that needs external spectra referenced by a common parser interface.

## When to use

Use when an mzIdentML file or downstream tool references an external mzData peak list file and you must extract spectrum IDs, cvParam metadata, and peak arrays into a structured record per spectrum.

## When NOT to use

- The input spectra are already in a parsed, structured form (e.g. an existing MGF export or feature table) and no cvParam/mzData re-parsing is needed
- You need to modify or convert peak list files to another format rather than extract per-spectrum records — jmzReader-gui's mgf export covers format conversion instead
- The spectra are embedded inside the mzIdentML file itself; mzIdentML files do not contain spectra data but refer to external peak list files

## Inputs

- mzData XML peak list file (e.g. sample mzData test resources from the jmzReader repository)
- Maven pom.xml for the Java project consuming uk.ac.ebi.pride.tools:jmzreader
- mzIdentML file that references the external mzData peak list file

## Outputs

- One structured JSON record per spectrum containing: spectrum identifier (spectrum id/index), CV parameter (cvParam) fields, and non-empty m/z-intensity peak arrays

## How to apply

Add the jmzReader parsers as a Maven dependency (uk.ac.ebi.pride.tools:jmzreader) against the EBI Nexus repositories. Instantiate the mzData reader on the target mzData XML file (e.g. the sample mzData file from the jmzReader test resources) and iterate over all spectra, capturing each spectrum's identifier (spectrum id/index) and its selected cvParam parameters. Decode the base64-encoded m/z and intensity peak arrays stored in the mzData binary data blocks and attach them to each spectrum record. Serialize one structured JSON record per spectrum (identifier, CV parameters, peak list). Rationale: all jmzReader parsers implement a common interface supporting mzIdentML's external spectra referencing methods, so only one interface must be supported instead of multiple peak list formats.

## Related tools

- **jmzReader** (Java parser library whose mzData reader exposes spectra (identifiers, cvParams, peak lists) through the common interface used by mzIdentML-referencing software) — https://github.com/PRIDE-Utilities/jmzReader
- **jmzML** (Underlying mzML API on which the jmzReader collection of parsers builds)
- **Maven** (Build/dependency manager used to pull the jmzReader parser dependencies from the EBI Nexus repository)

## Examples

```
MzDataFile mzDataFile = new MzDataFile(new File("sample.mzData")); for (Spectrum s : mzDataFile.getSpectra()) { extract s.getId(), s.getCVParams(), decode peaks; }
```

## Evaluation signals

- The output JSON file exists and is valid JSON
- The output contains exactly one record per spectrum in the source mzData file
- Every record includes a spectrum identifier (spectrum id/index)
- Every record includes CV parameter (cvParam) fields
- Every record includes non-empty m/z-intensity peak arrays decoded from the mzData binary data blocks

## Limitations

- jmzReader's version number must be set to the latest version in the Maven dependency; the README only supplies a placeholder
- mzIdentML files do not contain spectra data themselves, so this skill only works when the referenced external peak list (mzData) file is available
- The library is described as still evolving; the parsers are a moving API surface

## Evidence

- [abstract] all parsers implement a common interface that supports the various methods used by mzIdentML to reference external spectra: "all parsers implement a common interface that supports the various methods used by mzIdentML to reference external spectra"
- [abstract] parsers support DTA, MS2, MGF, PKL, mzXML, mzData, mzML: "parse the most commonly used peak list and XML-based mass spectrometry (MS) data formats: DTA, MS2, MGF, PKL, mzXML, mzData, and mzML"
