---
name: proteomics-spectrum-database-storage
description: Use when raw vendor-specific MS/MS peak lists (e.g., Waters, ABI, Bruker
  formats) must be normalized to MGF and stored centrally so downstream ms_lims identification
  and quantification steps can query them.
when_to_use_negative:
- The spectra are already stored in MGF format inside the central ms_lims database
  (re-import would duplicate records).
- The task is manipulating FASTA protein sequence databases (merge, shuffle, truncate)
  — that is dbtoolkit's scope, not Spectrum Storage.
- The task is downstream peptide/protein identification or quantification rather than
  getting spectra into the central store.
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: ms_lims
  role: Open-source LIMS whose central database stores the MGF-converted MS/MS spectra;
    Spectrum Storage is its import application
  repo: https://github.com/compomics/ms-lims
- name: ms-lims 7.7.7
  role: Distribution zip supplying the runnable ms-lims jar and libraries used to
    run the Spectrum Storage import adaptor
  repo: https://github.com/compomics/ms-lims
- name: compomics-utilities
  role: Library ms-lims builds upon for spectrum reading and MGF serialization during
    format conversion
- name: Java
  role: Language of the spectrum conversion component implementing the vendor-format-to-MGF
    parsing
- name: Maven
  role: Build system for compiling the ms_lims spectrum conversion component
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.200900409
    title: ms_lims, a simple yet powerful open source laboratory information management
      system for MS‐driven proteomics
  source_content_hash: sha256:b868e62ece2cf744a6924f6c34db82dfbac532a43846fe531610d170dd77b4c4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/compomics/ms-lims
  tools:
  - Java
  - Maven
  - compomics-utilities
  - ms-lims 7.7.7
  - ms_lims
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# proteomics-spectrum-database-storage

## Summary

Reconstructs the ms_lims Spectrum Storage import adaptor, which converts heterogeneous vendor-specific MS/MS peak lists into Mascot Generic File (MGF) format and persists the converted spectra as gzipped records in the central ms_lims relational database.

## When to use

Use when raw vendor-specific MS/MS peak lists (e.g., Waters, ABI, Bruker formats) must be normalized to MGF and stored centrally so downstream ms_lims identification and quantification steps can query them.

## When NOT to use

- The spectra are already stored in MGF format inside the central ms_lims database (re-import would duplicate records).
- The task is manipulating FASTA protein sequence databases (merge, shuffle, truncate) — that is dbtoolkit's scope, not Spectrum Storage.
- The task is downstream peptide/protein identification or quantification rather than getting spectra into the central store.

## Inputs

- Vendor-specific MS/MS peak list files (heterogeneous formats from external MS instruments)
- ms-lims 7.7.7 distribution zip containing the ms-lims jar and required libraries

## Outputs

- spectra_converted.mgf (spectra in Mascot Generic File format)
- Gzipped MGF spectrum records persisted in the central ms_lims relational database, queryable per spectrum

## How to apply

Load the vendor-specific MS/MS peak list files targeted for ingestion, then parse each vendor format and convert every spectrum to Mascot Generic File (MGF) format using the ms_lims spectrum conversion component, which is built with Java and Maven and leans on compomics-utilities for spectrum reading and MGF serialization. Insert the MGF-formatted spectra into the central ms_lims relational database via the Spectrum Storage application shipped in the ms-lims 7.7.7 distribution zip, persisting them gzipped to keep the central store compact. The rationale is that the central database is the pivot of ms_lims: converting all vendor formats to the commonly used MGF format at import time lets identification, quantification, and interpretation tools consume a single uniform spectrum format. Judge success by checking that spectra_converted.mgf exists, that each stored spectrum is queryable from the central ms_lims database, and that the MGF records conform to the Mascot Generic File format expected by downstream ms_lims processing.

## Related tools

- **ms_lims** (Open-source LIMS whose central database stores the MGF-converted MS/MS spectra; Spectrum Storage is its import application) — https://github.com/compomics/ms-lims
- **ms-lims 7.7.7** (Distribution zip supplying the runnable ms-lims jar and libraries used to run the Spectrum Storage import adaptor) — https://github.com/compomics/ms-lims
- **compomics-utilities** (Library ms-lims builds upon for spectrum reading and MGF serialization during format conversion)
- **Java** (Language of the spectrum conversion component implementing the vendor-format-to-MGF parsing)
- **Maven** (Build system for compiling the ms_lims spectrum conversion component)

## Examples

```
java -jar ms-lims-7.7.7.jar
```

## Evaluation signals

- spectra_converted.mgf file exists after the conversion run
- Each imported spectrum is queryable from the central ms_lims database
- MGF records conform to the Mascot Generic File format expected by downstream ms_lims processing
- Stored spectra are gzipped rather than plain text in the central store
- Vendor-specific peak lists of different origins all round-trip through conversion without loss of spectra

## Limitations

- Conversion covers the vendor peak-list formats supported by the ms_lims spectrum conversion component / compomics-utilities; unsupported vendor formats must be pre-converted.
- ms_lims is a lightweight, portable LIMS: it is intended for data management of MS-driven proteomics workflows, not for general laboratory record keeping outside spectra.
- The ms-lims distribution requires its bundled jar and libraries (double-clicking the jar starts ms-lims); missing libraries or an outdated database schema (see schema_7.6.png and DatabaseUpdates wiki) can block import.

## Evidence
