---
name: r-package-data-processing
description: Use when a PTM-site sequence table (e.g., phosphorylation sites with
  a fixed central residue) needs motif discovery and visualization and the workflow
  requires a locally installed R package rather than the web server.
when_to_use_negative:
- Sequences are not pre-aligned or do not share a fixed central residue — rmotifx
  only supports fixed-length, fixed central residue sequences
- The goal is kinase-substrate network analysis for non-human datasets — the NetworKIN-based
  feature is for human phosphoproteomics data
- A non-R, non-shiny environment is required — motifeR is an R/shiny tool; use the
  online version or another motif extractor instead
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3520
tools:
- name: motifeR
  role: Integrated R/shiny web software for sequence pre-alignment, motif discovery,
    and visualization of PTM motifs
  repo: https://github.com/wangshisheng/motifeR
- name: rmotifx
  role: R implementation of motif-x used by motifeR to extract overrepresented motifs
    from pre-aligned foreground/background sequences
  repo: https://github.com/omarwagih/rmotifx
- name: ggseqlogo
  role: R package used to draw publication-ready sequence logos of discovered motifs
  repo: https://github.com/omarwagih/ggseqlogo
- name: R
  role: Language in which motifeR is developed and locally run
  repo: https://www.r-project.org/
- name: RStudio
  role: Recommended (but not necessary) IDE for running motifeR locally
  repo: https://www.rstudio.com/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201900245
    title: 'motifeR: An Integrated Web Software for Identification and Visualization
      of Protein Posttranslational Modification Motifs'
  source_content_hash: sha256:783776db05bf607518ee55cb8228506d51d16917e8846dff26083db9b1eafbf9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/wangshisheng/motifeR
  tools:
  - R
  - RStudio
  - ggseqlogo
  - motifeR
  - rmotifx
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# r-package-data-processing

## Summary

Install and run the motifeR R package locally for extracting and visualizing statistically significant PTM motifs, using rmotifx (motif-x implementation) and ggseqlogo as supporting packages.

## When to use

Use when a PTM-site sequence table (e.g., phosphorylation sites with a fixed central residue) needs motif discovery and visualization and the workflow requires a locally installed R package rather than the web server.

## When NOT to use

- Sequences are not pre-aligned or do not share a fixed central residue — rmotifx only supports fixed-length, fixed central residue sequences
- The goal is kinase-substrate network analysis for non-human datasets — the NetworKIN-based feature is for human phosphoproteomics data
- A non-R, non-shiny environment is required — motifeR is an R/shiny tool; use the online version or another motif extractor instead

## Inputs

- Pre-aligned foreground sequences of PTM sites with a fixed central residue (e.g., serine-centered phosphopeptides)
- Background sequence set (e.g., 10,000 random serine-centered sites)
- PTM-site table with per-site localization probabilities

## Outputs

- Table of statistically significant motifs with score, fg.matches, fg.size, bg.matches, bg.size, fold.increase
- Sequence logos visualized with ggseqlogo
- Motif comparison tables (e.g., motif_lowlocp_comparison.csv)

## How to apply

Install R (and optionally RStudio), then check for required packages (shiny, shinyBS, shinyjs, DT, gdata, knitr, ggplot2, ggsci, openxlsx, data.table, Biostrings, stringi, stringr, ggrepel, igraph, ggraph, graphlayouts, scales) with a require() loop. Install motifeR locally via devtools::install_github("wangshisheng/motifeRapp"), plus rmotifx and ggseqlogo from the tarballs or GitHub repos hosted under wangshisheng/motifeR. Load motifeR and launch motifeR_app() to run the shiny web interface locally; motif extraction is performed by the motifx() function of rmotifx, which requires a pre-aligned foreground set with a fixed central residue (e.g., phosphorylation sites centered on S), a background sequence set, min.seqs and a pval.cutoff (README example: min.seqs = 20, pval.cutoff = 1e-6). The rationale is that motif-x builds successive overrepresented patterns through comparison to a dynamic statistical background, so sequence pre-alignment before motif discovery is a prerequisite. Outputs are a motif table with score, fg.matches, fg.size, bg.matches, bg.size, and fold.increase, which can be visualized with ggseqlogo.

## Related tools

- **motifeR** (Integrated R/shiny web software for sequence pre-alignment, motif discovery, and visualization of PTM motifs) — https://github.com/wangshisheng/motifeR
- **rmotifx** (R implementation of motif-x used by motifeR to extract overrepresented motifs from pre-aligned foreground/background sequences) — https://github.com/omarwagih/rmotifx
- **ggseqlogo** (R package used to draw publication-ready sequence logos of discovered motifs) — https://github.com/omarwagih/ggseqlogo
- **R** (Language in which motifeR is developed and locally run) — https://www.r-project.org/
- **RStudio** (Recommended (but not necessary) IDE for running motifeR locally) — https://www.rstudio.com/

## Examples

```
if(!require(motifeR)) devtools::install_github("wangshisheng/motifeRapp"); library(motifeR); motifeR_app()
```

## Evaluation signals

- The script runs without error after install_github("wangshisheng/motifeRapp") and library(motifeR)
- The motifx() output table contains all expected columns: motif, score, fg.matches, fg.size, bg.matches, bg.size, fold.increase
- Motif counts (fg.matches) are non-negative and <= fg.size, and fold.increase values are consistent with fg/bg match ratios
- Derived comparison outputs are valid, e.g., the count of motifs supported only by low-localization-probability peptides lies within [0, total motifs]
- motif_lowlocp_comparison.csv exists with presence and count fields populated

## Limitations

- rmotifx only supports sequences of fixed length with a fixed central residue; degenerate motifs and DNA sequences are not supported
- Local installation requires many prerequisite packages (shiny, shinyBS, shinyjs, DT, gdata, knitr, ggplot2, ggsci, openxlsx, data.table, Biostrings, stringi, stringr, ggrepel, igraph, ggraph, graphlayouts, scales) and locally installed rmotifx/ggseqlogo tarballs
- Sequences must be pre-aligned before motif discovery; motifeR provides sequence pre-alignment as a separate function

## Evidence

- [readme] the current implementation only supports sequences with a fixed length (i.e. pre-aligned) and have a fixed central residue: [evidence span withheld — anchor status: external_doc]
- [readme] mot = motifx(fg.seqs, bg.seqs, central.res = 'S', min.seqs = 20, pval.cutoff = 1e-6): [evidence span withheld — anchor status: external_doc]
