---
name: peptide-spectrum-identification-storage
description: Use when search-engine output (e.g. Mascot Server results) must be parsed
  into a peptide-centric LIMS database with spectrum links and a single representative
  primary protein per peptide.
when_to_use_negative:
- The analysis is protein-centric and requires inferred protein groups rather than
  peptide-centric storage with all progenitors retained
- The search engine output is not Mascot Server format and no parser for it exists
  in the ms-lims/compomics-utilities stack
- Only raw spectra processing or quantification is needed, without identification
  storage
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ms-lims 7.7.7
  role: provides the codebase, central database schema (schema_7.6.png), and jar release
    used to build the Mascot parser and identification storage
  repo: https://github.com/compomics/ms-lims
- name: compomics-utilities
  role: library dependency used by ms-lims for parsing search-engine output and proteomics
    data handling
  repo: http://code.google.com/p/compomics-utilities
- name: Java
  role: implementation language for the parsing and storage component
- name: Maven
  role: build tool for the Java component with ms-lims and compomics-utilities dependencies
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.200900409
    title: ms_lims, a simple yet powerful open source laboratory information management
      system for MS‐driven proteomics
  source_content_hash: sha256:cd79c7bcce3acffd2f405e3a62a17a10f460d119eb5bb824d13f82bf1f1bfabc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/compomics/ms-lims
  tools:
  - Java
  - Maven
  - compomics-utilities
  - ms-lims 7.7.7
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# peptide-spectrum-identification-storage

## Summary

Ingest Mascot Server search-engine output into a central relational store (ms_lims), linking each peptide identification to its source mass spectrum, retaining all progenitor proteins per peptide, and selecting one representative primary protein hit via the Mueller et al. (2006) algorithm.

## When to use

Use when search-engine output (e.g. Mascot Server results) must be parsed into a peptide-centric LIMS database with spectrum links and a single representative primary protein per peptide.

## When NOT to use

- The analysis is protein-centric and requires inferred protein groups rather than peptide-centric storage with all progenitors retained
- The search engine output is not Mascot Server format and no parser for it exists in the ms-lims/compomics-utilities stack
- Only raw spectra processing or quantification is needed, without identification storage

## Inputs

- Mascot Server search-engine output files
- mass spectrum identifiers associated with each peptide identification
- protein sequence database for resolving progenitor proteins

## Outputs

- central relational store of peptide identifications linked to source spectra
- per-peptide list of all progenitor proteins
- selected primary protein hit per peptide
- exported table (e.g. identifications_with_primary_hits.csv) with spectrum links, progenitor sets, and primary hits

## How to apply

Set up a Java component using the ms-lims 7.7.7 codebase with compomics-utilities as a dependency, built with Maven. Parse each Mascot Server output file to extract every peptide identification together with the identifier of the mass spectrum it was derived from. For each peptide, resolve the complete set of progenitor proteins containing the peptide sequence, then apply the primary-hit selection algorithm of Mueller et al. (2006) to choose one representative protein while keeping all other progenitors in the database. Persist identifications, spectrum links, progenitor lists, and primary hits according to the published ms-lims relational schema (schema_7.6.png). The peptide-centric design means the database stores all possible protein origins per peptide, avoiding loss of shared-peptide provenance at storage time.

## Related tools

- **ms-lims 7.7.7** (provides the codebase, central database schema (schema_7.6.png), and jar release used to build the Mascot parser and identification storage) — https://github.com/compomics/ms-lims
- **compomics-utilities** (library dependency used by ms-lims for parsing search-engine output and proteomics data handling) — http://code.google.com/p/compomics-utilities
- **Java** (implementation language for the parsing and storage component)
- **Maven** (build tool for the Java component with ms-lims and compomics-utilities dependencies)

## Evaluation signals

- The exported identifications table (e.g. identifications_with_primary_hits.csv) exists and contains a non-empty primary protein hit for every row
- Every row's progenitor protein set is complete for its peptide sequence (all proteins containing the peptide are present)
- Each stored identification is linked to the identifier of the mass spectrum it was derived from
- Stored records conform to the published ms-lims database schema (schema_7.6.png)

## Limitations

- The primary-hit selection depends on the Mueller et al. (2006) algorithm; peptides shared across multiple proteins yield a representative hit while full provenance is only preserved via the stored progenitor lists
- ms-lims is built around Mascot Server output and a specific relational schema; other search-engine formats require adaptation
- The database schema version is tied to the release (7.6 schema shipped with 7.7.7); database updates are handled via the project wiki

## Evidence

- [methods] peptide-centric storage mechanism with primary hit selection: [evidence span withheld — anchor status: not_found]
- [methods] all progenitor proteins retained: [evidence span withheld — anchor status: not_found]
