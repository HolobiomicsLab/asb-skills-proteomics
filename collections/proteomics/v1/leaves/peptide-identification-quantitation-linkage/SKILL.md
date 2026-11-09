---
name: peptide-identification-quantitation-linkage
description: Use when importing reporter-ion quantitation (Mascot iTRAQ result files
  or Mascot Distiller output) into ms_lims and linking each quantitation record to
  matching peptide identifications in the central database.
when_to_use_negative:
- Input quantitation does not come from Mascot Server (iTRAQ) or Mascot Distiller,
  the only sources the Quantitation Storage application currently supports
- The task is raw spectra storage or identification storage rather than linking quantitation
  to existing identifications
- No populated ms_lims database exists yet to map quantitation records onto via spectral
  and peptide keys
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ms_lims
  role: LIMS providing the central database into which the Quantitation Storage importer
    stores quantitation and links it to spectra and identifications
  repo: https://github.com/compomics/ms-lims
- name: ms-lims 7.7.7
  role: Distribution whose jar and libraries are built with Maven and run against
    a populated test database to verify import and linkage
  repo: https://github.com/compomics/ms-lims
- name: compomics-utilities
  role: Shared parsing and IO classes reused to parse Mascot iTRAQ and Mascot Distiller
    quantitation output
  repo: http://code.google.com/p/compomics-utilities
- name: Maven
  role: Build system used to compile the extended codebase into the ms-lims jar
- name: IntelliJ IDEA
  role: IDE used to set up the Java build environment for implementing the quantitation
    import component
- name: Netbeans
  role: Alternative IDE for the Java/Maven build environment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.200900409
    title: ms_lims, a simple yet powerful open source laboratory information management
      system for MS‐driven proteomics
  source_content_hash: sha256:4e93e291a953bdfab3fa2cbe42ba75931545d700f930335114b191fec6c71de6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/compomics/ms-lims
  tools:
  - IntelliJ IDEA
  - Maven
  - Netbeans
  - compomics-utilities
  - ms-lims 7.7.7
  - ms_lims
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# peptide-identification-quantitation-linkage

## Summary

A method for storing and auto-linking quantitative peptide data from differential proteomics (Mascot iTRAQ and Mascot Distiller quantification) to existing peptide identifications and mass spectra in the ms_lims central database, so that spectra, identifications and quantitation remain associated even when generated at different times by different users.

## When to use

Use when importing reporter-ion quantitation (Mascot iTRAQ result files or Mascot Distiller output) into ms_lims and linking each quantitation record to matching peptide identifications in the central database.

## When NOT to use

- Input quantitation does not come from Mascot Server (iTRAQ) or Mascot Distiller, the only sources the Quantitation Storage application currently supports
- The task is raw spectra storage or identification storage rather than linking quantitation to existing identifications
- No populated ms_lims database exists yet to map quantitation records onto via spectral and peptide keys

## Inputs

- Mascot Server (iTRAQ) result files with reporter-ion-based quantitation
- Mascot Distiller quantitation output files
- Populated ms_lims central database containing existing spectra and peptide identifications

## Outputs

- Quantitation records stored in the ms_lims central database
- Spectra–identification–quantitation linkage entries with no orphan quantitation records

## How to apply

Set up the Java/Maven build environment (IntelliJ IDEA or Netbeans) from the ms-lims 7.7.7 distribution and source, and inspect the relational structure using the schema_7.6.png database schema diagram and the wiki DatabaseUpdates notes to identify the identification and spectral tables. Implement a Quantitation Storage import component that parses Mascot Server iTRAQ result files to extract reporter-ion-based peptide quantitation values, then extend it to parse Mascot Distiller quantitation output, reusing shared parsing and IO classes from compomics-utilities. Map each imported quantitation record onto the matching existing identification record via the spectral and peptide keys in the central ms_lims database; the separate storage applications link spectra, identifications and quantitation automatically regardless of when or by whom the files were produced. Build with Maven into the ms-lims jar and run against a populated test database to confirm import and linkage.

## Related tools

- **ms_lims** (LIMS providing the central database into which the Quantitation Storage importer stores quantitation and links it to spectra and identifications) — https://github.com/compomics/ms-lims
- **ms-lims 7.7.7** (Distribution whose jar and libraries are built with Maven and run against a populated test database to verify import and linkage) — https://github.com/compomics/ms-lims
- **compomics-utilities** (Shared parsing and IO classes reused to parse Mascot iTRAQ and Mascot Distiller quantitation output) — http://code.google.com/p/compomics-utilities
- **Maven** (Build system used to compile the extended codebase into the ms-lims jar)
- **IntelliJ IDEA** (IDE used to set up the Java build environment for implementing the quantitation import component)
- **Netbeans** (Alternative IDE for the Java/Maven build environment)

## Evaluation signals

- Every imported quantitation record is auto-linked to an existing identification in the central ms_lims database
- Spectra–identification–quantitation linkage verified on the test database
- No orphan quantitation entries exist after import
- Extended codebase builds cleanly with Maven into the ms-lims jar
- Linkage works for files produced at different time points by different users

## Limitations

- Quantitation Storage currently supports only Mascot Server (iTRAQ) and Mascot Distiller-based quantification; other quantification sources require extending the importer
- Linkage depends on matching spectral and peptide keys already present in the central database; quantitation for absent identifications would be orphaned

## Evidence

- [other] Quantitation Storage supports Mascot iTRAQ and Distiller quantification with automatic linkage: [evidence span withheld — anchor status: not_found]
- [other] Automatic linkage across time points and users: [evidence span withheld — anchor status: not_found]
