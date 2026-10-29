---
name: mass-spectrometry-record-serialization
description: Use when input is a peak list or XML-based MS data file (DTA, MS2, MGF,
  PKL, mzXML, mzData, mzML, PRIDE XML) referenced by mzIdentML, and the goal is extracting
  per-spectrum records (id, CV parameters, peak lists) into structured JSON.
when_to_use_negative:
- The input is an mzIdentML file itself — mzIdentML contains no spectra data, only
  references to external peak list files, so there is nothing to serialize from it
  directly.
- The goal is visualizing peak list files or converting them to MGF rather than producing
  structured per-spectrum records — use the jmzReader-gui export instead.
- The input is a raw or proprietary instrument format outside the library's supported
  set (DTA, MS2, MGF, PKL, mzXML, mzData, mzML, PRIDE XML).
edam_operation: http://edamontology.org/operation_0253
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: jmzReader
  role: Java API collection providing the format parsers (including the mzData and
    MS2 readers) and the single common interface supporting mzIdentML's external-spectra
    reference methods
  repo: http://code.google.com/p/jmzreader/
- name: jmzReader-gui
  role: Companion viewer for peak list files that demonstrates the jmzReader interface
    usage
  repo: http://code.google.com/p/jmzreader/
- name: Maven
  role: Build/dependency mechanism used to pull the jmzReader parser from the EBI
    pst-release Nexus repository
- name: jmzML
  role: Existing API on which the jmzReader mzML parser is based
provenance:
  source_task_ids:
  - task_006
  - task_002
  - task_008
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:3ad8ae1f22f9261761410fc32d21b6b3bff593937644e8d5b84075208d5b908f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Maven
  - jmzML
  - jmzReader
  - jmzReader-gui
  edam_operation: http://edamontology.org/operation_0253
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mass-spectrometry-record-serialization

## Summary

Serializes spectra from MS peak list and XML-based files (DTA, MS2, MGF, PKL, mzXML, mzData, mzML, PRIDE XML) into one structured record per spectrum, carrying spectrum identifiers, controlled-vocabulary (cvParam) parameters, and m/z-intensity peak lists. jmzReader exposes all formats through a single common interface so downstream mzIdentML software needs only one parser API.

## When to use

Use when input is a peak list or XML-based MS data file (DTA, MS2, MGF, PKL, mzXML, mzData, mzML, PRIDE XML) referenced by mzIdentML, and the goal is extracting per-spectrum records (id, CV parameters, peak lists) into structured JSON.

## When NOT to use

- The input is an mzIdentML file itself — mzIdentML contains no spectra data, only references to external peak list files, so there is nothing to serialize from it directly.
- The goal is visualizing peak list files or converting them to MGF rather than producing structured per-spectrum records — use the jmzReader-gui export instead.
- The input is a raw or proprietary instrument format outside the library's supported set (DTA, MS2, MGF, PKL, mzXML, mzData, mzML, PRIDE XML).

## Inputs

- mzData XML file with binary peak arrays
- MS2 peak list text file (header fields: charge state, precursor m/z, retention time; plus m/z-intensity peak lists)
- Other supported peak list/XML MS formats: DTA, MGF, PKL, mzXML, mzML, PRIDE XML
- Maven pom.xml to which the jmzReader dependency is added

## Outputs

- JSON file with one record per spectrum, each carrying a spectrum identifier, CV parameter fields, and a non-empty m/z-intensity peak array (e.g. mzdata_spectra.json, ms2_spectra.json)
- Per-format, per-method spectrum-reference resolution report recording which lookup (byIndex, byID, byScanNumber) resolved which spectrum

## How to apply

Add the jmzReader parser as a Maven dependency (uk.ac.ebi.pride.tools:jmzreader) against the EBI pst-release Nexus repository, updating the version number to the latest release. Load the sample file with the parser matching its format (e.g. the mzData reader for mzData XML, the MS2 reader for MS2 peak lists) and iterate over all spectra. For each spectrum capture the identifier (spectrum id/index) and its selected cvParam parameters; for mzData, additionally decode the m/z and intensity peak arrays stored in the binary data blocks. Because mzIdentML files contain no spectra themselves and refer to external peak list files, resolve spectra through the single common interface using the three mzIdentML-style lookups — by spectrum index, by spectrum ID, and by scan number. Serialize one structured JSON record per spectrum (identifier, CV parameters, peak list) and flag any format/method combination whose lookup fails.

## Related tools

- **jmzReader** (Java API collection providing the format parsers (including the mzData and MS2 readers) and the single common interface supporting mzIdentML's external-spectra reference methods) — http://code.google.com/p/jmzreader/
- **jmzReader-gui** (Companion viewer for peak list files that demonstrates the jmzReader interface usage) — http://code.google.com/p/jmzreader/
- **Maven** (Build/dependency mechanism used to pull the jmzReader parser from the EBI pst-release Nexus repository)
- **jmzML** (Existing API on which the jmzReader mzML parser is based)

## Examples

```
<dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><version>LATEST</version><type>jar</type></dependency>
```

## Evaluation signals

- Output file exists and parses as valid JSON.
- JSON contains exactly one record per spectrum in the input peak list or mzData file.
- Every record includes a spectrum identifier and CV parameter fields.
- Every record's m/z-intensity peak array is non-empty (binary data blocks correctly decoded).
- Each of the three mzIdentML reference methods (byIndex, byID, byScanNumber) resolves the expected spectrum in every supported format; failures are explicitly flagged.

## Limitations

- The Maven snippet's version number is a placeholder that must be changed to the latest version, and the library is still evolving with more classes planned.
- mzIdentML files contain no spectra data; all resolution depends on the external peak list file being available and readable by the corresponding parser.
- Parsers handle arbitrarily large files only via the custom line-by-line text parsing class — memory efficiency applies to text-based peak list formats, not necessarily to XML binary blocks.

## Evidence
