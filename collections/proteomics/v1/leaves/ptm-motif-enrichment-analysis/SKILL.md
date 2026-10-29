---
name: ptm-motif-enrichment-analysis
description: Use when you have a PTM-site table (protein identifiers, modified-residue
  positions, fixed-length sequence windows centered on modification sites) from a
  phosphoproteomics MS dataset and need to find overrepresented consensus motifs around
  the modified residue.
when_to_use_negative:
- Sequences are not pre-aligned or lack a fixed central modified residue — rmotifx
  only supports fixed-length sequences with a fixed central residue
- The goal is de novo motif discovery in non-centered k-mers or degenerate motifs,
  which rmotifx does not support (listed as TODO)
- Input is raw mass-spec spectra rather than a PTM-site table with modification-site
  sequence windows
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0080
tools:
- name: motifeR
  role: Integrated Shiny web tool that wraps sequence pre-alignment, motif discovery,
    kinase-substrate analysis, and motif visualization
  repo: https://github.com/wangshisheng/motifeRapp
- name: rmotifx
  role: R implementation of the motif-x algorithm that extracts overrepresented patterns
    from foreground vs. background sequence windows
  repo: https://github.com/omarwagih/rmotifx
- name: ggseqlogo
  role: R package for generating publication-ready residue-frequency sequence logos
    of the discovered motifs
  repo: https://github.com/omarwagih/ggseqlogo
- name: R
  role: Runtime environment in which motifeR, rmotifx, and ggseqlogo are installed
    and run locally
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201900245
    title: 'motifeR: An Integrated Web Software for Identification and Visualization
      of Protein Posttranslational Modification Motifs'
  source_content_hash: sha256:1598614eee24f8490b319b3188d503f1c9f99571a2f58f64dd6ffccd50f2d9c4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/wangshisheng/motifeRapp
  tools:
  - R
  - ggseqlogo
  - motifeR
  - rmotifx
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# ptm-motif-enrichment-analysis

## Summary

Extracts statistically significant overrepresented post-translational modification (PTM) motifs from large mass spectrometry phosphoproteomics datasets using the motif-x algorithm (via rmotifx) wrapped by the motifeR Shiny app, and visualizes them as residue-frequency sequence logos with ggseqlogo.

## When to use

Use when you have a PTM-site table (protein identifiers, modified-residue positions, fixed-length sequence windows centered on modification sites) from a phosphoproteomics MS dataset and need to find overrepresented consensus motifs around the modified residue.

## When NOT to use

- Sequences are not pre-aligned or lack a fixed central modified residue — rmotifx only supports fixed-length sequences with a fixed central residue
- The goal is de novo motif discovery in non-centered k-mers or degenerate motifs, which rmotifx does not support (listed as TODO)
- Input is raw mass-spec spectra rather than a PTM-site table with modification-site sequence windows

## Inputs

- PTM-site table with protein identifiers, modified-residue positions, and sequence windows centered on modification sites
- Foreground sequence set: fixed-length, pre-aligned windows with a fixed central modified residue
- Background sequence set: residue-matched or proteome-wide windows for the dynamic statistical background
- Per-site localization probabilities (optional, for sensitivity analysis of low-localization-probability peptides)

## Outputs

- motifeR_motifs.csv: significant motifs with consensus sequence, score/p-value, and fold enrichment
- motifeR_motif_logo.png: residue-frequency sequence logo visualization of extracted motifs
- Per-motif comparison table (e.g., motif_lowlocp_comparison.csv) when testing motif support across site subsets

## How to apply

Prepare a foreground set of pre-aligned, fixed-length sequence windows centered on the modified residue (e.g., phosphorylation sites on S/T/Y) and a background set (e.g., proteome-wide or residue-matched windows); rmotifx only supports sequences with a fixed length and a fixed central residue. Load the site table in R, then run motif extraction with rmotifx's motifx() using parameters such as central.res (e.g., 'S'), min.seqs (e.g., 20), and a pval.cutoff (e.g., 1e-6) to extract statistically significant consensus motifs against the dynamic statistical background. Each reported motif carries a consensus sequence, score, foreground/background match counts, and fold.increase. Export significant motifs (consensus, score/p-value, fold enrichment) to a CSV such as motifeR_motifs.csv. Render the motifs as residue-frequency sequence logos with the ggseqlogo package and save the figure (e.g., motifeR_motif_logo.png). Alternatively, install and launch the integrated web tool locally with devtools::install_github('wangshisheng/motifeRapp') followed by motifeR_app(), which also integrates sequence pre-alignment, multi-modification-site enrichment, and NetworKIN-based kinase-substrate analysis for human phosphoproteomics datasets.

## Related tools

- **motifeR** (Integrated Shiny web tool that wraps sequence pre-alignment, motif discovery, kinase-substrate analysis, and motif visualization) — https://github.com/wangshisheng/motifeRapp
- **rmotifx** (R implementation of the motif-x algorithm that extracts overrepresented patterns from foreground vs. background sequence windows) — https://github.com/omarwagih/rmotifx
- **ggseqlogo** (R package for generating publication-ready residue-frequency sequence logos of the discovered motifs) — https://github.com/omarwagih/ggseqlogo
- **R** (Runtime environment in which motifeR, rmotifx, and ggseqlogo are installed and run locally) — https://www.r-project.org/

## Examples

```
mot = motifx(fg.seqs, bg.seqs, central.res = 'S', min.seqs = 20, pval.cutoff = 1e-6)
```

## Evaluation signals

- The extraction script runs to completion and motifeR_motifs.csv exists as a valid CSV containing motif, score, and fold fields
- All reported motif significance p-values fall within [0, 1]
- Output motifs follow the motif-x dot-notation consensus format (e.g., .......SD.E....) with fg.matches, fg.size, bg.matches, bg.size, and fold.increase columns populated
- Fold enrichment (fold.increase) values for significant motifs exceed 1, confirming overrepresentation against the background
- If comparing site subsets, the count of motifs supported only by low-localization-probability peptides lies within [0, total motifs]

## Limitations

- The current rmotifx implementation only supports fixed-length pre-aligned sequences with a fixed central residue; degenerate motifs and DNA sequences are unsupported
- motifeR requires several extra packages installed locally (rmotifx_1.0.tar.gz and ggseqlogo_0.1.tar.gz, plus shiny, shinyBS, shinyjs, DT, gdata, knitr, ggplot2, and others) before launching
- Kinase-substrate analysis via the NetworKIN database is an optional feature applicable to human phosphoproteomics datasets

## Evidence

- [readme] rmotifx fixed-length central-residue constraint: [evidence span withheld — anchor status: external_doc]
- [readme] motifx call parameters: [evidence span withheld — anchor status: external_doc]
- [readme] motifeR three main functions: [evidence span withheld — anchor status: external_doc]
