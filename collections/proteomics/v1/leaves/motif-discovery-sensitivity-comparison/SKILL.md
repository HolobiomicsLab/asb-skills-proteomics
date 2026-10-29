---
name: motif-discovery-sensitivity-comparison
description: Use when a PTM-site table includes per-site localization probabilities
  and you must decide whether low-probability identifications can be legitimately
  pooled into motif discovery, or should be filtered out as random identifications.
when_to_use_negative:
- The PTM-site table lacks per-site localization probabilities, so no high/low probability
  partition can be made
- Sequences are not pre-aligned to a fixed length with a fixed central residue (rmotifx's
  stated requirement), e.g. unaligned k-mer screening or DNA sequences
- The goal is de novo motif discovery itself rather than a sensitivity comparison
  of an existing motif set
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: motifeR
  role: Integrated R web software used to run motif discovery and sequence pre-alignment
    on the full and subset site sets
  repo: https://github.com/wangshisheng/motifeR
- name: rmotifx
  role: R implementation of motif-x that extracts overrepresented motifs from the
    foreground (site set) against a dynamic statistical background; installed locally
    via motifeR
  repo: https://github.com/omarwagih/rmotifx
- name: R
  role: Environment in which the site-set partitioning, dual motif-x runs, and per-motif
    comparison are scripted
  repo: https://www.r-project.org/
- name: ggseqlogo
  role: Optional companion package used by motifeR for sequence-logo visualization
    of motifs
  repo: https://github.com/omarwagih/ggseqlogo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201900245
    title: 'motifeR: An Integrated Web Software for Identification and Visualization
      of Protein Posttranslational Modification Motifs'
  source_content_hash: sha256:53339ab58d6c4c078d8a56fbe8c98cd372aa4fd20755255acfd8870eca9b106d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/wangshisheng/motifeR
  tools:
  - R
  - ggseqlogo
  - motifeR
  - rmotifx
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# motif-discovery-sensitivity-comparison

## Summary

A sensitivity-analysis skill that re-runs motif-x-based motif discovery on both the full PTM-site set and a high-localization-probability subset, to test whether motifs supported only by low-localization-probability peptides are real signal or random artifacts.

## When to use

Use when a PTM-site table includes per-site localization probabilities and you must decide whether low-probability identifications can be legitimately pooled into motif discovery, or should be filtered out as random identifications.

## When NOT to use

- The PTM-site table lacks per-site localization probabilities, so no high/low probability partition can be made
- Sequences are not pre-aligned to a fixed length with a fixed central residue (rmotifx's stated requirement), e.g. unaligned k-mer screening or DNA sequences
- The goal is de novo motif discovery itself rather than a sensitivity comparison of an existing motif set

## Inputs

- PTM-site table with per-site localization probabilities (e.g. phosphosite table from a phosphoproteomics search/PTM scoring tool)
- Motif outputs from a prior motif discovery run (task_001)
- Background sequence set for motif-x (pre-aligned sequences with fixed central residue)

## Outputs

- Per-motif comparison table written to motif_lowlocp_comparison.csv with presence and peptide-count fields for the full-set run versus the high-probability-subset run
- Count of motifs supported only by peptides with low localization probability

## How to apply

Load the PTM-site table (per-site localization probabilities plus motif outputs from the prior discovery run) into R. Partition the site set into the full set and a high-localization-probability-only subset using a probability threshold, then run motif extraction on both with motif-x as wrapped by motifeR's locally installed rmotifx package (sequences must be pre-aligned to a fixed length with a fixed central residue, e.g. phosphorylation sites on S/T/Y; rmotifx parameters such as min.seqs and pval.cutoff control motif significance). Compare the two motif result sets, recording for each motif its presence and foreground peptide counts (fg.matches/fg.size) in the full-set run versus the high-probability-subset run. Count motifs present in the full run but absent from the subset run — these are supported only by low-probability peptides. If such motifs exist and are overrepresented (low p-values, high fold.increase), the finding is that low-location-probability peptides are not identified randomly and can be included to support motif discovery rather than discarded.

## Related tools

- **motifeR** (Integrated R web software used to run motif discovery and sequence pre-alignment on the full and subset site sets) — https://github.com/wangshisheng/motifeR
- **rmotifx** (R implementation of motif-x that extracts overrepresented motifs from the foreground (site set) against a dynamic statistical background; installed locally via motifeR) — https://github.com/omarwagih/rmotifx
- **R** (Environment in which the site-set partitioning, dual motif-x runs, and per-motif comparison are scripted) — https://www.r-project.org/
- **ggseqlogo** (Optional companion package used by motifeR for sequence-logo visualization of motifs) — https://github.com/omarwagih/ggseqlogo

## Examples

```
mot = motifx(fg.seqs, bg.seqs, central.res = 'S', min.seqs = 20, pval.cutoff = 1e-6)
```

## Evaluation signals

- The script runs without error end to end
- motif_lowlocp_comparison.csv exists and contains presence and peptide-count fields for each motif in both runs
- The count of motifs supported only by low-localization-probability peptides lies within [0, total motifs]
- Each reported motif carries motif-x significance fields (p-value/score and fold.increase), confirming the runs used statistically significant thresholds rather than ad hoc pattern counts
- Motifs retained in both runs should have consistent fg.matches/fg.size relationships between the full-set and subset runs

## Limitations

- rmotifx only supports sequences of fixed (pre-aligned) length with a fixed central residue; degenerate motifs and non-centered k-mers are not supported
- The conclusion that low-probability peptides are not identified randomly depends on the chosen probability threshold for the subset partition; results may vary with different thresholds
- min.seqs (minimum foreground matches) and pval.cutoff settings influence which motifs are reported, so very low-count motifs may be missed
- The skill assumes the background set is appropriate (e.g. random serine-centered sites); a poorly matched background biases fold.increase and p-values

## Evidence

- [abstract] Low-location-probability peptides may not be random: "peptides with low location probability may be not identified randomly and can be included to support motif discovery"
- [methods] motifeR wraps rmotifx and ggseqlogo: "motifeR needs two more packages ([rmotifx](https://github.com/omarwagih/rmotifx) and [ggseqlogo]"
- [abstract] motifeR purpose: "In this study, a versatile and user-friendly web tool is developed, motifeR, for extracting and visualizing statistically significant motifs from large datasets."
- [readme] rmotifx pre-alignment requirement: [evidence span withheld — anchor status: external_doc]
- [readme] motif-x algorithm description: [evidence span withheld — anchor status: external_doc]
