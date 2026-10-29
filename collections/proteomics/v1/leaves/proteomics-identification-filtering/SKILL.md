---
name: proteomics-identification-filtering
description: Use when MS-driven proteomics spectra, identifications and quantification
  results are stored in a central ms_lims database and the goal is to extract filtered
  peptide or protein lists with per-project descriptive statistics.
when_to_use_negative:
- Input is raw mass spectra that still require processing, identification or quantification
  — ms_lims assumes these results are already stored in the central database
- The task is manipulation of FASTA sequence databases (merging, shuffling, truncating)
  — use dbtoolkit instead
- No populated ms_lims database exists and data reside in unrelated flat files or
  other LIMS formats
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0622
tools:
- name: ms-lims
  role: Client application providing the Project Analyzer and its pluggable analysis
    tools that query the central database and filter peptide/protein identifications
  repo: https://github.com/compomics/ms-lims/wiki/ReleaseNotes
- name: compomics-utilities
  role: Library on which the ms-lims pluggable analysis tools are built for identification
    filtering and statistics
  repo: http://code.google.com/p/compomics-utilities
- name: Java
  role: Runtime environment for launching the ms-lims jar client
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.200900409
    title: ms_lims, a simple yet powerful open source laboratory information management
      system for MS‐driven proteomics
  source_content_hash: sha256:96e364b6c988e75ac0ecd5f10994ecdccfe62100a225c732ed88dd3fe4d0e9c0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - compomics-utilities
  - ms-lims
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# proteomics-identification-filtering

## Summary

Interrogate a populated central ms_lims database using the Project Analyzer's pluggable analysis tools (built on compomics-utilities) to produce filtered peptide/protein lists with descriptive statistics for a selected proteomics project.

## When to use

Use when MS-driven proteomics spectra, identifications and quantification results are stored in a central ms_lims database and the goal is to extract filtered peptide or protein lists with per-project descriptive statistics.

## When NOT to use

- Input is raw mass spectra that still require processing, identification or quantification — ms_lims assumes these results are already stored in the central database
- The task is manipulation of FASTA sequence databases (merging, shuffling, truncating) — use dbtoolkit instead
- No populated ms_lims database exists and data reside in unrelated flat files or other LIMS formats

## Inputs

- Populated central ms_lims database containing processed mass spectra, identifications and quantification results
- ms-lims 7.7.7 zip download containing the ms-lims jar file and all needed libraries
- Selected proteomics project identifier within the database

## Outputs

- Filtered peptide/protein list for the selected project
- Descriptive statistics (counts and per-project summary values) over the filtered list
- Exported project analysis output (e.g. project_analysis_output.csv)

## How to apply

Launch the ms-lims client by double-clicking the ms-lims jar file from the ms-lims 7.7.7 download, and connect it to the populated central ms_lims database holding processed mass spectra, identifications and quantification results. In the Project Analyzer, select the target proteomics project to analyze. Load a pluggable analysis tool — ms-lims makes use of compomics-utilities — that queries identifications for the selected project and applies its filtering criteria to the peptide and protein hits. Compute descriptive statistics (counts and per-project summary values) over the filtered peptide/protein list. Export the filtered list and statistics for downstream inspection or validation.

## Related tools

- **ms-lims** (Client application providing the Project Analyzer and its pluggable analysis tools that query the central database and filter peptide/protein identifications) — https://github.com/compomics/ms-lims/wiki/ReleaseNotes
- **compomics-utilities** (Library on which the ms-lims pluggable analysis tools are built for identification filtering and statistics) — http://code.google.com/p/compomics-utilities
- **Java** (Runtime environment for launching the ms-lims jar client)

## Examples

```
unzip ms-lims-7.7.7.zip && java -jar ms-lims-7.7.7/ms-lims.jar  # connect to the central ms_lims database and use the Project Analyzer
```

## Evaluation signals

- The exported project_analysis_output.csv exists and contains peptide/protein fields for the requested project
- The output presents the filtered identification list together with the descriptive statistics computed from the central database
- Statistics (counts, per-project summary values) match the filtered peptide/protein list actually exported
- The ms-lims client successfully connected to the central ms_lims database and reports identifications for the selected project

## Limitations

- The skill depends on a pre-populated central ms_lims database; raw spectra must be processed, identified and possibly quantified before interpretation can be undertaken
- Analysis capability is limited to the pluggable tools loaded from the database; each tool performs one specific data analysis task
- Issues or feature requests must be reported via the GitHub issues section of compomics/ms-lims

## Evidence

- [other] The Project Analyzer is an easily extensible ms_lims application that loads a series of pluggable tools from the database, each performing a specific data analysis task such as filtered peptide or protein lists and descriptive statistics for a project.: [evidence span withheld — anchor status: not_found]
