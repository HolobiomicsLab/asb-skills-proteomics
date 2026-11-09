---
name: mascot-search-output-parsing
description: Use when ingesting Mascot Server output (peptide identifications, iTRAQ
  reporter-ion quantitation, or Mascot Distiller quantitation) into a peptide-centric
  relational store linked to source mass spectra.
when_to_use_negative:
- The quantitation source is not Mascot-based — Quantitation Storage currently supports
  only Mascot Server (iTRAQ) and Mascot Distiller-based quantification.
- The input is raw, unsearched mass spectra — spectra require processing and identification
  first; this skill ingests search-engine output, not raw instrument data.
- The goal is protein-level inference across peptides — ms_lims storage is peptide-centric,
  selecting one primary hit per peptide while retaining all progenitor proteins instead.
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ms-lims
  role: Provides the Identification Storage and Quantitation Storage importers that
    parse Mascot Server output and persist spectra, identifications, progenitor proteins,
    primary hits and quantitation into the central relational database
  repo: https://github.com/compomics/ms-lims
- name: compomics-utilities
  role: Shared parsing and IO library reused to parse Mascot Server iTRAQ result files
    and Mascot Distiller quantitation output
  repo: http://code.google.com/p/compomics-utilities
- name: Maven
  role: Builds the extended ms-lims Java codebase into the ms-lims jar that runs the
    importers
  repo: http://maven.apache.org/
- name: Java
  role: Implementation language for the Mascot-output parsing component in the ms-lims
    7.7.7 codebase
  repo: http://java.com/en/
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1002/pmic.200900409
    title: ms_lims, a simple yet powerful open source laboratory information management
      system for MS‐driven proteomics
  source_content_hash: sha256:79f1ee783286703b3b69eaac391e794adc5f31f378bb1374417b4085839642a9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/compomics/ms-lims
  tools:
  - IntelliJ IDEA
  - Java
  - Maven
  - Netbeans
  - compomics-utilities
  - ms-lims
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mascot-search-output-parsing

## Summary

The ms_lims approach to ingesting Mascot Server search-engine output: each peptide identification is stored in a central relational database linked to its source mass spectrum, with the full progenitor-protein set retained and one primary protein hit selected via the Mueller et al. (2006) algorithm. A companion Quantitation Storage importer auto-links Mascot (iTRAQ) reporter-ion and Mascot Distiller quantitation to the stored identifications.

## When to use

Use when ingesting Mascot Server output (peptide identifications, iTRAQ reporter-ion quantitation, or Mascot Distiller quantitation) into a peptide-centric relational store linked to source mass spectra.

## When NOT to use

- The quantitation source is not Mascot-based — Quantitation Storage currently supports only Mascot Server (iTRAQ) and Mascot Distiller-based quantification.
- The input is raw, unsearched mass spectra — spectra require processing and identification first; this skill ingests search-engine output, not raw instrument data.
- The goal is protein-level inference across peptides — ms_lims storage is peptide-centric, selecting one primary hit per peptide while retaining all progenitor proteins instead.

## Inputs

- Mascot Server search-engine output files (peptide identifications)
- Mascot Server iTRAQ result files (reporter-ion-based peptide quantitation values)
- Mascot Distiller quantitation output
- ms-lims 7.7.7 source codebase with Maven build (Java)
- Published database schema diagram (schema_7.6.png) and wiki DatabaseUpdates notes defining the central ms_lims relational tables

## Outputs

- Peptide identifications stored in the central ms_lims relational database, each linked to the identifier of its source mass spectrum
- Complete progenitor protein list persisted per identified peptide
- Selected primary protein hit per peptide (Mueller et al. 2006 algorithm)
- Auto-linked spectra–identification–quantitation records in the central database
- identifications_with_primary_hits.csv (identifications with spectrum links, all progenitor proteins, and the primary protein hit per peptide)

## How to apply

