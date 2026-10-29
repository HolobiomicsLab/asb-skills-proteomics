---
name: spectral-peak-list-conversion
description: Use when heterogeneous vendor-specific MS/MS peak list files must be
  normalized to MGF and ingested into the central ms_lims database for MS-driven proteomics
  data management.
when_to_use_negative:
- Input is already MGF and only needs direct database loading, not vendor-format conversion
- The goal is manipulating FASTA protein sequence databases — that is dbtoolkit's
  scope (merge, shuffle, truncate), not spectrum storage
- The analysis needs peak identification, quantification, or Peptizer-based validation
  rather than peak-list format normalization and storage
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: ms-lims 7.7.7
  role: Provides the Spectrum Storage application used to convert vendor peak lists
    to MGF and insert them into the central ms_lims relational database
  repo: http://genesis.ugent.be/uvpublicdata/ms-lims/ms-lims-7.7.7.zip
- name: compomics-utilities
  role: Library ms-lims builds upon; supplies spectrum reading and MGF serialization
    for the conversion
  repo: http://code.google.com/p/compomics-utilities
- name: Java
  role: Language in which the ms_lims spectrum conversion component and ms-lims jar
    are built and run
  repo: https://github.com/compomics/ms-lims
- name: Maven
  role: Build tool for the ms_lims spectrum conversion component
  repo: https://github.com/compomics/ms-lims
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.200900409
    title: ms_lims, a simple yet powerful open source laboratory information management
      system for MS‐driven proteomics
  source_content_hash: sha256:2b0905cb209c9cb9e1bb79d0d074105282406f9c017c8cc0d1e0f4e5eb1cf0d7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - Maven
  - compomics-utilities
  - ms-lims 7.7.7
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# spectral-peak-list-conversion

## Summary

Convert heterogeneous, vendor-specific MS/MS peak lists into the Mascot Generic File (MGF) format and persist the converted spectra (gzipped) in the central ms_lims database via the Spectrum Storage application, so that downstream ms_lims processing has a uniform spectral input.

## When to use

Use when heterogeneous vendor-specific MS/MS peak list files must be normalized to MGF and ingested into the central ms_lims database for MS-driven proteomics data management.

## When NOT to use

- Input is already MGF and only needs direct database loading, not vendor-format conversion
- The goal is manipulating FASTA protein sequence databases — that is dbtoolkit's scope (merge, shuffle, truncate), not spectrum storage
- The analysis needs peak identification, quantification, or Peptizer-based validation rather than peak-list format normalization and storage

## Inputs

- Vendor-specific MS/MS peak list files (heterogeneous formats produced by external applications)
- ms-lims 7.7.7 distribution (zip containing the ms-lims jar and libraries)
- Central ms_lims relational database schema (schema_7.6)

## Outputs

- spectra_converted.mgf — spectra in Mascot Generic File (MGF) format
- Gzipped MGF spectra persisted in the central ms_lims database
- Database records for each spectrum queryable from ms_lims

## How to apply

Load the vendor-specific MS/MS peak list files to be ingested and parse each vendor peak list format, converting every spectrum to Mascot Generic File (MGF) format with the ms_lims spectrum conversion component, which is built with Java and Maven. Leverage the compomics-utilities library — which ms-lims builds upon — for spectrum reading and MGF serialization rather than hand-writing format parsers. Insert the MGF-formatted spectra into the central ms_lims relational database via the Spectrum Storage application from the ms-lims 7.7.7 distribution, which stores the MGF content gzipped. The rationale is that a central database of uniformly formatted MGF spectra lets ms_lims automate data management and processing for high-throughput MS-driven proteomics. Validate by checking that spectra_converted.mgf exists and that each stored spectrum is queryable from the central database.

## Related tools

- **ms-lims 7.7.7** (Provides the Spectrum Storage application used to convert vendor peak lists to MGF and insert them into the central ms_lims relational database) — http://genesis.ugent.be/uvpublicdata/ms-lims/ms-lims-7.7.7.zip
- **compomics-utilities** (Library ms-lims builds upon; supplies spectrum reading and MGF serialization for the conversion) — http://code.google.com/p/compomics-utilities
- **Java** (Language in which the ms_lims spectrum conversion component and ms-lims jar are built and run) — https://github.com/compomics/ms-lims
- **Maven** (Build tool for the ms_lims spectrum conversion component) — https://github.com/compomics/ms-lims

## Examples

```
java -jar ms-lims-7.7.7.jar
```

## Evaluation signals

- spectra_converted.mgf exists after the conversion run
- Each stored spectrum is queryable from the central ms_lims database
- MGF records conform to the Mascot Generic File format expected by downstream ms_lims processing
- Converted spectra are persisted gzipped in the central database
- Every loaded vendor peak list file is represented by at least one converted spectrum record (no silent drops)

## Limitations

- Conversion is bounded by the vendor peak list formats readable through compomics-utilities; unsupported vendor formats will not ingest
- ms-lims is distributed as a jar launched by double-clicking (GUI-oriented), so bulk scripted ingestion may require building the conversion component from source with Maven
- Storage targets the ms_lims relational database with the schema_7.6 schema; database updates must be applied per the project wiki before use

## Evidence
