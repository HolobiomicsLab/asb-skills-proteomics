---
name: mass-spectrometry-spectrum-archival-schema-design
description: Use when a laboratory needs a central relational database schema to store,
  link, and archive mass spectra, peptide identifications, and quantitative results
  from MS-driven proteomics experiments.
when_to_use_negative:
- Input is already a populated LIMS database — this skill covers schema design and
  installation, not data migration or importing existing identifications
- The need is spectrum processing, identification, or quantification itself (e.g.
  via Peptizer) rather than storage of the results
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: ms_lims
  role: Open-source central-database system whose schema is being reconstructed and
    installed
  repo: https://github.com/compomics/ms-lims
- name: ms-lims 7.7.7
  role: Release zip providing the ms-lims jar and libraries; its schema diagram and
    configuration tool drive the deployment
  repo: https://github.com/compomics/ms-lims
- name: Java
  role: Runtime for the ms-lims database configuration tool used to create the schema
- name: compomics-utilities
  role: Library dependency used by ms-lims
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.200900409
    title: ms_lims, a simple yet powerful open source laboratory information management
      system for MS‐driven proteomics
  source_content_hash: sha256:b39a650b5fd2cef84a848b300a2b467c74bfac6cfce0c511688950c18c842f76
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/compomics/ms-lims
  tools:
  - Java
  - compomics-utilities
  - ms-lims 7.7.7
  - ms_lims
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# mass-spectrometry-spectrum-archival-schema-design

## Summary

Design and deploy a peptide-identification-centric relational schema for MS-driven proteomics LIMS, following the ms_lims central-database layout in which projects contain LC runs, spectra are grouped per run, and peptide identifications link to spectra with gzipped archival of spectra and search-engine outputs.

## When to use

Use when a laboratory needs a central relational database schema to store, link, and archive mass spectra, peptide identifications, and quantitative results from MS-driven proteomics experiments.

## When NOT to use

- Input is already a populated LIMS database — this skill covers schema design and installation, not data migration or importing existing identifications
- The need is spectrum processing, identification, or quantification itself (e.g. via Peptizer) rather than storage of the results

## Inputs

- ms-lims 7.7.7 release zip containing the ms-lims jar and bundled libraries
- Published schema diagram (schema_7.6.png) from genesis.ugent.be
- ms_lims_manual.pdf database manual
- DatabaseUpdates wiki page describing schema changes between 7.6 and 7.7.7

## Outputs

- Relational DDL script (e.g. ms_lims_schema.sql) implementing the peptide-identification-centric central schema
- Installed ms_lims schema on MySQL (and optionally PostgreSQL/Oracle)
- Verified table set matching schema_7.6.png after applying DatabaseUpdates ALTER statements

## How to apply

Download the ms-lims 7.7.7 release zip from genesis.ugent.be and unpack it to obtain the ms-lims jar and bundled libraries. Open the published schema diagram (schema_7.6.png) and enumerate its tables and foreign-key relations, then author DDL (e.g. ms_lims_schema.sql) reproducing the central-database layout: projects containing LC runs, spectra grouped per run, peptide identifications linked to spectra, and gzipped archival of spectra and original search-engine outputs. Consult the wiki DatabaseUpdates page for schema changes between 7.6 and 7.7.7 and apply any required ALTER statements. Run the Java-based ms-lims database configuration tool against a local MySQL instance to create the schema, and repeat on PostgreSQL/Oracle if available.

## Related tools

- **ms_lims** (Open-source central-database system whose schema is being reconstructed and installed) — https://github.com/compomics/ms-lims
- **ms-lims 7.7.7** (Release zip providing the ms-lims jar and libraries; its schema diagram and configuration tool drive the deployment) — https://github.com/compomics/ms-lims
- **Java** (Runtime for the ms-lims database configuration tool used to create the schema)
- **compomics-utilities** (Library dependency used by ms-lims)

## Examples

```
java -jar ms-lims-7.7.7.jar
```

## Evaluation signals

- Every table shown in schema_7.6.png exists in the installed database
- The ms-lims database configuration tool completes without SQL errors
- Foreign-key relations connect peptide identifications to their spectra, and spectra to LC runs within projects
- Quantitative information, when available, is linked to peptide identifications
- All DatabaseUpdates ALTER statements between 7.6 and 7.7.7 have been applied to the schema

## Limitations

- The published schema diagram corresponds to version 7.6; 7.7.7 may require schema updates listed on the DatabaseUpdates wiki page
- The schema is peptide-identification-centric for MS-driven proteomics and does not cover general-purpose laboratory workflows outside MS data management
- Only MySQL is demonstrated in the deployment workflow; PostgreSQL/Oracle support is conditional on availability

## Evidence

- [abstract] Central database automates MS data management: "We here present ms_lims (http://genesis.UGent.be/ms_lims), a freely available, open-source system based on a central database to automate data management and processing in MS-driven proteomics"
- [readme] Schema diagram download: "Download the database schema [here](http://genesis.ugent.be/uvpublicdata/ms-lims/schema_7.6.png)."
- [readme] Database updates wiki: "Check out the [release notes](https://github.com/compomics/ms-lims/wiki/ReleaseNotes) and [database updates](https://github.com/compomics/ms-lims/wiki/DatabaseUpdates) in the"
