---
name: graph-network-construction-and-visualization
description: Use when a human phosphoproteomics differential site regulation table
  (site IDs, kinase-mappable phosphosites, regulation/differential values) must be
  turned into a kinase-substrate signaling network that can be visualized and exported.
when_to_use_negative:
- The dataset is not a human phosphoproteomics dataset, since the NetworKIN kinase-substrate
  feature in motifeR is described specifically for human data
- The goal is motif extraction or multi-modification site enrichment only, with no
  kinase-substrate network to build
- Input is already a parsed GraphML/edge list network rather than a site-level regulation
  table
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: motifeR
  role: provides the optional NetworKIN kinase-substrate analysis feature that maps
    modification sites to kinases for human phosphoproteomics datasets
  repo: https://github.com/wangshisheng/motifeR
- name: igraph
  role: constructs the kinase-substrate signaling graph structure
- name: ggraph
  role: lays out and visualizes the kinase-substrate network
- name: graphlayouts
  role: network layout algorithms used alongside ggraph for the kinase-substrate graph
- name: data.table
  role: loads and aggregates the differential site regulation table and mapped kinase
    relations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201900245
    title: 'motifeR: An Integrated Web Software for Identification and Visualization
      of Protein Posttranslational Modification Motifs'
  source_content_hash: sha256:41afcf617033beb1c6fd5707f4902f81f700b8c7cc64cbbc87c87a548fdc6d65
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
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# graph-network-construction-and-visualization

## Summary

Constructs a kinase-substrate signaling graph from NetworKIN-mapped phosphosite-kinase relations and visualizes it with igraph/ggraph, as offered by motifeR's optional differential kinase signaling network feature for human phosphoproteomics data.

## When to use

Use when a human phosphoproteomics differential site regulation table (site IDs, kinase-mappable phosphosites, regulation/differential values) must be turned into a kinase-substrate signaling network that can be visualized and exported.

## When NOT to use

- The dataset is not a human phosphoproteomics dataset, since the NetworKIN kinase-substrate feature in motifeR is described specifically for human data
- The goal is motif extraction or multi-modification site enrichment only, with no kinase-substrate network to build
- Input is already a parsed GraphML/edge list network rather than a site-level regulation table

## Inputs

- human phosphoproteomics differential site regulation table (site IDs, kinase-mappable phosphosites, regulation/differential values)
- NetworKIN kinase-substrate relations (as integrated in motifeR as an optional feature)

## Outputs

- kinase_activity_summary.csv (per-kinase substrate count and differential score)
- kinase_network.graphml (kinase-substrate signaling graph with kinase, substrate, and regulation fields on nodes/edges)
- ggraph visualization of the kinase-substrate network

## How to apply

Load the differential site regulation table with data.table in R, then map modification sites to kinases via the NetworKIN kinase-substrate relations, integrated in motifeR as an optional feature for human datasets. Aggregate mapped relations per kinase to compute a substrate count and a differential score from the site regulation values, writing kinase_activity_summary.csv. Build the kinase-substrate signaling graph with igraph, carry kinase/substrate/regulation as node and edge attributes, and lay out/visualize the graph with ggraph and graphlayouts. Export the network as kinase_network.graphml with the kinase, substrate, and regulation fields on nodes/edges so downstream tools can parse the topology and per-edge regulation values.

## Related tools

- **motifeR** (provides the optional NetworKIN kinase-substrate analysis feature that maps modification sites to kinases for human phosphoproteomics datasets) — https://github.com/wangshisheng/motifeR
- **igraph** (constructs the kinase-substrate signaling graph structure)
- **ggraph** (lays out and visualizes the kinase-substrate network)
- **graphlayouts** (network layout algorithms used alongside ggraph for the kinase-substrate graph)
- **data.table** (loads and aggregates the differential site regulation table and mapped kinase relations)

## Examples

```
library(igraph); g <- graph_from_data_frame(edge_df, directed=TRUE); write_graph(g, 'kinase_network.graphml', format='graphml')
```

## Evaluation signals

- kinase_network.graphml exists and parses as valid GraphML
- the exported GraphML contains kinase, substrate, and regulation fields on nodes/edges
- the graph has a positive integer node count
- kinase_activity_summary.csv contains one aggregated row per kinase with substrate count and differential score derived from the site regulation values
- every edge in the graph corresponds to a mapped NetworKIN kinase-substrate relation for a phosphosite present in the input regulation table

## Limitations

- NetworKIN kinase-substrate integration is an optional feature restricted to human datasets, so non-human data cannot be mapped this way
- only kinase-mappable phosphosites contribute to the network; unmapped modification sites are excluded from kinase aggregation
- no discussion section content is available in the article, so failure modes beyond the human-data scope are not documented

## Evidence

- [abstract] for human phosphoproteomics datasets, the characterization of differential kinase signaling networks can be estimated and modeled by combining kinase-substrate relations based on the NetworKIN: "for human phosphoproteomics datasets, the characterization of differential kinase signaling networks can be estimated and modeled by combining kinase-substrate relations based on the NetworKIN"
- [abstract] combining kinase-substrate relations based on the NetworKIN database as an optional feature for users: "combining kinase-substrate relations based on the NetworKIN database as an optional feature for users"
- [readme] you should check whether you have installed these packages (shiny, shinyBS, shinyjs, DT, gdata, knitr, ggplot2, ggsci, openxlsx, data.table, Biostrings, stringi, stringr, ggrepel, igraph, ggraph, graphlayouts, scales): "you should check whether you have installed these packages (shiny, shinyBS, shinyjs, DT, gdata, knitr, ggplot2, ggsci, openxlsx, data.table, Biostrings, stringi, stringr, ggrepel, igraph, ggraph,"
- [readme] motifeR is a free, user-friendly and comprehensive web software, can provide three main functions (sequence pre-alignment, motif discovery, kinase-substrate analysis): [evidence span withheld — anchor status: not_found]
