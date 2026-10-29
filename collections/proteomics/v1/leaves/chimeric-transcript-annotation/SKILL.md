---
name: chimeric-transcript-annotation
description: Use when a cell line's RNA-seq fusion calls (STAR-Fusion output with
  SpliceType, LargeAnchorSupport, FFPM columns) must be filtered to biologically credible
  fusions and visualized as a Circos plot.
when_to_use_negative:
- Input is raw RNA-seq FASTQ rather than already-called STAR-Fusion fusion output
  — this skill filters and visualizes existing fusion calls, it does not perform fusion
  detection
- The goal is somatic fusion discovery in tumor-normal patient cohorts rather than
  cell line fusion landscape annotation
- Fusions lacking the required STAR-Fusion annotation columns (SpliceType, LargeAnchorSupport,
  FFPM) cannot be filtered by the IMOPAC rules
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: IMOPAC
  role: Defines the fusion filtering rules (mitochondrial/HLA, red herrings, 10% recurrence,
    SpliceType/LargeAnchorSupport/FFPM) and the reference 'Fusions' module Circos
    visualization
  repo: https://github.com/wolfgangsk07/IMOPAC
- name: circlize
  role: R package used to render the comprehensive gene fusion landscape as a Circos
    plot from the filtered gene-pair coordinates
- name: DepMap
  role: Source of the Public 22Q1 RNA-seq fusion profiles and pan-cell-line recurrence
    information used for filtering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.023
    title: 'IMOPAC: A web server for interactive multiomics and pharmacological analyses
      of patient-derived cancer cell lines'
  source_content_hash: sha256:7962fd5f32d5fd9898ad35e62cd9c6218990fbd38e45a92c546ee52749054fd4
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# chimeric-transcript-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate and filter a cell line's gene fusion profile from STAR-Fusion-derived DepMap RNA-seq calls (e.g. DepMap Public 22Q1), then render the surviving chimeric transcripts as a Circos plot using the circlize R package, mirroring IMOPAC's 'Fusions' module.

## When to use

Use when a cell line's RNA-seq fusion calls (STAR-Fusion output with SpliceType, LargeAnchorSupport, FFPM columns) must be filtered to biologically credible fusions and visualized as a Circos plot.

## When NOT to use

- Input is raw RNA-seq FASTQ rather than already-called STAR-Fusion fusion output — this skill filters and visualizes existing fusion calls, it does not perform fusion detection
- The goal is somatic fusion discovery in tumor-normal patient cohorts rather than cell line fusion landscape annotation
- Fusions lacking the required STAR-Fusion annotation columns (SpliceType, LargeAnchorSupport, FFPM) cannot be filtered by the IMOPAC rules

## Inputs

- STAR-Fusion-derived gene fusion call table from DepMap Public 22Q1 RNA-seq (columns include SpliceType, LargeAnchorSupport, FFPM, fusion gene-pair and chromosome coordinates)
- DepMap pan-cell-line fusion table for recurrence filtering (10% threshold)

## Outputs

- Filtered CSV table of fusion gene pairs passing all four IMOPAC filtering criteria, with gene-pair coordinates
- High-resolution Circos plot figure (pdf/tiff/svg/eps) of the cell line's comprehensive fusion landscape

## How to apply

Load the STAR-Fusion-derived fusion calls from the DepMap Public 22Q1 RNA-seq release for the cell line of interest. Apply IMOPAC's four filtering rules: (1) remove fusions involving mitochondrial chromosomes or HLA genes, (2) remove common false-positive fusions annotated as red herrings in the STAR-Fusion docs, (3) remove recurrent fusions observed in 10% or more of DepMap cell lines, and (4) remove fusions where SpliceType='INCL_NON_REF_SPLICE' and LargeAnchorSupport='NO_LDAS' and FFPM<0.1. Export the surviving gene pairs as a CSV with the gene-pair coordinates used for plotting. Render the fusion landscape as a Circos plot with the circlize R package, linking the two genomic coordinates of each fusion pair, mirroring the 'Fusions' module of IMOPAC.

## Related tools

- **IMOPAC** (Defines the fusion filtering rules (mitochondrial/HLA, red herrings, 10% recurrence, SpliceType/LargeAnchorSupport/FFPM) and the reference 'Fusions' module Circos visualization) — https://github.com/wolfgangsk07/IMOPAC
- **circlize** (R package used to render the comprehensive gene fusion landscape as a Circos plot from the filtered gene-pair coordinates)
- **DepMap** (Source of the Public 22Q1 RNA-seq fusion profiles and pan-cell-line recurrence information used for filtering)

## Examples

```
Rscript -e "library(circlize); circos.initializeWithIdeogram(); circos.genomicLink(fused_1, fused_2)"
```

## Evaluation signals

- The filtered CSV contains only fusions passing all four IMOPAC filtering criteria (no mitochondrial/HLA fusions, no red herrings, no fusions seen in >=10% of DepMap cell lines, no INCL_NON_REF_SPLICE+NO_LDAS+FFPM<0.1 calls)
- The output figure is a non-empty, downloadable high-resolution file in one of the IMOPAC download formats (pdf/tiff/svg/eps)
- The Circos plot shows genomic links connecting the two chromosome coordinates of each surviving fusion gene pair
- Filtered fusion count is substantially lower than the raw STAR-Fusion call count, consistent with four sequential filters being applied
- The rendering matches the visualization style of IMOPAC's 'Fusions' module for the same cell line

## Limitations

- Filtering depends on DepMap-wide recurrence rates; a fusion recurrent across cell lines is removed even if biologically real in the query line
- FFPM<0.1 combined with INCL_NON_REF_SPLICE and NO_LDAS may discard low-support but genuine fusions
- The IMOPAC source repository cannot ship the large .RData datasets (GitHub file-size limits), so the exact reference fusion data must be fetched separately via the URLs in urls.txt
- No software version number is stated for IMOPAC, complicating exact reproduction of the Fusions module

## Evidence

- [methods] The gene fusion profile was estimated based on RNA-seq data from DepMap Public 22Q1: "The gene fusion profile was estimated based on RNA-seq data from DepMap Public 22Q1"
- [methods] A Circos plot was used to visualize the comprehensive gene fusion landscape of the user-defined cell line in the “Fusions” module of IMOPAC: "A Circos plot was used to visualize the comprehensive gene fusion landscape of the user-defined cell line in the “Fusions” module of IMOPAC"
- [methods] removing common false-positive fusions (red herring annotations as described in the STAR-Fusion docs): "removing common false-positive fusions (red herring annotations as described in the STAR-Fusion docs)"
- [readme] Input 'R' to enter R console and install the following packages:: [evidence span withheld — anchor status: not_found]
