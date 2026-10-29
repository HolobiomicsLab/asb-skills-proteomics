---
name: pride-submission-format-conversion
description: Use when stored ms_lims identification and spectrum records must be exported
  to PRIDE submission XML (pride_export.xml) for deposition to public proteomics data
  repositories.
when_to_use_negative:
- Proteomics data are not stored in the ms_lims central database (e.g. raw vendor
  files or flat search results) — first populate the ms_lims database before exporting
- The target repository format is not PRIDE (e.g. submission to a non-PRIDE archive
  requires a different converter)
- Data management rather than repository submission is the goal — ms_lims itself handles
  automated data management and processing of MS-driven proteomics analyses
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ms-lims
  role: Provides the ms-lims jar and libraries used to connect to the populated central
    database and query stored identifications with associated mass spectra before
    PRIDE conversion
  repo: https://github.com/compomics/ms-lims/wiki/ReleaseNotes
- name: compomics-utilities
  role: Library dependency of ms-lims underpinning the transformation of queried database
    records into the PRIDE submission format
  repo: http://code.google.com/p/compomics-utilities
- name: Pride Converter
  role: Tool coupled directly to the populated ms_lims database to greatly facilitate
    submission of data to PRIDE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.200900409
    title: ms_lims, a simple yet powerful open source laboratory information management
      system for MS‐driven proteomics
  source_content_hash: sha256:007c8cb127e58dd432f5e091b32793d85c2090112c1a4321bac3a9c3345041c2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Pride Converter
  - compomics-utilities
  - ms-lims
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# pride-submission-format-conversion

## Summary

Converts identifications and associated mass spectra stored in a populated ms_lims central database into PRIDE submission format (pride_export.xml), enabling direct submission of MS-driven proteomics data to the PRIDE public repository.

## When to use

Use when stored ms_lims identification and spectrum records must be exported to PRIDE submission XML (pride_export.xml) for deposition to public proteomics data repositories.

## When NOT to use

- Proteomics data are not stored in the ms_lims central database (e.g. raw vendor files or flat search results) — first populate the ms_lims database before exporting
- The target repository format is not PRIDE (e.g. submission to a non-PRIDE archive requires a different converter)
- Data management rather than repository submission is the goal — ms_lims itself handles automated data management and processing of MS-driven proteomics analyses

## Inputs

- Populated ms_lims central database holding identification and spectrum records
- schema_7.6 database schema (schema_7.6.png)
- ms_lims manual (ms_lims_manual.pdf)
- ms-lims 7.7.7 distribution zip with bundled jar and libraries

## Outputs

- pride_export.xml (well-formed PRIDE-format submission file containing identification/spectrum pairs)

## How to apply

Obtain the ms-lims 7.7.7 distribution zip, which bundles the ms-lims jar and all libraries needed to run the tool. Consult the schema_7.6 database schema image and the ms_lims manual to map the central-database tables holding identification and spectrum records. Using the bundled ms-lims jar libraries (built with Java/Maven and relying on compomics-utilities), connect to the populated central database and query the stored identifications together with their associated mass spectra. Couple the populated database to the Pride Converter tool to transform the queried records into the PRIDE submission format and write them to pride_export.xml. The rationale is that ms_lims' central database is designed to automate data management, and coupling it to Pride Converter greatly facilitates PRIDE submission.

## Related tools

- **ms-lims** (Provides the ms-lims jar and libraries used to connect to the populated central database and query stored identifications with associated mass spectra before PRIDE conversion) — https://github.com/compomics/ms-lims/wiki/ReleaseNotes
- **compomics-utilities** (Library dependency of ms-lims underpinning the transformation of queried database records into the PRIDE submission format) — http://code.google.com/p/compomics-utilities
- **Pride Converter** (Tool coupled directly to the populated ms_lims database to greatly facilitate submission of data to PRIDE)

## Evaluation signals

- pride_export.xml exists as an output file
- pride_export.xml parses as a well-formed PRIDE-format file
- Exported identification/spectrum pairs are consistent with the records in the source ms_lims central database
- The queried identifications and associated mass spectra cover the central-database tables mapped via the schema_7.6 schema

## Limitations

- Requires a populated ms_lims central database; ms_lims only manages data once spectra have been processed, identified and quantified upstream
- Version-specific dependencies: ms-lims 7.7.7 distribution and the schema_7.6 database schema; database updates are documented in the wiki
- Issues or feature requests must be reported via the GitHub issues section; no in-conversion error handling is described

## Evidence
