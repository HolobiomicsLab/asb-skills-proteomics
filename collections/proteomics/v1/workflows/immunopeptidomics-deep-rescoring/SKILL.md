---
name: immunopeptidomics-deep-rescoring-workflow
description: 'Use when MGF immunopeptidomics spectra must be searched without enzyme
  specificity, augmented with run-specific retention-time and predicted-spectrum features,
  rescored, and controlled at both PSM and peptide false-discovery levels.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - comet-database-searching
  - mgf-spectra-conversion-and-database-searching
  - cross-search-engine-peptide-overlap-analysis
  - immunopeptidomics-fdr-filtering
  - psm-target-decoy-confidence-filtering
  - target-decoy-fdr-filtering
  - target-decoy-fdr-estimation
  - retention-time-prediction-feature-generation
  - retention-time-prediction-transfer-learning
  - rt-feature-distribution-significance-testing
  - spectral-prediction-feature-generation
  - msms-spectrum-prediction-spectral-angle-calculation
  - spectral-angle-similarity-scoring
  - immunopeptidomics-psm-feature-extraction
  - psm-rescoring-with-percolator
  - psm-rescoring-with-predicted-spectral-features
  member_tools:
  - Comet
  - DeepRescore
  - PDV
  - ProteoWizard
  - AutoRT
  - Docker
  - GPTime
  - MS-GF+
  - MaxQuant
  - NextFlow
  - PGA
  - Percolator
  - X!Tandem
  - pDeep2
  - ANNOVAR
  - Customprodbj
  - HLAthena
  - MHCflurry
  - NeoFlow
  - NetMHCpan
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

# Immunopeptidomics Search, Feature Generation, and PSM Rescoring

## Summary

Search target-decoy spectra, select high-confidence training peptides, generate retention-time and spectral-angle features, then rescore and FDR-filter PSMs and peptides.


## When to use

Use when MGF immunopeptidomics spectra must be searched without enzyme specificity, augmented with run-specific retention-time and predicted-spectrum features, rescored, and controlled at both PSM and peptide false-discovery levels.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — search_psms

**Goal:** search immunopeptidomics spectra against a target-decoy protein database

**EDAM operation:** http://edamontology.org/operation_3759

**Inputs:** mgf/immunopeptidomics-msms, fasta/target-decoy-proteome, text/comet-search-parameters · **Outputs:** tsv/target-decoy-psms

**Candidate leaf skills:** `comet-database-searching` (primary), `mgf-spectra-conversion-and-database-searching`, `cross-search-engine-peptide-overlap-analysis`

**Tools (primary):** Comet, DeepRescore, PDV, ProteoWizard

**Other candidate tools:** AutoRT, Docker, GPTime, MS-GF+, MaxQuant, NextFlow, PGA, X!Tandem, pDeep2

**Grounding:** 1 KB(s); DOIs: 10.1002/pmic.201900334

### Stage 2 — select_training_psms

**Goal:** select high-confidence peptides for run-specific feature models

**EDAM operation:** http://edamontology.org/operation_2239

**Inputs:** tsv/target-decoy-psms · **Outputs:** tsv/fdr-filtered-training-psms

**Candidate leaf skills:** `immunopeptidomics-fdr-filtering` (primary), `psm-target-decoy-confidence-filtering`, `target-decoy-fdr-filtering`, `target-decoy-fdr-estimation`

**Tools (primary):** AutoRT, DeepRescore, Docker, GPTime, MS-GF+, MaxQuant, NextFlow, PGA, Percolator, ProteoWizard, X!Tandem, pDeep2

**Other candidate tools:** Koina, Oktoberfest, Prosit, Python, mokapot, ANNOVAR, Comet, Customprodbj, HLAthena, MHCflurry, NeoFlow, NetMHCpan, PDV

**Grounding:** 2 KB(s); DOIs: 10.1002/pmic.201900334, 10.1002/pmic.202300112

### Stage 3 — generate_rt_features

**Goal:** derive run-specific retention-time error features for every PSM

**EDAM operation:** http://edamontology.org/operation_3767

**Inputs:** mgf/immunopeptidomics-msms, tsv/target-decoy-psms, tsv/fdr-filtered-training-psms, model/pretrained-autort-base, text/deeprescore-search-and-instrument-parameters · **Outputs:** tsv/psm-retention-time-features

**Candidate leaf skills:** `retention-time-prediction-feature-generation` (primary), `retention-time-prediction-transfer-learning`, `rt-feature-distribution-significance-testing`

**Tools (primary):** ANNOVAR, AutoRT, Customprodbj, DeepRescore, Docker, GPTime, HLAthena, MHCflurry, MS-GF+, MaxQuant, NeoFlow, NetMHCpan, NextFlow, PGA, Percolator, ProteoWizard, X!Tandem, pDeep2


**Grounding:** 1 KB(s); DOIs: 10.1002/pmic.201900334

### Stage 4 — generate_spectral_features

**Goal:** derive predicted-versus-observed fragment-spectrum features for every PSM

**EDAM operation:** http://edamontology.org/operation_3646

**Inputs:** mgf/immunopeptidomics-msms, tsv/target-decoy-psms, tsv/fdr-filtered-training-psms, model/pretrained-pdeep2-instrument-model, text/deeprescore-search-and-instrument-parameters · **Outputs:** tsv/psm-spectral-angle-features

**Candidate leaf skills:** `spectral-prediction-feature-generation` (primary), `msms-spectrum-prediction-spectral-angle-calculation`, `spectral-angle-similarity-scoring`, `immunopeptidomics-psm-feature-extraction`

**Tools (primary):** ANNOVAR, AutoRT, Customprodbj, DeepRescore, Docker, HLAthena, MHCflurry, MS-GF+, MaxQuant, NeoFlow, NetMHCpan, NextFlow, PGA, Percolator, ProteoWizard, X!Tandem, pDeep2

**Other candidate tools:** Comet

**Grounding:** 1 KB(s); DOIs: 10.1002/pmic.201900334

### Stage 5 — rescore_and_control_fdr

**Goal:** rescore PSMs and emit final PSM and peptide sets at one-percent FDR

**EDAM operation:** http://edamontology.org/operation_3648

**Inputs:** mgf/immunopeptidomics-msms, tsv/target-decoy-psms, tsv/psm-retention-time-features, tsv/psm-spectral-angle-features, text/deeprescore-search-and-instrument-parameters · **Outputs:** tsv/deeprescore-psm-final, tsv/deeprescore-peptide-final

**Candidate leaf skills:** `psm-rescoring-with-percolator` (primary), `psm-rescoring-with-predicted-spectral-features`

**Tools (primary):** ANNOVAR, AutoRT, Customprodbj, DeepRescore, Docker, GPTime, HLAthena, MHCflurry, MS-GF+, MaxQuant, NeoFlow, NetMHCpan, NextFlow, PGA, Percolator, ProteoWizard, X!Tandem, pDeep2

**Other candidate tools:** Koina, Oktoberfest, Prosit, Python, mokapot

**Grounding:** 2 KB(s); DOIs: 10.1002/pmic.201900334, 10.1002/pmic.202300112

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
