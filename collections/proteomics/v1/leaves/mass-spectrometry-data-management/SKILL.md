---
name: mass-spectrometry-data-management
description: Use when raw vendor-specific MS/MS peak lists must be converted to Mascot
  Generic File (MGF) format, persisted in a central proteomics database, and later
  interrogated via filtered peptide/protein lists and descriptive statistics.
when_to_use_negative:
- Input is already MGF or an already-populated ms_lims database where only the Project
  Analyzer step is needed
- Data are non-proteomic (e.g., non-MS 'omics) or require quantification workflows
  outside ms_lims' central-database scope
- A web-based or cloud LIMS is required; ms_lims is a lightweight local, portable
  client-database system
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ms-lims 7.7.7
  role: The LIMS distribution providing the ms-lims jar (client plus libraries), the
    Spectrum Storage import adaptor, and the Project Analyzer analysis application
  repo: http://genesis.ugent.be/uvpublicdata/ms-lims/ms-lims-7.7.7.zip
- name: compomics-utilities
  role: Library ms-lims builds upon; used for spectrum reading, MGF serialization,
    and the pluggable Project Analyzer analysis tools
  repo: http://code.google.com/p/compomics-utilities
- name: dbtoolkit
  role: Related CompOmics project for FASTA sequence database manipulation usable
    alongside ms_lims workflows
  repo: https://github.com/compomics/dbtoolkit
- name: Java / Maven
  role: Build and runtime environment for the ms_lims spectrum conversion component
    and client
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1002/pmic.200900409
    title: ms_lims, a simple yet powerful open source laboratory information management
      system for MS‐driven proteomics
  source_content_hash: sha256:f24cd115be38d7a00dc41b736529e5de37e918faf22b289cb84acbb09f89ed16
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java / Maven
  - compomics-utilities
  - dbtoolkit
  - ms-lims 7.7.7
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# mass-spectrometry-data-management

## Summary

Automates data management and processing in MS-driven proteomics using ms_lims, an open-source LIMS built on a central relational database that ingests heterogeneous vendor peak lists, converts them to MGF, and serves identifications and quantification results back to users.

## When to use

Use when raw vendor-specific MS/MS peak lists must be converted to Mascot Generic File (MGF) format, persisted in a central proteomics database, and later interrogated via filtered peptide/protein lists and descriptive statistics.

## When NOT to use

- Input is already MGF or an already-populated ms_lims database where only the Project Analyzer step is needed
- Data are non-proteomic (e.g., non-MS 'omics) or require quantification workflows outside ms_lims' central-database scope
- A web-based or cloud LIMS is required; ms_lims is a lightweight local, portable client-database system

## Inputs

- Vendor-specific MS/MS peak list files in heterogeneous formats
- Converted Mascot Generic File (MGF) spectra (e.g., spectra_converted.mgf)
- Populated central ms_lims relational database with spectra, identifications and quantification results

## Outputs

- MGF-formatted spectra persisted in the central ms_lims database
- Filtered peptide/protein lists with descriptive statistics (e.g., project_analysis_output.csv)
- Per-project summary counts computed from stored identifications

## How to apply

Start from the ms-lims 7.7.7 distribution (a zip containing the ms-lims jar plus all required libraries); launch it by running the jar, which opens the ms_lims client. For ingestion, load vendor-specific MS/MS peak list files into the Spectrum Storage application, which parses the heterogeneous vendor formats and converts each spectrum to Mascot Generic File (MGF) format before inserting the MGF records into the central ms_lims relational database; conversion relies on the compomics-utilities library that ms-lims builds upon. For downstream interrogation, connect the client to the populated central database, select a target proteomics project in the Project Analyzer, and run its pluggable tools (also built on compomics-utilities), which query the stored identifications, apply filtering criteria to peptide and protein hits, and compute descriptive statistics such as counts and per-project summary values. Export the filtered list and statistics (e.g., as project_analysis_output.csv) and verify that each stored spectrum remains queryable from the database and that MGF records conform to the Mascot Generic File format expected by downstream ms_lims processing.

## Related tools

- **ms-lims 7.7.7** (The LIMS distribution providing the ms-lims jar (client plus libraries), the Spectrum Storage import adaptor, and the Project Analyzer analysis application) — http://genesis.ugent.be/uvpublicdata/ms-lims/ms-lims-7.7.7.zip
- **compomics-utilities** (Library ms-lims builds upon; used for spectrum reading, MGF serialization, and the pluggable Project Analyzer analysis tools) — http://code.google.com/p/compomics-utilities
- **dbtoolkit** (Related CompOmics project for FASTA sequence database manipulation usable alongside ms_lims workflows) — https://github.com/compomics/dbtoolkit
- **Java / Maven** (Build and runtime environment for the ms_lims spectrum conversion component and client)

## Examples

```
java -jar ms-lims-7.7.7.jar
```

## Evaluation signals

- spectra_converted.mgf exists after the Spectrum Storage import run
- Each stored spectrum is queryable from the central ms_lims database
- MGF records conform to the Mascot Generic File format expected by downstream ms_lims processing
- project_analysis_output.csv exists and contains peptide/protein fields for the requested project
- The exported file presents the filtered identification list together with the descriptive statistics computed from the central database

## Limitations

- ms_lims is a client that must be connected to a populated central ms_lims database; without a populated database there is nothing to analyze
- The zip contains the ms-lims jar and all needed libraries; running it requires a working Java environment and the downloadable 7.7.7 distribution
- Project Analyzer analysis tools are pluggable and must be loaded from the database for the selected project; only the loaded tools' specific analysis tasks (e.g., filtered lists, descriptive statistics) are performed

## Evidence

- [abstract] ms_lims as open-source central-database LIMS: "We here present ms_lims (http://genesis.UGent.be/ms_lims), a freely available, open-source system based on a central database to automate data management and processing in MS-driven proteomics"
- [methods] download ms-lims 7.7.7: "Download ms-lims 7.7.7 [here](http://genesis.ugent.be/uvpublicdata/ms-lims/ms-lims-7.7.7.zip)."
- [methods] compomics-utilities dependency: "Ms-lims makes use of [compomics-utilities](http://code.google.com/p/compomics-utilities)."
- [readme] jar launch instructions: "The zip file contains the *ms-lims* jar file and all the libraries needed to run the tool. *Double clicking this resulting jar file will start ms-lims.*"
- [readme] build toolchain: "| IntelliJ | Netbeans | Java | Maven |"
