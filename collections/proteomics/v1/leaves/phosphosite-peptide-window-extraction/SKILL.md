---
name: phosphosite-peptide-window-extraction
description: Use when preparing candidate phosphosites from protein sequences for
  sequence-alignment-based inference, e.g. relabeling predicted false positives as
  inferred positives via DIAMOND alignment to experimentally derived phosphosite databases.
when_to_use_negative:
- Protein sequences contain missing or non-canonical amino acids, or the phosphorylation
  position does not match the protein sequence — these proteins/sites are discarded
  per the article's filter steps before window extraction.
- The goal is classifier training rather than alignment-based inference — PhosBoost
  training uses full residue embedding vectors, not extracted 31-residue peptide windows
  for alignment.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0601
- http://edamontology.org/topic_3510
tools:
- name: DIAMOND
  role: Consumes the extracted 31-residue peptide windows for ultra-sensitive pairwise
    alignment against the qPTMplants subject database (v2.1.6, --masking none --ultra-sensitive
    --max-target-seqs 100)
  repo: https://github.com/eporetsky/PhosBoost
- name: PhosBoost
  role: Pipeline whose inference step (infer.sh) extracts the peptide windows and
    builds the DIAMOND subject database from qPTMplants FASTA and site lists
  repo: https://github.com/eporetsky/PhosBoost
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:dc1454621f9cc1ac65a72ee1ef46ea7147060f01cb750802747734bf44ebde00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eporetsky/PhosBoost
  tools:
  - DIAMOND
  - PhosBoost
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0601
  - http://edamontology.org/topic_3510
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# phosphosite-peptide-window-extraction

## Summary

Extracts fixed-length (31-residue) peptide windows centered on candidate Ser/Thr/Tyr phosphosites from protein FASTA sequences, so that phosphorylation sites can be compared via pairwise sequence alignment (e.g. DIAMOND) rather than relying on classifier probabilities alone.

## When to use

Use when preparing candidate phosphosites from protein sequences for sequence-alignment-based inference, e.g. relabeling predicted false positives as inferred positives via DIAMOND alignment to experimentally derived phosphosite databases.

## When NOT to use

- Protein sequences contain missing or non-canonical amino acids, or the phosphorylation position does not match the protein sequence — these proteins/sites are discarded per the article's filter steps before window extraction.
- The goal is classifier training rather than alignment-based inference — PhosBoost training uses full residue embedding vectors, not extracted 31-residue peptide windows for alignment.

## Inputs

- Protein FASTA file (e.g. Arabidopsis.qPTMplants.fasta or a query proteome FASTA)
- Phosphosite list of candidate sites with residue type and position (e.g. data/sites/qPTMplants.sites)
- Subject database FASTA (complete qPTMplants protein sequences) and its positive phosphosite list for building the DIAMOND database

## Outputs

- 31-residue peptide window per candidate Ser/Thr/Tyr phosphosite, centered on the site
- Inferred-positive site list CSV in data/inferred/ (e.g. <query>_qPTMplants_15.csv) with supporting annotation

## How to apply

For each candidate phosphosite in the query protein set, take the residue's position within its protein sequence and cut a peptide of length 31 with the phosphosite at the center (offset by distance from the sequence edge for N- or C-terminal sites so the window still has the phosphosite positioned by edge distance). Sites in proteins with missing or non-canonical amino acids, or whose phosphorylation position does not cross-reference correctly to the protein sequence, must be discarded before window extraction to avoid misaligned peptides. The resulting peptides are then fed to DIAMOND (v2.1.6, --masking none --ultra-sensitive --max-target-seqs 100) against a subject database built from the complete qPTMplants FASTA plus its positive phosphosite list (qPTMplants.sites), with matches excluding self-alignments relabeled as inferred positives.

## Related tools

- **DIAMOND** (Consumes the extracted 31-residue peptide windows for ultra-sensitive pairwise alignment against the qPTMplants subject database (v2.1.6, --masking none --ultra-sensitive --max-target-seqs 100)) — https://github.com/eporetsky/PhosBoost
- **PhosBoost** (Pipeline whose inference step (infer.sh) extracts the peptide windows and builds the DIAMOND subject database from qPTMplants FASTA and site lists) — https://github.com/eporetsky/PhosBoost

## Examples

```
sbatch infer.sh <query> qPTMplants 15
```

## Evaluation signals

- Each extracted peptide is exactly 31 residues long with the phosphosite residue at the center (or positioned by distance from the sequence edge for edge sites).
- Peptide count equals the number of cross-referenced candidate Ser/Thr/Tyr phosphosites in the query FASTA after discarding non-canonical or position-mismatched proteins.
- Downstream DIAMOND step yields inferred positives that increase true positive counts (16,185 → 23,728) and reduce false positives (38,770 → 31,227) relative to classification alone.
- Inferred-positive CSV is written to data/inferred/ with the expected naming pattern (<query>_qPTMplants_15.csv) and includes supporting annotation columns.

## Limitations

- For Tyr phosphosites, the downstream alignment-inference step was not as informative at predicting inferred positives, possibly due to higher label imbalance.
- Label uncertainty and mislabeling of phosphosites is only partially addressed by the alignment-based annotation step; many phosphosites remain undetected by high-throughput phosphoproteomics.
- PhosBoost's precision-recall tradeoff means window extraction feeds into predictions with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos.

## Evidence

- [methods] We extracted a peptide of length 31 for each candidate phosphosite of Ser, Thr, and Tyr within a protein sequence, with the phosphosite at the center of the sequence: "We extracted a peptide of length 31 for each candidate phosphosite of Ser, Thr, and Tyr within a protein sequence, with the phosphosite at the center of the sequence"
- [results] Phosphosites matching any experimentally derived phosphosite, excluding self-matches, were relabeled as inferred positives (IP): "Phosphosites matching any experimentally derived phosphosite, excluding self‐matches, were relabeled as inferred positives (IP)"
- [methods] Proteins that are not present in genomic databases or contain missing or non-canonical amino-acid sequences were discarded: "Proteins that are not present in genomic databases or contain missing or non‐canonical amino‐acid sequences were discarded"
- [readme] sbatch infer.sh <query> qPTMplants 15 (workflow card; README DIAMOND inference section): "# 4. DIAMOND inference"
