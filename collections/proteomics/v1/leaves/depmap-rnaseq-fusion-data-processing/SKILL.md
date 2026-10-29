---
name: depmap-rnaseq-fusion-data-processing
description: Use when a task needs to process DepMap Public 22Q1 RNA-seq fusion calls
  (STAR-Fusion format) for a specific cell line and produce a filtered, plot-ready
  fusion-gene-pair table or fusion Circos visualization.
when_to_use_negative:
- The input is already a filtered, plot-ready fusion pair table (filters already applied)
  — re-running the filter steps would be redundant
- The task requires de novo fusion detection from raw FASTQ; this skill operates on
  STAR-Fusion-derived DepMap fusion calls, not on alignment/fusion-calling itself
- The goal is drug response or expression analysis in IMOPAC (e.g. pancancer expression,
  drug-omics correlation) rather than fusion landscape characterization
edam_operation: http://edamontology.org/operation_3080
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: IMOPAC
  role: Source of the four fusion filtering rules and the reference 'Fusions' module
    Circos visualization of a cell line's fusion landscape
  repo: https://github.com/wolfgangsk07/IMOPAC
- name: circlize
  role: R package used to render the comprehensive gene fusion landscape as a Circos
    plot from the filtered fusion pair coordinates
- name: DepMap
  role: Public 22Q1 RNA-seq release providing the STAR-Fusion-derived gene fusion
    profile that constitutes the raw input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.023
    title: 'IMOPAC: A web server for interactive multiomics and pharmacological analyses
      of patient-derived cancer cell lines'
  source_content_hash: sha256:62b4c7935c3b8daa61184ef7580b9c2f51ef09c9a68fb92616800d0de0c32ef3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wolfgangsk07/IMOPAC
  repo_url: https://github.com/wolfgangsk07/IMOPAC
  tools:
  - DepMap
  - IMOPAC
  - circlize
  edam_operation: http://edamontology.org/operation_3080
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# depmap-rnaseq-fusion-data-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derive a curated gene fusion landscape from STAR-Fusion-based DepMap Public 22Q1 RNA-seq fusion calls by applying the IMOPAC four-step false-positive filtering rules, then export surviving fusion pairs as a CSV and render them as a Circos plot.

## When to use

Use when a task needs to process DepMap Public 22Q1 RNA-seq fusion calls (STAR-Fusion format) for a specific cell line and produce a filtered, plot-ready fusion-gene-pair table or fusion Circos visualization.

## When NOT to use

- The input is already a filtered, plot-ready fusion pair table (filters already applied) — re-running the filter steps would be redundant
- The task requires de novo fusion detection from raw FASTQ; this skill operates on STAR-Fusion-derived DepMap fusion calls, not on alignment/fusion-calling itself
- The goal is drug response or expression analysis in IMOPAC (e.g. pancancer expression, drug-omics correlation) rather than fusion landscape characterization

## Inputs

- DepMap Public 22Q1 RNA-seq gene fusion calls (STAR-Fusion-derived fusion profile) for a user-defined cell line, e.g. HUH28
- STAR-Fusion red herring / common false-positive annotation list
- DepMap-wide fusion recurrence table across cell lines (for the 10% recurrence filter)

## Outputs

- Filtered CSV table of surviving fusion pairs with the gene-pair coordinates used for plotting
- High-resolution Circos plot (pdf/tiff/svg/eps) of the cell line's comprehensive fusion landscape, e.g. result_huh28_fusion_landscape.png

## How to apply

Load the cell line's gene fusion calls from the DepMap Public 22Q1 RNA-seq release, where the fusion profile was estimated from RNA-seq data. Apply the IMOPAC filtering rules in order: (1) remove fusions involving mitochondrial chromosomes or HLA genes; (2) remove common false-positive fusions annotated as red herrings in the STAR-Fusion docs; (3) remove recurrent fusions observed in 10% or more of DepMap cell lines; (4) remove fusions where SpliceType='INCL_NON_REF_SPLICE' AND LargeAnchorSupport='NO_LDAS' AND FFPM<0.1. Export the surviving fusion pairs as a CSV table containing the gene-pair coordinates used for plotting. Finally, render the comprehensive fusion landscape as a Circos plot with the circlize R package, mirroring the 'Fusions' module visualization of IMOPAC. The rationale for each filter is to suppress annotation artifacts and low-confidence calls (mitochondrial/HLA events, red herring false positives, ubiquitous recurrent fusions, and weakly supported non-canonical splice fusions lacking large anchor support and low fusion-per-million evidence).

## Related tools

- **IMOPAC** (Source of the four fusion filtering rules and the reference 'Fusions' module Circos visualization of a cell line's fusion landscape) — https://github.com/wolfgangsk07/IMOPAC
- **circlize** (R package used to render the comprehensive gene fusion landscape as a Circos plot from the filtered fusion pair coordinates)
- **DepMap** (Public 22Q1 RNA-seq release providing the STAR-Fusion-derived gene fusion profile that constitutes the raw input)

## Examples

```
library(circlize); circos.initializeWithIdeogram(); circos.genomicLink(fusion_pairs[, c('g1_chr','g1_start','g1_end')], fusion_pairs[, c('g2_chr','g2_start','g2_end')])
```

## Evaluation signals

- The filtered CSV contains only fusions passing all four filtering criteria: no mitochondrial-chromosome or HLA-gene fusions, no STAR-Fusion red herring annotations, no fusions recurrent in ≥10% of DepMap cell lines, and no fusions with SpliceType='INCL_NON_REF_SPLICE', LargeAnchorSupport='NO_LDAS', and FFPM<0.1
- result_huh28_fusion_landscape.png exists as a non-empty, downloadable high-resolution figure in one of the IMOPAC download formats (pdf/tiff/svg/eps)
- Each surviving row in the CSV carries the gene-pair coordinates consumed by the Circos plot (no plotting orphaned or coordinate-less pairs)
- The rendered Circos plot visually mirrors the IMOPAC 'Fusions' module output for the same cell line and dataset release (DepMap Public 22Q1)

## Limitations

- Only fusions passing all four IMOPAC filters are kept; biologically real but recurrent (≥10% of DepMap cell lines) or weakly supported events are discarded by design
- The skill depends on the DepMap Public 22Q1 release; other releases or fusion callers are not validated by this workflow
- Figures are produced externally (circlize); IMOPAC's own figures are SVG generated by SVG.js v2.7, so direct pixel-level identity with the web server is not expected
- Per the README, the IMOPAC repository does not include the *.RData datasets due to GitHub file size limits; they must be downloaded separately via urls.txt

## Evidence
