---
name: protein-progenitor-mapping
description: Use when storing Mascot Server peptide identifications into ms_lims and
  each identification must carry a full progenitor protein set plus a single selected
  primary protein hit.
when_to_use_negative:
- Input is not search-engine output (e.g. already a processed protein-level table)
  — the skill operates on Mascot peptide identifications
- The analysis requires de novo peptide sequencing or quantification rather than linking
  stored peptide identifications to progenitor proteins
- The central ms_lims database schema has not been set up (see schema_7.6.png and
  the DatabaseUpdates wiki), since identifications are persisted following that published
  schema
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: ms-lims 7.7.7
  role: LIMS whose central relational database stores identifications, spectrum links,
    progenitor protein lists, and primary hits following the schema_7.6.png schema
  repo: https://github.com/compomics/ms-lims
- name: compomics-utilities
  role: Library used by ms-lims as a dependency when building the parsing component
    in Java with Maven
  repo: http://code.google.com/p/compomics-utilities
- name: dbtoolkit
  role: Related CompOmics project for manipulating FASTA sequence databases, useful
    for preparing the protein database against which progenitor proteins are resolved
  repo: https://github.com/compomics/dbtoolkit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.200900409
    title: ms_lims, a simple yet powerful open source laboratory information management
      system for MS‐driven proteomics
  source_content_hash: sha256:a7cc3a393c127cfa4eb99325ac72aab50d9d42405b0fca1d2380886fd1dfc693
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/compomics/ms-lims
  tools:
  - compomics-utilities
  - dbtoolkit
  - ms-lims 7.7.7
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# protein-progenitor-mapping

## Summary

Resolves, for every stored peptide identification in ms_lims, the complete set of progenitor proteins that contain the peptide sequence and selects one representative primary protein hit using the Mueller et al. (2006) algorithm, so protein attribution is consistent across all identifications in the central ms_lims database.

## When to use

Use when storing Mascot Server peptide identifications into ms_lims and each identification must carry a full progenitor protein set plus a single selected primary protein hit.

## When NOT to use

- Input is not search-engine output (e.g. already a processed protein-level table) — the skill operates on Mascot peptide identifications
- The analysis requires de novo peptide sequencing or quantification rather than linking stored peptide identifications to progenitor proteins
- The central ms_lims database schema has not been set up (see schema_7.6.png and the DatabaseUpdates wiki), since identifications are persisted following that published schema

## Inputs

- Mascot Server search-engine output files
- Peptide identifications with identifiers of the mass spectra they were derived from
- ms-lims 7.7.7 codebase and compomics-utilities library (Maven/Java project)
- Published ms-lims database schema (schema_7.6.png)

## Outputs

- identifications_with_primary_hits.csv containing spectrum links, all progenitor proteins, and the primary protein hit per peptide
- Peptide identifications stored in the central ms_lims relational database, each linked to its source spectrum
- Per-peptide progenitor protein sets with one algorithmically selected primary protein hit

## How to apply

Parse each Mascot Server search-engine output file and extract every peptide identification together with the identifier of the mass spectrum it was derived from. For each stored peptide identification, resolve the complete set of progenitor proteins that contain the peptide sequence rather than reporting only one protein. Apply the primary-hit selection algorithm from reference [7] (Mueller et al., 2006) to choose one representative protein per peptide from its progenitor set, while retaining all other progenitor proteins in the database structure. Persist identifications, spectrum links, progenitor protein lists, and selected primary hits into the central relational store following the published ms-lims schema (schema_7.6.png), then export the full identifications table to identifications_with_primary_hits.csv.

## Related tools

- **ms-lims 7.7.7** (LIMS whose central relational database stores identifications, spectrum links, progenitor protein lists, and primary hits following the schema_7.6.png schema) — https://github.com/compomics/ms-lims
- **compomics-utilities** (Library used by ms-lims as a dependency when building the parsing component in Java with Maven) — http://code.google.com/p/compomics-utilities
- **dbtoolkit** (Related CompOmics project for manipulating FASTA sequence databases, useful for preparing the protein database against which progenitor proteins are resolved) — https://github.com/compomics/dbtoolkit

## Examples

```
java -jar ms-lims-7.7.7.jar
```

## Evaluation signals

- identifications_with_primary_hits.csv exists after the workflow completes
- Every row in the output CSV contains a non-empty primary protein hit
- Every row contains the complete progenitor protein set for its peptide, not just a subset
- Each stored identification is linked to the identifier of the mass spectrum it was derived from
- The persisted records conform to the published ms-lims database schema (schema_7.6.png)

## Limitations

- The primary protein hit is a single representative chosen by the Mueller et al. (2006) algorithm; it does not by itself resolve the true protein origin of a shared peptide — all other progenitor proteins are only maintained alongside the identification in the database
- Building the parsing component requires the ms-lims 7.7.7 codebase with compomics-utilities as a Maven dependency in Java
- Database schema changes require applying the documented database updates when upgrading ms-lims versions

## Evidence

- [methods] a representative protein is selected as the primary protein hit according to the algorithm published in reference [7] (Mueller et al., 2006), while all other possible progenitor proteins for the peptide are maintained in the database structure: [evidence span withheld — anchor status: not_found]
