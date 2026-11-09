---
name: diamond-protein-sequence-alignment
description: Use when classification-only phosphosite predictions (e.g. PhosBoost
  S/T predictions) contain false positives that could be recovered as inferred positives
  by aligning 31-residue peptide windows against a curated plant phosphosite database
  such as qPTMplants.
when_to_use_negative:
- 'Tyr phosphosite inference: the article reports PhosBoost was not as informative
  at predicting inferred positives for Tyr compared to false positives, possibly due
  to higher label imbalance'
- When the goal is de novo phosphorylation prediction rather than phosphosite annotation/label-uncertainty
  reduction — this step only relabels candidates that match experimentally derived
  sites, it does not predict new sites
- When no experimentally derived phosphosite database (e.g. qPTMplants) is available
  for the organism, since the inference relies entirely on matching known sites
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: DIAMOND
  role: Pairwise aligner run in --ultra-sensitive mode (--masking none --max-target-seqs
    100) to align 31-residue query phosphosite peptides against the complete qPTMplants
    protein database
  repo: https://github.com/eporetsky/PhosBoost
- name: PhosBoost
  role: Gradient-boosting/ProteinLM classifier whose predicted false positives are
    re-annotated by the DIAMOND inference step
  repo: https://github.com/eporetsky/PhosBoost
- name: qPTMplants
  role: Source of the subject database (proteins + experimentally derived phosphosite
    list) used for inferring positives
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:34b7f7ad38d26d3d24e840288bb50b4ecc9bd14aff77c16fe66bc546d432c7b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eporetsky/PhosBoost
  tools:
  - DIAMOND
  - PhosBoost
  - qPTMplants
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# diamond-protein-sequence-alignment

## Summary

A DIAMOND pairwise sequence-alignment post-processing step that supplements PhosBoost phosphorylation predictions by matching predicted false positives to experimentally derived phosphosites in the qPTMplants database and relabeling them as inferred positives (IP).

## When to use

Use when classification-only phosphosite predictions (e.g. PhosBoost S/T predictions) contain false positives that could be recovered as inferred positives by aligning 31-residue peptide windows against a curated plant phosphosite database such as qPTMplants.

## When NOT to use

- Tyr phosphosite inference: the article reports PhosBoost was not as informative at predicting inferred positives for Tyr compared to false positives, possibly due to higher label imbalance
- When the goal is de novo phosphorylation prediction rather than phosphosite annotation/label-uncertainty reduction — this step only relabels candidates that match experimentally derived sites, it does not predict new sites
- When no experimentally derived phosphosite database (e.g. qPTMplants) is available for the organism, since the inference relies entirely on matching known sites

## Inputs

- Query protein FASTA with candidate phosphosites (e.g. PhosBoost predicted false positives from an ST or Y model)
- Complete qPTMplants protein FASTA (qPTMplants.fasta) for the DIAMOND subject database
- qPTMplants positive phosphosite list (data/sites/qPTMplants.sites)
- PhosBoost trained model outputs (e.g. Arabidopsi.qPTMplants.ST.joblib.pkl predictions)

## Outputs

- Inferred-positive site list CSV in data/inferred/ (e.g. <query>_qPTMplants_15.csv) with supporting annotation
- Per-classifier counts of inferred positives and remaining false positives comparable to Figure 4c-d

## How to apply

Extract a 31-residue peptide window centered on each candidate Ser/Thr/Tyr phosphosite (shifting the site position for edge sites so the phosphosite is placed by distance from the sequence edge). Build the DIAMOND subject database from the complete qPTMplants protein FASTA (qPTMplants.fasta) plus its positive phosphosite list (data/sites/qPTMplants.sites), as configured by the repo's infer.sh step. Align each query peptide with DIAMOND v2.1.6 using --masking none --ultra-sensitive --max-target-seqs 100 against all protein sequences in the subject database. Test all pairwise hits for each query, exclude self-matches, and relabel query phosphosites that match any experimentally derived phosphosite as inferred positives. Compile matched sites into an inferred-positive CSV saved to data/inferred/ (e.g. <query>_qPTMplants_15.csv) with supporting annotation, then tabulate inferred positives and remaining false positives per classifier against the classification-only baseline.

## Related tools

- **DIAMOND** (Pairwise aligner run in --ultra-sensitive mode (--masking none --max-target-seqs 100) to align 31-residue query phosphosite peptides against the complete qPTMplants protein database) — https://github.com/eporetsky/PhosBoost
- **PhosBoost** (Gradient-boosting/ProteinLM classifier whose predicted false positives are re-annotated by the DIAMOND inference step) — https://github.com/eporetsky/PhosBoost
- **qPTMplants** (Source of the subject database (proteins + experimentally derived phosphosite list) used for inferring positives)

## Examples

```
sbatch infer.sh <query> qPTMplants 15
```

## Evaluation signals

- The DIAMOND step increases true positive + inferred positive phosphosites relative to classification alone (article: 16,185 to 23,728 for PhosBoost S/T) and reduces false positives (38,770 to 31,227)
- Resulting precision/recall reach approximately .43 precision and .84 recall for PhosBoost S/T after inference
- Every query peptide is exactly 31 residues with the candidate phosphosite centered (or offset by edge distance for edge sites)
- Self-matches are excluded when testing pairwise alignment results for inferred positives
- Output CSV follows the repo naming convention in data/inferred/ (e.g. <query>_qPTMplants_15.csv) and the step improves inferred-positive counts for all classifiers, consistent with the reported finding

## Limitations

- Label uncertainty and mislabeling in phosphorylation data is a common problem only partially addressed by the DIAMOND annotation step
- For Tyr phosphosites the inference step was not as informative, possibly due to higher label imbalance in Tyr experimental data
- Many phosphosites remain undetected by high-throughput phosphoproteomic approaches, limiting the reliability of negative labels and thus of the false-positive set being re-annotated

## Evidence

- [intro] supplemented PhosBoost predictions with a DIAMOND (Buchfink et al., 2021) pairwise alignment analysis step: "supplemented PhosBoost predictions with a DIAMOND (Buchfink et al., 2021) pairwise alignment analysis step"
- [results] 31-residue windows and pairwise alignments: "Using DIAMOND, we extracted peptide sequences of size 31 centered at the Ser/Thr phosphosites in the test dataset and conducted pairwise alignments"
- [results] relabeling matches as inferred positives: "Phosphosites matching any experimentally derived phosphosite, excluding self‐matches, were relabeled as inferred positives (IP)"
- [other] infer.sh invocation pattern: [evidence span withheld — anchor status: not_found]
