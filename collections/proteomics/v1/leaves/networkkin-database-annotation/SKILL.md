---
name: networkkin-database-annotation
description: Use when a human phosphoproteomics differential site regulation table
  (site IDs, regulation/differential values) must be mapped to upstream kinases to
  build a kinase-substrate signaling network.
when_to_use_negative:
- Dataset is not human — the motifeR NetworKIN kinase-substrate feature is offered
  for human phosphoproteomics datasets
- Goal is de novo motif extraction rather than kinase-substrate mapping (motifeR's
  core motif-x/rmotifx workflow is the right path instead)
- Input lacks phosphosite identifiers or regulation/differential values, so sites
  cannot be mapped or scored per kinase
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: motifeR
  role: R package whose optional feature integrates NetworKIN kinase-substrate relations
    to map human phosphoproteomics sites to kinases
  repo: https://github.com/wangshisheng/motifeR
- name: data.table
  role: Loads the differential site regulation table in R
- name: igraph
  role: Constructs the kinase-substrate signaling graph
- name: ggraph
  role: Lays out and visualizes the kinase-substrate signaling network
- name: graphlayouts
  role: Network layout support for ggraph visualization
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201900245
    title: 'motifeR: An Integrated Web Software for Identification and Visualization
      of Protein Posttranslational Modification Motifs'
  source_content_hash: sha256:3ec31fe790b67976391e0b2fec2fca1ccb6faa350e0266e5315c4de560bbab65
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/wangshisheng/motifeR
  tools:
  - R
  - data.table
  - ggraph
  - graphlayouts
  - igraph
  - motifeR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# networkkin-database-annotation

## Summary

Annotates human phosphoproteomics modification sites with upstream kinases via the NetworKIN kinase-substrate relation database, as integrated in motifeR as an optional feature, enabling differential kinase signaling network characterization.

## When to use

Use when a human phosphoproteomics differential site regulation table (site IDs, regulation/differential values) must be mapped to upstream kinases to build a kinase-substrate signaling network.

## When NOT to use

- Dataset is not human — the motifeR NetworKIN kinase-substrate feature is offered for human phosphoproteomics datasets
- Goal is de novo motif extraction rather than kinase-substrate mapping (motifeR's core motif-x/rmotifx workflow is the right path instead)
- Input lacks phosphosite identifiers or regulation/differential values, so sites cannot be mapped or scored per kinase

## Inputs

- human phosphoproteomics differential site regulation table (site IDs, kinase-mappable phosphosites, regulation/differential values)
- NetworKIN kinase-substrate relation database (integrated in motifeR)

## Outputs

- kinase_activity_summary.csv (per-kinase substrate count and differential score)
- kinase_network.graphml (kinase-substrate signaling graph with kinase, substrate, and regulation fields on nodes/edges)

## How to apply

Load the human phosphoproteomics differential site regulation table (site IDs, kinase-mappable phosphosites, regulation/differential values) with data.table in R. Map modification sites to kinases using the NetworKIN kinase-substrate relations, which motifeR offers as an optional feature specifically for human phosphoproteomics datasets. Aggregate the mapped relations per kinase to compute a substrate count and a differential score from the site regulation values, writing kinase_activity_summary.csv. Construct the kinase-substrate signaling graph with igraph and lay it out/visualize with ggraph and graphlayouts, then export the network as kinase_network.graphml carrying kinase, substrate, and regulation fields on nodes/edges.

## Related tools

- **motifeR** (R package whose optional feature integrates NetworKIN kinase-substrate relations to map human phosphoproteomics sites to kinases) — https://github.com/wangshisheng/motifeR
- **data.table** (Loads the differential site regulation table in R)
- **igraph** (Constructs the kinase-substrate signaling graph)
- **ggraph** (Lays out and visualizes the kinase-substrate signaling network)
- **graphlayouts** (Network layout support for ggraph visualization)

## Examples

```
if(!require(motifeR)) devtools::install_github("wangshisheng/motifeRapp")
```

## Evaluation signals

- kinase_network.graphml exists and parses as valid GraphML
- GraphML contains kinase, substrate, and regulation fields on nodes/edges
- GraphML has a positive integer node count
- kinase_activity_summary.csv contains one row per mapped kinase with a substrate count and a differential score derived from the site regulation values
- Every edge in the graph corresponds to a NetworKIN kinase-substrate relation mapped onto a phosphosite in the input regulation table

## Limitations

- NetworKIN-based kinase-substrate mapping is an optional feature of motifeR, not its core motif-extraction function, and is applied to human phosphoproteomics datasets
- Only kinase-mappable phosphosites contribute to the network; unmapped sites are excluded from kinase differential scores
- Running motifeR locally requires installing R (optionally RStudio) plus packages including shiny, shinyBS, shinyjs, DT, gdata, knitr, ggplot2, ggsci, openxlsx, data.table, Biostrings, stringi, stringr, ggrepel, igraph, ggraph, graphlayouts, scales

## Evidence

- [readme] readme: motifeR can provide kinase-substrate analysis for human data sets: [evidence span withheld — anchor status: external_doc]
