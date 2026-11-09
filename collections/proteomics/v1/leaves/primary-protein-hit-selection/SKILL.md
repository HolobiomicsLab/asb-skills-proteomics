---
name: primary-protein-hit-selection
description: Use when storing or reporting peptide identifications from Mascot Server
  (or similar) output where each peptide maps to multiple progenitor proteins and
  a single representative protein must be reported per peptide.
when_to_use_negative:
- Input is protein-centric rather than peptide-centric (e.g. protein inference reports
  already providing a single protein group per identification)
- The Mascot output already designates a single top protein hit and no peptide-to-multiple-proteins
  ambiguity exists
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ms-lims 7.7.7
  role: Provides the peptide-centric storage schema and importer codebase in which
    Mascot Server output is parsed, identifications are stored with spectrum links,
    and the primary protein hit is selected per peptide
  repo: https://github.com/compomics/ms-lims
- name: compomics-utilities
  role: Library dependency used when building the parsing component that reads Mascot
    Server search-engine output files
  repo: http://code.google.com/p/compomics-utilities
- name: dbtoolkit
  role: Related CompOmics project for protein/fasta sequence database manipulation,
    listed alongside ms-lims
  repo: https://github.com/compomics/dbtoolkit
- name: Java
  role: Language in which the Mascot parsing/primary-hit selection component is implemented
- name: Maven
  role: Build system for the Java parsing component with ms-lims and compomics-utilities
    as dependencies
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.200900409
    title: ms_lims, a simple yet powerful open source laboratory information management
      system for MS‐driven proteomics
  source_content_hash: sha256:31dabafd5952d32bc906c7060d9c385804453d565746813acb24b2ef9482ecdf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/compomics/ms-lims
  tools:
  - Java
  - Maven
  - compomics-utilities
  - dbtoolkit
  - ms-lims 7.7.7
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# primary-protein-hit-selection

## Summary

Selects one representative (primary) protein hit per identified peptide from its full set of progenitor proteins, as implemented in ms_lims' peptide-centric storage of Mascot Server search results, while retaining all other progenitor proteins in the database.

## When to use

Use when storing or reporting peptide identifications from Mascot Server (or similar) output where each peptide maps to multiple progenitor proteins and a single representative protein must be reported per peptide.

## When NOT to use

- Input is protein-centric rather than peptide-centric (e.g. protein inference reports already providing a single protein group per identification)
- The Mascot output already designates a single top protein hit and no peptide-to-multiple-proteins ambiguity exists

## Inputs

- Mascot Server search-engine output files (one per search run)
- Peptide identifications with the identifier of the mass spectrum each was derived from
- Protein sequence database used to resolve progenitor proteins per peptide

## Outputs

- Stored peptide identifications linked to their source spectra
- Complete progenitor protein list per identified peptide
- One primary protein hit per peptide (selected via the Mueller et al. 2006 algorithm)
- Exported table, e.g. identifications_with_primary_hits.csv, with spectrum links, all progenitor proteins, and the primary protein hit per peptide

## How to apply

Parse each Mascot Server search-engine output file to extract every peptide identification together with the identifier of the mass spectrum it was derived from, e.g. within a Java/Maven project using ms-lims 7.7.7 and compomics-utilities. For each stored peptide identification, resolve the complete set of progenitor proteins that contain the peptide sequence. Apply the primary-hit selection algorithm of Mueller et al. (2006) to choose one representative protein per peptide from its progenitor set. Persist the identifications, spectrum links, all progenitor protein lists, and the selected primary hits into the central relational store following the ms-lims database schema (schema_7.6.png), then export the full table (e.g. identifications_with_primary_hits.csv). The rationale is peptide-centric storage: a single representative protein makes downstream reporting unambiguous while the complete progenitor set is preserved in the database structure.

## Related tools

- **ms-lims 7.7.7** (Provides the peptide-centric storage schema and importer codebase in which Mascot Server output is parsed, identifications are stored with spectrum links, and the primary protein hit is selected per peptide) — https://github.com/compomics/ms-lims
- **compomics-utilities** (Library dependency used when building the parsing component that reads Mascot Server search-engine output files) — http://code.google.com/p/compomics-utilities
- **dbtoolkit** (Related CompOmics project for protein/fasta sequence database manipulation, listed alongside ms-lims) — https://github.com/compomics/dbtoolkit
- **Java** (Language in which the Mascot parsing/primary-hit selection component is implemented)
- **Maven** (Build system for the Java parsing component with ms-lims and compomics-utilities as dependencies)

## Evaluation signals

- The output file (e.g. identifications_with_primary_hits.csv) exists and contains one row per stored peptide identification
- Every row contains a non-empty primary protein hit
- Every row contains the complete progenitor protein set for its peptide, not just the primary hit
- Each stored identification is linked to the identifier of the mass spectrum it was derived from
- The persisted records conform to the published ms-lims database schema (schema_7.6.png)

## Limitations

- The primary-hit choice depends entirely on the Mueller et al. (2006) algorithm; peptides mapping to multiple proteins may receive a representative that is not the most biologically relevant
- Progenitor protein resolution is only as complete as the protein sequence database used at storage time
- ms-lims requires its central relational database and corresponding schema version (e.g. schema_7.6.png); database updates between releases must be applied per the project wiki

## Evidence

- [results] a representative protein is selected as the primary protein hit according to the algorithm published in reference [7] (Mueller et al., 2006), while all other possible progenitor proteins for the peptide are maintained in the database structure: [evidence span withheld — anchor status: not_found]
