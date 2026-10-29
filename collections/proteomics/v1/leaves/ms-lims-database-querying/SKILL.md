---
name: ms-lims-database-querying
description: Use when MS-driven proteomics spectra, identifications and quantification
  results are already stored in a populated ms_lims central database and need to be
  interrogated, filtered, statistically summarized, or exported (e.g., for PRIDE submission).
when_to_use_negative:
- Input is raw instrument data or unprocessed spectra that have not yet been processed,
  identified or quantified into the ms_lims central database
- The proteomics data are managed outside ms_lims (e.g., an unrelated LIMS or flat
  files), since the Project Analyzer tools and Pride Converter coupling query the
  ms_lims schema specifically
- FASTA sequence database manipulation is needed — dbtoolkit is the CompOmics tool
  for merging, shuffling and truncating sequence databases, not ms_lims
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0622
tools:
- name: ms-lims
  role: Client application whose jar file connects to the populated central ms_lims
    database and runs the Project Analyzer for querying and filtered analysis
  repo: https://github.com/compomics/ms-lims
- name: compomics-utilities
  role: Library on which ms-lims is built and on which the pluggable Project Analyzer
    analysis tools rely
  repo: http://code.google.com/p/compomics-utilities
- name: Maven
  role: Build tool used for the ms-lims Java codebase that connects to the central
    database
- name: Java
  role: Language/runtime used to run the ms-lims jar and its bundled libraries
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1002/pmic.200900409
    title: ms_lims, a simple yet powerful open source laboratory information management
      system for MS‐driven proteomics
  source_content_hash: sha256:0ba2b52809038acafdcb29f44c0b8d4d62d942378c3861efb1fc1e686cf7dec3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/compomics/ms-lims
  tools:
  - Java
  - Maven
  - compomics-utilities
  - ms-lims
  - ms_lims
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ms-lims-database-querying

## Summary

Query and explore a populated ms_lims central database holding processed mass spectra, peptide/protein identifications and quantification results, using the ms-lims client (notably the Project Analyzer) to retrieve filtered peptide/protein lists and descriptive statistics for a project.

## When to use

Use when MS-driven proteomics spectra, identifications and quantification results are already stored in a populated ms_lims central database and need to be interrogated, filtered, statistically summarized, or exported (e.g., for PRIDE submission).

## When NOT to use

- Input is raw instrument data or unprocessed spectra that have not yet been processed, identified or quantified into the ms_lims central database
- The proteomics data are managed outside ms_lims (e.g., an unrelated LIMS or flat files), since the Project Analyzer tools and Pride Converter coupling query the ms_lims schema specifically
- FASTA sequence database manipulation is needed — dbtoolkit is the CompOmics tool for merging, shuffling and truncating sequence databases, not ms_lims

## Inputs

- Populated ms_lims central database (processed mass spectra, peptide/protein identifications, quantification results)
- ms-lims 7.7.7 distribution zip containing the ms-lims jar file and all libraries
- schema_7.6 database schema diagram
- ms_lims manual (ms_lims_manual.pdf)

## Outputs

- Filtered peptide/protein lists for a selected project
- Descriptive statistics over the filtered list (counts and per-project summary values)
- Queried identification/spectrum record sets (e.g., for PRIDE export via Pride Converter)

## How to apply

Obtain the ms-lims 7.7.7 distribution zip, which bundles the ms-lims jar file plus all libraries needed to run the tool, and start it (double-clicking the jar launches ms-lims). Connect the client to the populated central ms_lims database that stores the processed spectra, identifications and quantification results; consult the schema_7.6 database schema diagram and the ms_lims manual to map the tables holding identification and spectrum records. For data analysis, use the Project Analyzer, an easily extensible ms_lims application that loads a series of pluggable tools from the database — each tool queries identifications for a selected project, applies its filtering criteria to peptide and protein hits, and computes descriptive statistics (counts and per-project summary values) over the filtered list. The underlying libraries are built with Java/Maven and rely on compomics-utilities. For repository submission, the same database coupling lets Pride Converter query stored identifications together with their associated mass spectra and transform them into PRIDE format.

## Related tools

- **ms-lims** (Client application whose jar file connects to the populated central ms_lims database and runs the Project Analyzer for querying and filtered analysis) — https://github.com/compomics/ms-lims
- **compomics-utilities** (Library on which ms-lims is built and on which the pluggable Project Analyzer analysis tools rely) — http://code.google.com/p/compomics-utilities
- **Maven** (Build tool used for the ms-lims Java codebase that connects to the central database)
- **Java** (Language/runtime used to run the ms-lims jar and its bundled libraries)

## Examples

```
java -jar ms-lims-7.7.7.jar
```

## Evaluation signals

- The output file (e.g., project_analysis_output.csv) exists, contains peptide/protein fields for the requested project, and presents the filtered identification list together with descriptive statistics computed from the central database
- Exported identification/spectrum pairs (e.g., in pride_export.xml) are consistent with the records in the source ms_lims database
- An exported PRIDE-format file parses as a well-formed PRIDE submission
- The client successfully connects to the central database and the schema_7.6 tables map correctly to the queried identification and spectrum records
- Per-project summary counts in the descriptive statistics match the identifications stored for the selected project

## Limitations

- The database schema (schema_7.6) and database updates must be consulted separately via the wiki; releases may require database update steps
- Functionality depends on the pluggable Project Analyzer tools loaded from the database; analysis capability is limited to the tools present
- Issues and feature requests are handled only through the GitHub issues section; no alternative support channel is documented

## Evidence
