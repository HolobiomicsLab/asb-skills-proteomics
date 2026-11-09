---
name: isobaric-reporter-quantification-workflow
description: 'Use when TMT, TMTpro, or iTRAQ mzML runs must be searched, quantified
  at the reporter-ion level, filtered for co-isolation interference, aggregated to
  proteins, and tested for condition-level abundance changes.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - proteomics-database-search-and-psm-validation
  - proteomics-search-result-conversion
  - comet-database-searching
  - peptide-spectrum-matching-database-search
  - isobaric-reporter-ion-quantification
  - isobaric-tmt-peptide-quantification
  - isobaric-quantification-cross-tool-benchmarking
  - precursor-interference-qc-score-calculation
  - isobaric-peptide-qc-score-filtering
  - reporter-ion-interference-assessment
  - tmt-interference-qc-score-filtering
  - protein-inference-and-quantification-aggregation
  - protein-level-peptide-aggregation
  - differential-abundance-significance-testing
  - intra-protein-cv-precision-analysis
  member_tools:
  - Comet
  - IsobaricQuant
  - Java
  - Mokapot
  - msconvert
  - Java 17
  - Percolator
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/proteomics/v1
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Isobaric Reporter-Ion Quantification and Differential Abundance

## Summary

Produce validated peptide-spectrum matches, interference-aware peptide reporter intensities, protein-level reporter matrices, and differential-abundance results.


## When to use

Use when TMT, TMTpro, or iTRAQ mzML runs must be searched, quantified at the reporter-ion level, filtered for co-isolation interference, aggregated to proteins, and tested for condition-level abundance changes.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — identify_psms

**Goal:** identify and validate peptide-spectrum matches for isobaric quantification

**EDAM operation:** http://edamontology.org/operation_3630

**Inputs:** mzml/isobaric-msms, fasta/target-decoy-proteome, text/database-search-and-fdr-parameters · **Outputs:** csv/isobaricquant-validated-psm-hits

**Candidate leaf skills:** `proteomics-database-search-and-psm-validation` (primary), `proteomics-search-result-conversion`, `comet-database-searching`, `peptide-spectrum-matching-database-search`

**Tools (primary):** Comet, IsobaricQuant, Java, Mokapot, msconvert

**Other candidate tools:** Java 17, DeepRescore, PDV, ProteoWizard, Percolator

**Grounding:** 2 KB(s); DOIs: 10.1002/pmic.201900334, 10.1002/pmic.202100253

### Stage 2 — quantify_reporter_ions

**Goal:** extract peptide-level reporter intensities and scan interference scores

**EDAM operation:** http://edamontology.org/operation_3630

**Inputs:** mzml/isobaric-msms, csv/isobaricquant-validated-psm-hits, text/isobaricquant-configuration · **Outputs:** csv/isobaric-peptide-quantification-with-qc

**Candidate leaf skills:** `isobaric-reporter-ion-quantification` (primary), `isobaric-tmt-peptide-quantification`, `isobaric-quantification-cross-tool-benchmarking`, `precursor-interference-qc-score-calculation`

**Tools (primary):** Comet, IsobaricQuant, Java, Java 17, Percolator, msconvert


**Grounding:** 1 KB(s); DOIs: 10.1002/pmic.202100253

### Stage 3 — filter_interference

**Goal:** remove peptide reporter measurements with unsupported co-isolation scores

**EDAM operation:** http://edamontology.org/operation_3634

**Inputs:** csv/isobaric-peptide-quantification-with-qc, text/interference-thresholds · **Outputs:** csv/filtered-peptide-reporter-quantification, csv/interference-filter-qc

**Candidate leaf skills:** `isobaric-peptide-qc-score-filtering` (primary), `reporter-ion-interference-assessment`, `tmt-interference-qc-score-filtering`

**Tools (primary):** Comet, IsobaricQuant, Java, Java 17, msconvert

**Other candidate tools:** Percolator

**Grounding:** 1 KB(s); DOIs: 10.1002/pmic.202100253

### Stage 4 — aggregate_proteins

**Goal:** roll matched unfiltered and filtered peptide intensities up to proteins

**EDAM operation:** http://edamontology.org/operation_3630

**Inputs:** csv/isobaric-peptide-quantification-with-qc, csv/filtered-peptide-reporter-quantification, csv/isobaricquant-validated-psm-hits · **Outputs:** csv/unfiltered-protein-reporter-matrix, csv/filtered-protein-reporter-matrix

**Candidate leaf skills:** `protein-inference-and-quantification-aggregation` (primary), `protein-level-peptide-aggregation`

**Tools (primary):** Comet, IsobaricQuant, Java, Percolator, msconvert


**Grounding:** 1 KB(s); DOIs: 10.1002/pmic.202100253

### Stage 5 — test_differential_abundance

**Goal:** compare protein significance before and after interference filtering

**EDAM operation:** http://edamontology.org/operation_3659

**Inputs:** csv/unfiltered-protein-reporter-matrix, csv/filtered-protein-reporter-matrix, csv/isobaric-peptide-quantification-with-qc, csv/sample-design · **Outputs:** csv/unfiltered-differential-protein-abundance, csv/filtered-differential-protein-abundance, csv/differential-significance-changes

**Candidate leaf skills:** `differential-abundance-significance-testing` (primary), `intra-protein-cv-precision-analysis`

**Tools (primary):** Comet, IsobaricQuant, Java, Java 17, msconvert

**Other candidate tools:** IsobaricQuant benchmark

**Grounding:** 1 KB(s); DOIs: 10.1002/pmic.202100253

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
