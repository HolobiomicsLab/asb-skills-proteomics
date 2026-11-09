---
name: mgf-format-parsing
description: Use when input consists of raw MS/MS peak lists in mixed vendor-specific
  formats that must be normalized to MGF before central database storage or downstream
  ms_lims processing.
when_to_use_negative:
- The peak lists are already in MGF format and only need direct database insertion
  without vendor-format parsing
- The data are non-spectral inputs (e.g. protein sequence FASTA databases handled
  by dbtoolkit) rather than MS/MS peak lists
- The analysis is downstream processing/identification/quantification of spectra already
  stored in ms_lims, not import/format conversion
edam_operation: http://edamontology.org/operation_0236
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: ms_lims
  role: Central open-source LIMS whose Spectrum Storage application performs the MGF
    conversion and database persistence of spectra
- name: ms-lims 7.7.7
  role: The specific distribution containing the Spectrum Storage import adaptor used
    to ingest and store MGF spectra
  repo: http://genesis.ugent.be/uvpublicdata/ms-lims/ms-lims-7.7.7.zip
- name: compomics-utilities
  role: Library ms-lims builds upon for spectrum reading and MGF serialization during
    conversion
  repo: http://code.google.com/p/compomics-utilities
- name: Java
  role: Language in which the ms_lims spectrum conversion component is implemented
    and rebuilt
- name: Maven
  role: Build system used to compile the spectrum conversion component when adapting
    to new vendor formats
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.200900409
    title: ms_lims, a simple yet powerful open source laboratory information management
      system for MS‐driven proteomics
  source_content_hash: sha256:d8849ec6e69e9ed2271915178984ccb05bf09f810e388455b33d9a271ad3d1a8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - Maven
  - compomics-utilities
  - ms-lims 7.7.7
  - ms_lims
  edam_operation: http://edamontology.org/operation_0236
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# mgf-format-parsing

## Summary

Parse heterogeneous, vendor-specific MS/MS peak list formats and convert each spectrum into Mascot Generic File (MGF) format so the spectra can be ingested, queried, and stored in the central ms_lims relational database.

## When to use

Use when input consists of raw MS/MS peak lists in mixed vendor-specific formats that must be normalized to MGF before central database storage or downstream ms_lims processing.

## When NOT to use

- The peak lists are already in MGF format and only need direct database insertion without vendor-format parsing
- The data are non-spectral inputs (e.g. protein sequence FASTA databases handled by dbtoolkit) rather than MS/MS peak lists
- The analysis is downstream processing/identification/quantification of spectra already stored in ms_lims, not import/format conversion

## Inputs

- Vendor-specific MS/MS peak list files (heterogeneous input formats)
- ms-lims 7.7.7 distribution (jar plus bundled libraries)
- compomics-utilities spectrum reading/serialization library

## Outputs

- Spectra converted to Mascot Generic File (MGF) format (e.g. spectra_converted.mgf)
- MGF spectra persisted as gzipped records in the central ms_lims relational database, queryable by spectrum

## How to apply

Load the vendor-specific MS/MS peak list files and use the ms_lims Spectrum Storage import adaptor (from the ms-lims 7.7.7 distribution) to parse each format and convert each spectrum to Mascot Generic File (MGF) format, relying on compomics-utilities for spectrum reading and MGF serialization. The conversion component is built with Java and Maven, so extend or rebuild it there if a new vendor format must be supported. Insert the resulting MGF-formatted spectra into the central ms_lims relational database. Decision point: convert-and-persist on import rather than storing raw vendor files, so that all spectra in the database conform to a single common format downstream.

## Related tools

- **ms_lims** (Central open-source LIMS whose Spectrum Storage application performs the MGF conversion and database persistence of spectra)
- **ms-lims 7.7.7** (The specific distribution containing the Spectrum Storage import adaptor used to ingest and store MGF spectra) — http://genesis.ugent.be/uvpublicdata/ms-lims/ms-lims-7.7.7.zip
- **compomics-utilities** (Library ms-lims builds upon for spectrum reading and MGF serialization during conversion) — http://code.google.com/p/compomics-utilities
- **Java** (Language in which the ms_lims spectrum conversion component is implemented and rebuilt)
- **Maven** (Build system used to compile the spectrum conversion component when adapting to new vendor formats)

## Evaluation signals

- spectra_converted.mgf exists after the import run
- Each converted spectrum is queryable from the central ms_lims relational database after insertion
- MGF records conform to the Mascot Generic File format expected by downstream ms_lims processing (e.g. proper BEGIN IONS/END IONS blocks with peak lists)
- Every input vendor-format peak list yielded a corresponding converted spectrum (no spectra silently dropped during format parsing)

## Limitations

- Conversion is implemented in the Java/Maven ms_lims spectrum conversion component, so unsupported vendor formats require code changes and a rebuild rather than configuration alone
- ms_lims is a lightweight LIMS for MS-driven proteomics spectra management; it does not itself perform identification or quantification of the imported spectra

## Evidence

- [other] vendor formats converted to MGF upon import: [evidence span withheld — anchor status: not_found]