Set up the Java/Maven build (IntelliJ IDEA or Netbeans) with the ms-lims 7.7.7 codebase and compomics-utilities as dependencies, and inspect the relational structure via the downloaded schema diagram (schema_7.6.png) and the wiki DatabaseUpdates notes. Parse each Mascot Server search-engine output file to extract every peptide identification together with the identifier of the mass spectrum it was derived from. For each stored peptide identification, resolve the complete set of progenitor proteins that contain the peptide sequence, persist all of them alongside the identification, and apply the primary-hit selection algorithm of reference [7] (Mueller et al., 2006) to choose one representative protein per peptide. For quantitation, parse Mascot Server iTRAQ result files for reporter-ion peptide values and Mascot Distiller quantitation output, reusing the shared parsing and IO classes from compomics-utilities. Map each imported quantitation record onto the matching existing identification record via the spectral and peptide keys in the central ms_lims database, so linkage works even when files were generated at different time points by different users. Persist identifications, spectrum links, progenitor lists and primary hits per the published schema, then export identifications_with_primary_hits.csv for validation.

## Related tools

- **ms-lims** (Provides the Identification Storage and Quantitation Storage importers that parse Mascot Server output and persist spectra, identifications, progenitor proteins, primary hits and quantitation into the central relational database) — https://github.com/compomics/ms-lims
- **compomics-utilities** (Shared parsing and IO library reused to parse Mascot Server iTRAQ result files and Mascot Distiller quantitation output) — http://code.google.com/p/compomics-utilities
- **Maven** (Builds the extended ms-lims Java codebase into the ms-lims jar that runs the importers) — http://maven.apache.org/
- **Java** (Implementation language for the Mascot-output parsing component in the ms-lims 7.7.7 codebase) — http://java.com/en/

## Examples

```
java -jar ms-lims-7.7.7.jar
```

## Evaluation signals

- identifications_with_primary_hits.csv exists and every row contains a non-empty primary protein hit and the complete progenitor protein set for its peptide
- Every imported quantitation record is auto-linked to an existing identification in the central database — no orphan quantitation entries on the test database
- Spectra–identification–quantitation linkage verified end-to-end, including records generated at different time points by different users
- The persisted primary hit for each peptide equals the representative protein chosen by the Mueller et al. (2006) algorithm from that peptide's progenitor set, with no progenitor proteins lost
- Stored rows conform to the published relational structure defined by the schema_7.6.png diagram

## Limitations

- Quantitation Storage currently supports only Mascot Server (iTRAQ) and Mascot Distiller-based quantification; other quantification sources are out of scope.
- Primary-hit selection is peptide-centric (one representative protein per peptide via Mueller et al. 2006); protein-level inference is not performed — all progenitor proteins are retained instead.
- Linkage depends on matching spectral and peptide keys against existing identification records in the central database, so quantitation imports assume identifications are already stored.

## Evidence

- [abstract] We here present ms_lims (http://genesis.UGent.be/ms_lims), a freely available, open-source system based on a central database to automate data management and processing in MS-driven proteomics: "We here present ms_lims (http://genesis.UGent.be/ms_lims), a freely available, open-source system based on a central database to automate data management and processing in MS-driven proteomics"
- [methods] Ms-lims makes use of [compomics-utilities](http://code.google.com/p/compomics-utilities).: "Ms-lims makes use of [compomics-utilities](http://code.google.com/p/compomics-utilities)."
- [readme] Download the database schema [here](http://genesis.ugent.be/uvpublicdata/ms-lims/schema_7.6.png).: "Download the database schema [here](http://genesis.ugent.be/uvpublicdata/ms-lims/schema_7.6.png)."
- [readme] The zip file contains the *ms-lims* jar file and all the libraries needed to run the tool.: "The zip file contains the *ms-lims* jar file and all the libraries needed to run the tool."
- [other] a representative protein is selected as the primary protein hit according to the algorithm published in reference [7] (Mueller et al., 2006): [evidence span withheld — anchor status: not_found]
- [other] The separate storage applications automatically link spectra, identifications and quantitation together in the central database, even when generated at different time points by different users.: [evidence span withheld — anchor status: not_found]
- [other] ms_lims stores this third input data type through the Quantitation Storage application, which currently supports Mascot Server (iTRAQ) and Mascot Distiller-based quantification: [evidence span withheld — anchor status: not_found]
