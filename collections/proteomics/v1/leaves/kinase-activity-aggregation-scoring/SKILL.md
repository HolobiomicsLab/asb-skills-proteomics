---
name: kinase-activity-aggregation-scoring
description: Use when a differential human phosphoproteomics site table (site IDs,
  regulation/differential values) must be transformed into kinase-level activity scores
  and a kinase-substrate signaling network, rather than remaining at single-site resolution.
when_to_use_negative:
- The dataset is not a human phosphoproteomics set — the NetworKIN-based kinase-substrate
  analysis is an optional feature for human data sets, so non-human or non-phospho
  PTM data would lack mappable kinase-substrate relations.
- The goal is motif extraction or sequence-logo visualization itself (motif discovery),
  which is motifeR's primary rmotifx/ggseqlogo function; this skill starts only after
  site-level regulation values exist.
- Input is already a kinase activity/network summary — no site-to-kinase mapping or
  aggregation remains to be done.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0082
tools:
- name: motifeR
  role: Integrates the NetworKIN kinase-substrate relations and performs kinase-substrate
    analysis / differential kinase signaling network characterization as an optional
    feature
  repo: https://github.com/wangshisheng/motifeRapp
- name: NetworKIN
  role: Source database of kinase-substrate relations used to map modification sites
    to upstream kinases
- name: igraph
  role: Constructs the kinase-substrate signaling graph from the aggregated kinase-activity
    summary
- name: ggraph
  role: Layout and visualization of the kinase-substrate signaling network (with graphlayouts)
- name: data.table
  role: Loads the differential site regulation table in R
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201900245
    title: 'motifeR: An Integrated Web Software for Identification and Visualization
      of Protein Posttranslational Modification Motifs'
  source_content_hash: sha256:86db6fae0984f65a0d192aa615072ed553f53fd5da235476d96e7444d0adba94
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/wangshisheng/motifeRapp
  tools:
  - NetworKIN
  - R
  - data.table
  - ggraph
  - igraph
  - motifeR
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0082
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# kinase-activity-aggregation-scoring

## Summary

Maps human phosphoproteomics modification sites to upstream kinases via the NetworKIN kinase-substrate database and aggregates site-level regulation values into per-kinase differential scores, then models the resulting differential kinase signaling network. This is offered by motifeR as an optional feature for human datasets.

## When to use

Use when a differential human phosphoproteomics site table (site IDs, regulation/differential values) must be transformed into kinase-level activity scores and a kinase-substrate signaling network, rather than remaining at single-site resolution.

## When NOT to use

- The dataset is not a human phosphoproteomics set — the NetworKIN-based kinase-substrate analysis is an optional feature for human data sets, so non-human or non-phospho PTM data would lack mappable kinase-substrate relations.
- The goal is motif extraction or sequence-logo visualization itself (motif discovery), which is motifeR's primary rmotifx/ggseqlogo function; this skill starts only after site-level regulation values exist.
- Input is already a kinase activity/network summary — no site-to-kinase mapping or aggregation remains to be done.

## Inputs

- Human phosphoproteomics differential site regulation table (site IDs, kinase-mappable phosphosites, regulation/differential values), e.g. a CSV/TSV read with data.table
- NetworKIN kinase-substrate relation database (integrated in motifeR as an optional feature)

## Outputs

- kinase_activity_summary.csv with per-kinase substrate counts and differential scores
- kinase_network.graphml containing kinase, substrate, and regulation fields on nodes/edges, visualizable via igraph/ggraph

## How to apply

Load the differential site regulation table (site IDs, kinase-mappable phosphosites, regulation/differential values) with data.table in R. Map modification sites to kinases using the NetworKIN kinase-substrate relations, which motifeR integrates as an optional feature specifically for human phosphoproteomics datasets. Aggregate the mapped relations per kinase to compute a substrate count and a differential score derived from the site regulation values, writing kinase_activity_summary.csv. Build the kinase-substrate signaling graph with igraph, lay it out with ggraph/ggraphlayouts, and export it as kinase_network.graphml carrying kinase, substrate, and regulation fields on nodes and edges. This step complements motifeR's core motif discovery (rmotifx/ggseqlogo) by moving from sequence motifs to network-level kinase characterization.

## Related tools

- **motifeR** (Integrates the NetworKIN kinase-substrate relations and performs kinase-substrate analysis / differential kinase signaling network characterization as an optional feature) — https://github.com/wangshisheng/motifeRapp
- **NetworKIN** (Source database of kinase-substrate relations used to map modification sites to upstream kinases)
- **igraph** (Constructs the kinase-substrate signaling graph from the aggregated kinase-activity summary)
- **ggraph** (Layout and visualization of the kinase-substrate signaling network (with graphlayouts))
- **data.table** (Loads the differential site regulation table in R)

## Examples

```
library(motifeR); motifeR_app()  # run locally: devtools::install_github("wangshisheng/motifeRapp")
```

## Evaluation signals

- kinase_network.graphml exists and parses as valid GraphML
- The GraphML contains kinase, substrate, and regulation fields on nodes/edges
- The network has a positive integer node count
- kinase_activity_summary.csv contains one row per mapped kinase with a computed substrate count and differential score aggregating site regulation values
- Every kinase in the summary traces back to at least one mappable phosphosite via NetworKIN relations

## Limitations

- NetworKIN-based kinase-substrate analysis is an optional feature of motifeR and applies to human phosphoproteomics datasets; non-human data is out of scope.
- Kinases can only be assigned to modification sites mappable by the NetworKIN database; unmappable phosphosites are excluded from the aggregation.
- Package installation is nontrivial locally: motifeR requires extra packages (rmotifx, ggseqlogo) and dependencies such as shiny, shinyBS, shinyjs, DT, gdata, knitr, ggplot2, ggsci, openxlsx, data.table, Biostrings, stringi, stringr, ggrepel, igraph, ggraph, graphlayouts, scales.

## Evidence

- [abstract] Differential kinase signaling network characterization: "for human phosphoproteomics datasets, the characterization of differential kinase signaling networks can be estimated and modeled by combining kinase-substrate relations based on the NetworKIN"
- [abstract] Optional NetworKIN feature scope: "combining kinase-substrate relations based on the NetworKIN database as an optional feature for users"
- [readme] Human-specific kinase-substrate analysis: [evidence span withheld — anchor status: external_doc]
- [readme] Dependency list including network packages: "data.table, Biostrings, stringi, stringr, ggrepel, igraph, ggraph, graphlayouts, scales"
