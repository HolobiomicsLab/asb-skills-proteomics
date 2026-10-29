---
name: proteomics-mass-spectrometry-file-format-handling
description: Use when a Java/Maven proteomics project must read spectra, precursor
  metadata, and peak m/z-intensity lists from mzML, mzXML, mzData, PRIDE XML, MGF,
  DTA, MS2, or PKL files through one common parser interface.
when_to_use_negative:
- Input format is outside jmzReader's coverage (DTA, MS2, MGF, PKL, mzXML, mzData,
  mzML, PRIDE XML) — e.g. vendor RAW files or mzIdentML result files, which do not
  themselves contain spectra.
- The consumer project is not Java/Maven-based — jmzReader is a collection of Java
  APIs added via a pom dependency, so it is the wrong choice for pure Python or R
  parsing pipelines.
- The task is downstream analysis (e.g. peptide identification scoring or quantification)
  on spectra already loaded into memory — jmzReader parses and exposes the peak list
  files, it does not analyze them.
edam_operation: http://edamontology.org/operation_0227
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: jmzReader
  role: Core parser library exposing one common Java parser interface across DTA,
    MS2, MGF, PKL, mzXML, mzData, mzML, and PRIDE XML; declared as the uk.ac.ebi.pride.tools:jmzreader
    Maven dependency
  repo: https://github.com/PRIDE-Archive/jmzReader
- name: jmzML
  role: Existing Java API on which jmzReader's mzML parser is built, so mzML spectra
    are read without an independent parser
- name: Maven
  role: 'Dependency mechanism: jmzReader parsers are added to a project''s pom, pointing
    at the EBI Nexus pst-release/pst-snapshots repositories'
- name: jmzReader-gui
  role: Visualizes peak list files, demonstrates usage of the jmzReader interface,
    and exports all supported formats to mgf
  repo: https://github.com/PRIDE-Archive/jmzReader
- name: Java
  role: Language in which the parser APIs are implemented and from which spectra iteration
    and peak extraction are coded
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:8c8cfe46f32390f12227168f410a68db16623cbfca23a8658620181fb6784850
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PRIDE-Archive/jmzReader
  tools:
  - Java
  - Maven
  - jmzML
  - jmzReader
  - jmzReader-gui
  edam_operation: http://edamontology.org/operation_0227
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# proteomics-mass-spectrometry-file-format-handling

## Summary

Parse the common proteomics MS peak list and XML formats (DTA, MS2, MGF, PKL, mzXML, mzData, mzML, PRIDE XML) in Java via the jmzReader library, whose mzML support reuses the existing jmzML API. All parsers implement one common interface optimized for resolving the external peak list files referenced by mzIdentML.

## When to use

Use when a Java/Maven proteomics project must read spectra, precursor metadata, and peak m/z-intensity lists from mzML, mzXML, mzData, PRIDE XML, MGF, DTA, MS2, or PKL files through one common parser interface.

## When NOT to use

- Input format is outside jmzReader's coverage (DTA, MS2, MGF, PKL, mzXML, mzData, mzML, PRIDE XML) — e.g. vendor RAW files or mzIdentML result files, which do not themselves contain spectra.
- The consumer project is not Java/Maven-based — jmzReader is a collection of Java APIs added via a pom dependency, so it is the wrong choice for pure Python or R parsing pipelines.
- The task is downstream analysis (e.g. peptide identification scoring or quantification) on spectra already loaded into memory — jmzReader parses and exposes the peak list files, it does not analyze them.

## Inputs

- mzML files containing spectra with peak lists (read via the jmzReader parser built on the jmzML API)
- Peak list and XML-based MS files in DTA, MS2, MGF, PKL, mzXML, mzData, or PRIDE XML format
- A Maven pom file into which the uk.ac.ebi.pride.tools:jmzreader dependency and EBI Nexus repositories are added

## Outputs

- Per-spectrum records with index, MS level, precursor m/z and charge (where present), and peak m/z-intensity arrays (e.g. written as mzml_spectra.tsv, one row per spectrum)
- Spectrum objects accessed through the common jmzReader parser interface, usable to resolve mzIdentML references to external peak list files
- mgf-format exports of any supported input format (via jmzReader-gui)

## How to apply

Declare the uk.ac.ebi.pride.tools:jmzreader artifact (type jar) in the Maven pom and register the EBI Nexus pst-release and pst-snapshots repositories, changing the version number to the latest release rather than using the placeholder. Instantiate the parser matching the input format — for mzML this is the jmzReader parser built on the already existing jmzML API — and iterate the spectra through the common jmzReader interface so the same code covers DTA, MS2, MGF, PKL, mzXML, mzData, and PRIDE XML. For each spectrum, record its index and MS level, extract precursor ion information (precursor m/z and charge, where present) from the file metadata, and pull out the peak m/z and intensity arrays. Rely on the custom line-by-line text-parsing class, which lets the parsers handle arbitrarily large files in minimal memory. When interoperating with mzIdentML, use the interface methods that support the various ways mzIdentML references external spectra instead of writing per-format access code; jmzReader-gui can additionally visualize or export any supported format to mgf.

## Related tools

- **jmzReader** (Core parser library exposing one common Java parser interface across DTA, MS2, MGF, PKL, mzXML, mzData, mzML, and PRIDE XML; declared as the uk.ac.ebi.pride.tools:jmzreader Maven dependency) — https://github.com/PRIDE-Archive/jmzReader
- **jmzML** (Existing Java API on which jmzReader's mzML parser is built, so mzML spectra are read without an independent parser)
- **Maven** (Dependency mechanism: jmzReader parsers are added to a project's pom, pointing at the EBI Nexus pst-release/pst-snapshots repositories)
- **jmzReader-gui** (Visualizes peak list files, demonstrates usage of the jmzReader interface, and exports all supported formats to mgf) — https://github.com/PRIDE-Archive/jmzReader
- **Java** (Language in which the parser APIs are implemented and from which spectra iteration and peak extraction are coded)

## Examples

```
<dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><version>latest</version><type>jar</type></dependency>
```

## Evaluation signals

- Output table (e.g. mzml_spectra.tsv) exists, is well-formed TSV, and contains the required columns: index, MS level, precursor m/z, precursor charge, and peak m/z-intensity arrays.
- Row count of the output equals the number of spectra in the input mzML file.
- Precursor fields are populated only where the format metadata provides them (precursor m/z and charge 'where present'); spectra without precursors must not break the run.
- The same iteration code runs unchanged across formats (mzML, mzXML, mzData, MGF, DTA, MS2, PKL, PRIDE XML) because all parsers implement the common jmzReader interface.
- Maven resolution of uk.ac.ebi.pride.tools:jmzreader succeeds against the EBI Nexus repositories with the version number set to the latest release (not the README placeholder).

## Limitations

- The Maven snippet ships a placeholder version — 'you need to change the version number to the latest version' — so copying it verbatim will not resolve a real artifact.
- The README's pst-release repository URL is malformed ('hhttps://ww.ebi.ac.uk/Tools/maven/repos/content/repositories/pst-release'); dependency resolution fails unless the EBI Nexus URL is corrected.
- The README notes 'the library is still evolving' and committed to adding more classes, so the API surface may change.
- mzIdentML files do not contain spectra data but refer to external peak list files — jmzReader parses those external peak list files, not mzIdentML itself.

## Evidence
