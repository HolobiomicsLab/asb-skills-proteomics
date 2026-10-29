---
name: peptide-protein-list-statistics
description: Use when MS-driven proteomics spectra, identifications and quantification
  results are already stored in a central ms_lims database and you need project-level
  filtered peptide/protein lists with descriptive statistics.
when_to_use_negative:
- Input is raw mass spectra not yet processed, identified or quantified — ms_lims
  manages and interprets processed results stored in its central database, it does
  not replace the identification pipeline.
- The data are FASTA sequence databases needing merging, shuffling or truncation —
  use dbtoolkit for that manipulation task instead.
- No ms_lims central database is available (e.g. scattered local result files) — this
  skill requires the client to connect to a populated central ms_lims database.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: ms_lims / ms-lims 7.7.7
  role: Client application (started from the ms-lims jar) that connects to the central
    database and hosts the Project Analyzer and its pluggable analysis tools
  repo: https://github.com/compomics/ms-lims/wiki
- name: compomics-utilities
  role: Library on which the pluggable Project Analyzer tools are built to query identifications
    and compute filtered lists and statistics
  repo: http://code.google.com/p/compomics-utilities
- name: Java
  role: Runtime/language for building and running the ms-lims client and its pluggable
    tools
  repo: http://java.com/en/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.200900409
    title: ms_lims, a simple yet powerful open source laboratory information management
      system for MS‐driven proteomics
  source_content_hash: sha256:4ba28d50de17771678f40405989adf9d69d221290c84258e2d85753289bde3be
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - compomics-utilities
  - ms_lims / ms-lims 7.7.7
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# peptide-protein-list-statistics

## Summary

Interrogate a populated ms_lims central database through the Project Analyzer's pluggable analysis tools to yield filtered peptide/protein lists and descriptive statistics for a selected proteomics project.

## When to use

Use when MS-driven proteomics spectra, identifications and quantification results are already stored in a central ms_lims database and you need project-level filtered peptide/protein lists with descriptive statistics.

## When NOT to use

- Input is raw mass spectra not yet processed, identified or quantified — ms_lims manages and interprets processed results stored in its central database, it does not replace the identification pipeline.
- The data are FASTA sequence databases needing merging, shuffling or truncation — use dbtoolkit for that manipulation task instead.
- No ms_lims central database is available (e.g. scattered local result files) — this skill requires the client to connect to a populated central ms_lims database.

## Inputs

- populated central ms_lims database (processed mass spectra, identifications, quantification results)
- target proteomics project selected in the Project Analyzer
- ms-lims 7.7.7 jar file plus bundled libraries

## Outputs

- filtered peptide list for the selected project
- filtered protein list for the selected project
- descriptive statistics (counts and per-project summary values) computed from the central database
- exportable project analysis output (e.g. project_analysis_output.csv)

## How to apply

Start the ms-lims client from the ms-lims jar file included in the ms-lims 7.7.7 download and connect it to the populated central ms_lims database holding processed mass spectra, identifications and quantification results. In the Project Analyzer, select the target proteomics project to analyze. The Project Analyzer is extensible and loads a series of pluggable tools from the database, each built on compomics-utilities; pick the tool matching the analysis task (filtered peptide list, filtered protein list, or descriptive statistics). Each tool queries the identifications stored for the selected project and applies its filtering criteria to the peptide and protein hits before computing per-project summary values. Judge success by the resulting output: the filtered identification list and the descriptive statistics must be traceable to the requested project's rows in the central database.

## Related tools

- **ms_lims / ms-lims 7.7.7** (Client application (started from the ms-lims jar) that connects to the central database and hosts the Project Analyzer and its pluggable analysis tools) — https://github.com/compomics/ms-lims/wiki
- **compomics-utilities** (Library on which the pluggable Project Analyzer tools are built to query identifications and compute filtered lists and statistics) — http://code.google.com/p/compomics-utilities
- **Java** (Runtime/language for building and running the ms-lims client and its pluggable tools) — http://java.com/en/

## Examples

```
java -jar ms-lims.jar
```

## Evaluation signals

- The ms-lims client successfully connects to the populated central database and lists the processed spectra, identifications and quantification results it holds.
- The Project Analyzer loads the pluggable analysis tools from the database for the selected project.
- The filtered peptide/protein list only contains hits whose identifications belong to the selected project in the central database.
- The descriptive statistics (counts and per-project summary values) recomputed from the database rows match the exported output.
- The exported output (e.g. project_analysis_output.csv) exists and contains peptide/protein fields for the requested project alongside the computed statistics.

## Limitations

- Requires the ms_lims central database to be populated and accessible; the client cannot analyze data that were never processed, identified and stored.
- Analysis capabilities are limited to the pluggable tools stored in the database; new analysis tasks require loading/extending additional tools.
- ms-lims depends on a Java runtime environment; the zip download bundles the jar and all libraries needed to run it.

## Evidence
