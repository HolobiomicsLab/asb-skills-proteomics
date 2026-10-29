---
name: motif-overrepresentation-significance-testing
description: Use when you have pre-aligned PTM peptide sequences (e.
when_to_use_negative:
- Sequences are not pre-aligned or lack a fixed central residue — rmotifx only supports
  fixed-length, centrally anchored sequences
- The goal is kinase-substrate network analysis or sequence logo visualization alone
  rather than motif overrepresentation testing — use motifeR's NetworKIN-based module
  or ggseqlogo instead
- DNA sequences or degenerate motifs are the input — listed as unsupported in the
  rmotifx TODO
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0601
- http://edamontology.org/topic_3520
tools:
- name: motifeR
  role: Integrated web/R software whose functions extract significant motifs and process
    multi-modification-site enrichment
  repo: https://github.com/wangshisheng/motifeR
- name: rmotifx
  role: R implementation of the motif-x algorithm used for overrepresentation logic;
    installed locally from the bundled rmotifx_1.0 tarball
  repo: https://github.com/omarwagih/rmotifx
- name: data.table
  role: R package used to load the public PTM site table (fread) into the workflow
- name: ggseqlogo
  role: R package bundled with motifeR (ggseqlogo_0.1.tar.gz) for visualization of
    extracted motifs as sequence logos
  repo: https://github.com/omarwagih/ggseqlogo
- name: motifeRapp
  role: Installable R/Shiny package of motifeR for local execution via devtools::install_github
    and motifeR_app()
  repo: https://github.com/wangshisheng/motifeRapp
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201900245
    title: 'motifeR: An Integrated Web Software for Identification and Visualization
      of Protein Posttranslational Modification Motifs'
  source_content_hash: sha256:d730f88fbe6a562e6a19d47c3040a22992c1b355db021aa745b7a735277061e5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/wangshisheng/motifeR
  tools:
  - R
  - data.table
  - ggseqlogo
  - motifeR
  - motifeRapp
  - rmotifx
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0601
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# motif-overrepresentation-significance-testing

## Summary

Tests whether post-translational modification motifs (including combinations of concurrent modification sites within multi-modified peptides) are statistically overrepresented, using the motif-x algorithm as implemented in rmotifx and the multi-modification-site enrichment functions integrated into motifeR.

## When to use

Use when you have pre-aligned PTM peptide sequences (e.g. phosphorylation sites with a fixed central residue) or a table of peptides carrying multiple concurrent modification sites, and you need to extract statistically significant motifs or compute enrichment ratios and p-values for modification-site combinations.

## When NOT to use

- Sequences are not pre-aligned or lack a fixed central residue — rmotifx only supports fixed-length, centrally anchored sequences
- The goal is kinase-substrate network analysis or sequence logo visualization alone rather than motif overrepresentation testing — use motifeR's NetworKIN-based module or ggseqlogo instead
- DNA sequences or degenerate motifs are the input — listed as unsupported in the rmotifx TODO

## Inputs

- Pre-aligned PTM peptide sequence set with fixed central residue (e.g. phosphorylation sites, foreground)
- Background sequence set (e.g. random serine-centered sites, ~10,000 sequences)
- PTM site table with concurrent modification sites (read via data.table::fread)

## Outputs

- Table of significant motifs with score, fg.matches, fg.size, bg.matches, bg.size, and fold.increase
- Data frame of multi-modification-site combinations with observed count, expected count, enrichment ratio, and p-value (e.g. multi_mod_enrichment.csv)

## How to apply

Load the PTM site table (e.g. with data.table::fread in R) and, for multi-modification-site analysis, use motifeR's integrated functions to extract peptides carrying multiple modification sites and enumerate their modification-site combinations. For motif discovery, run the rmotifx `motifx` function with a foreground sequence set and a background sequence set, fixing the central residue (e.g. central.res = 'S'), a minimum sequence count threshold (e.g. min.seqs = 20), and a p-value cutoff (e.g. pval.cutoff = 1e-6). Sequences must be pre-aligned to a fixed length with a fixed central residue, as rmotifx does not support degenerate or unaligned motifs. Compute the enrichment ratio (observed/expected counts) and a significance p-value for each motif or modification-site combination, then assemble results into a data frame with columns for combination/motif, observed count, expected count, enrichment ratio (or fold.increase), and p-value (or score).

## Related tools

- **motifeR** (Integrated web/R software whose functions extract significant motifs and process multi-modification-site enrichment) — https://github.com/wangshisheng/motifeR
- **rmotifx** (R implementation of the motif-x algorithm used for overrepresentation logic; installed locally from the bundled rmotifx_1.0 tarball) — https://github.com/omarwagih/rmotifx
- **data.table** (R package used to load the public PTM site table (fread) into the workflow)
- **ggseqlogo** (R package bundled with motifeR (ggseqlogo_0.1.tar.gz) for visualization of extracted motifs as sequence logos) — https://github.com/omarwagih/ggseqlogo
- **motifeRapp** (Installable R/Shiny package of motifeR for local execution via devtools::install_github and motifeR_app()) — https://github.com/wangshisheng/motifeRapp

## Examples

```
mot = motifx(fg.seqs, bg.seqs, central.res = 'S', min.seqs = 20, pval.cutoff = 1e-6); print(mot)
```

## Evaluation signals

- Output file (e.g. multi_mod_enrichment.csv) exists, parses without error, and contains the required fields: combination, observed count, expected count, enrichment ratio, p-value
- All enrichment ratios are numeric values in [0, inf)
- Returned motifs match the motifx output schema: motif, score, fg.matches, fg.size, bg.matches, bg.size, fold.increase
- Every reported motif/combination satisfies the chosen pval.cutoff (e.g. 1e-6) and min.seqs threshold (e.g. 20 foreground matches)
- The enrichment script runs end-to-end without error on the loaded PTM site table

## Limitations

- rmotifx only supports sequences of fixed length with a fixed central residue; degenerate motifs and DNA sequences are not supported
- Peptides with low location probability may not be identified randomly and can still be included to support motif discovery, so input filtering choices affect results
- Only some concurrent modification sites form motifs — multi-site enrichment is not guaranteed to yield significant combinations for every dataset

## Evidence

- [abstract] several functions are also integrated for processing multi-modification sites enrichment: "several functions are also integrated for processing multi-modification sites enrichment"
- [methods] motifeR needs two more packages: "motifeR needs two more packages ([rmotifx](https://github.com/omarwagih/rmotifx) and [ggseqlogo]"
- [readme] motif-x extracts overrepresented patterns from any sequence data set: [evidence span withheld — anchor status: external_doc]
- [readme] current implementation only supports fixed-length sequences with fixed central residue: [evidence span withheld — anchor status: external_doc]
- [abstract] indicating that some concurrent modification sites may form motifs: "indicating that some concurrent modification sites may form motifs"
