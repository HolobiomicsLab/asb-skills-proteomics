---
name: peak-mz-intensity-structuring
description: Use when input is a raw peak list file in DTA (or MS2/MGF/PKL/mzXML/mzData/mzML)
  format and you need per-spectrum structured records of precursor mass, charge state,
  and m/z–intensity peak pairs.
when_to_use_negative:
- Input is already a structured/converted format such as mgf exported from jmzReader-gui
  or an mzIdentML file, which contains no spectra but only references to external
  peak list files
- The task requires visualization rather than programmatic parsing — jmzReader-gui
  is the tool for viewing peak list files
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: jmzReader
  role: Java API providing the DTA (and MS2/MGF/PKL/mzXML/mzData/mzML) parsers implementing
    a common interface used to produce the structured spectrum records
  repo: http://code.google.com/p/jmzreader/
- name: Java
  role: Programming language in which the parser is implemented and consumed through
    the jmzReader interfaces
- name: jmzReader-gui
  role: Optional viewer to inspect supported peak list files and export them to mgf
    format for cross-checking
  repo: http://code.google.com/p/jmzreader/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:fb5ec5b3cbfe366f98fca8550c8667e6e721ae5e2ee35f5e733325517666acfe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - jmzReader
  - jmzReader-gui
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# peak-mz-intensity-structuring

## Summary

Parses text-based MS peak list formats (DTA, MS2, MGF, PKL, mzXML, mzData, mzML) into structured spectrum records carrying precursor mass, charge state, and paired peak m/z–intensity lists, using the jmzReader Java API and its common parser interface.

## When to use

Use when input is a raw peak list file in DTA (or MS2/MGF/PKL/mzXML/mzData/mzML) format and you need per-spectrum structured records of precursor mass, charge state, and m/z–intensity peak pairs.

## When NOT to use

- Input is already a structured/converted format such as mgf exported from jmzReader-gui or an mzIdentML file, which contains no spectra but only references to external peak list files
- The task requires visualization rather than programmatic parsing — jmzReader-gui is the tool for viewing peak list files

## Inputs

- DTA-format peak list text file (or MS2, MGF, PKL, mzXML, mzData, mzML supported by jmzReader)
- Sample DTA file from the jmzReader repository test resources for validation

## Outputs

- One structured record per spectrum (JSON) containing charge state, precursor mass, and a list of peak m/z–intensity pairs

## How to apply

Add jmzReader to the Maven project via the uk.ac.ebi.pride.tools:jmzreader dependency, or read the peak list file line by line mirroring jmzReader's custom line-by-line text-file parsing class, which lets parsers handle arbitrarily large files in minimal memory. For DTA, parse the first whitespace-separated line of each spectrum to extract the precursor mass and charge state, then parse each subsequent line as a whitespace-separated m/z–intensity peak pair. Emit one structured record per spectrum (e.g. JSON) containing charge state, precursor mass, and the ordered list of parsed peak m/z and intensity values, honoring the common jmzReader parser interface semantics so the same code path applies across peak list formats. Validate by running the parser on a sample DTA file from the jmzReader repository test resources and confirming one record per input spectrum, each with a charge state field and non-empty m/z–intensity pairs.

## Related tools

- **jmzReader** (Java API providing the DTA (and MS2/MGF/PKL/mzXML/mzData/mzML) parsers implementing a common interface used to produce the structured spectrum records) — http://code.google.com/p/jmzreader/
- **Java** (Programming language in which the parser is implemented and consumed through the jmzReader interfaces)
- **jmzReader-gui** (Optional viewer to inspect supported peak list files and export them to mgf format for cross-checking) — http://code.google.com/p/jmzreader/

## Examples

```
<dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><version>LATEST</version><type>jar</type></dependency>
```

## Evaluation signals

- dta_parsed_records exists after the run and contains exactly one record per spectrum in the input DTA file
- Every record carries a charge state field plus non-empty m/z–intensity peak pairs
- Precursor mass and charge state in each record match the first whitespace-separated header line of the corresponding input spectrum
- Peak pair counts and m/z values are invariant whether parsed via the jmzReader interface or via direct line-by-line parsing
- Parser completes on a large peak list file within minimal memory, consistent with jmzReader's line-by-line parsing design

## Limitations

- jmzReader version number is not pinned in the documentation; the user must substitute the latest version in the Maven dependency
- mzIdentML files do not contain spectra data but refer to external peak list files, so this skill cannot extract peaks from mzIdentML directly
- The library is still evolving; the API may expand or change

## Evidence
