---
name: phosphosite-label-annotation
description: Use when classifier phosphosite predictions show a precision-recall tradeoff
  with many predicted false positives, and an experimentally derived phosphosite database
  (e.g. qPTMplants) is available to relabel matching false positives as inferred positives.
when_to_use_negative:
- You need calibrated per-site probability scores rather than alignment-based label
  relabeling — DIAMOND inference supplements classification but does not replace model
  probabilities
- The query phosphosites have no plausible homologs in the experimentally derived
  database (e.g. divergent species with no qPTMplants coverage), since the step depends
  on pairwise sequence similarity
- Tyr phosphosite analysis where the article found the step 'not as informative at
  predicting inferred positives compared to false positives, possibly due to higher
  label imbalance'
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: DIAMOND
  role: Pairwise aligns 31-residue phosphosite-centered peptides (v2.1.6, --ultra-sensitive)
    against the qPTMplants database to find matches with experimentally derived phosphosites
- name: PhosBoost
  role: Supplies the S/T and Y phosphosite predictions whose false positives are relabeled
    as inferred positives
  repo: https://github.com/eporetsky/PhosBoost
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_005
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:e6484da511053ec376e4272d74fa260050a12ff4b25f616ce00592dc091dbf57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eporetsky/PhosBoost
  tools:
  - CatBoost
  - DIAMOND
  - PhosBoost
  - ProtT5-XL-U50
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# phosphosite-label-annotation

## Summary

A supplementary DIAMOND pairwise sequence-alignment step that annotates PhosBoost (or other classifier) phosphosite predictions by relabeling predicted false positives that match experimentally derived phosphosites as inferred positives, reducing label uncertainty and boosting effective recall.

## When to use

Use when classifier phosphosite predictions show a precision-recall tradeoff with many predicted false positives, and an experimentally derived phosphosite database (e.g. qPTMplants) is available to relabel matching false positives as inferred positives.

## When NOT to use

- You need calibrated per-site probability scores rather than alignment-based label relabeling — DIAMOND inference supplements classification but does not replace model probabilities
- The query phosphosites have no plausible homologs in the experimentally derived database (e.g. divergent species with no qPTMplants coverage), since the step depends on pairwise sequence similarity
- Tyr phosphosite analysis where the article found the step 'not as informative at predicting inferred positives compared to false positives, possibly due to higher label imbalance'

## Inputs

- Query protein FASTA with candidate Ser/Thr/Tyr phosphosites (e.g. PhosBoost prediction output)
- PhosBoost prediction TSV from /data/preds/
- Complete qPTMplants protein FASTA for the DIAMOND subject database
- qPTMplants positive phosphosite list (data/sites/qPTMplants.sites)

## Outputs

- Inferred-positive site list CSV saved to data/inferred/ (e.g. <query>_qPTMplants_15.csv) with supporting annotation
- Per-classifier counts of inferred positives and remaining false positives for comparison with classification-only predictions

## How to apply

Extract a 31-residue peptide window centered on each candidate Ser/Thr/Tyr phosphosite from the query protein sequences, positioning edge sites by their distance from the sequence edge. Build a DIAMOND subject database from the complete qPTMplants protein FASTA and its positive site list (data/sites/qPTMplants.sites) and align each query peptide with DIAMOND v2.1.6 using --masking none --ultra-sensitive --max-target-seqs 100. For each query, test all pairwise alignment results for matches to experimentally derived phosphosites, excluding self-matches, and relabel matching query phosphosites as inferred positives (IP). Compile matched sites into an inferred-positive CSV in data/inferred/ (e.g. <query>_qPTMplants_15.csv) with supporting annotation, then tabulate inferred positives and remaining false positives per classifier to compare against classification-only results.

## Related tools

- **DIAMOND** (Pairwise aligns 31-residue phosphosite-centered peptides (v2.1.6, --ultra-sensitive) against the qPTMplants database to find matches with experimentally derived phosphosites)
- **PhosBoost** (Supplies the S/T and Y phosphosite predictions whose false positives are relabeled as inferred positives) — https://github.com/eporetsky/PhosBoost

## Examples

```
sbatch infer.sh <query> qPTMplants 15
```

## Evaluation signals

- The inferred-positive counts increase and false-positive counts decrease relative to classification alone — reported outcome: S/T true positives rose from 16,185 to 23,728 and false positives fell from 38,770 to 31,227
- Resulting precision ~0.43 and recall ~0.84 for the S/T inference step on the A. thaliana test set
- Output CSV exists in data/inferred/ with the expected naming pattern <query>_qPTMplants_<offset>.csv and per-site supporting annotation
- All reported inferred positives exclude self-matches between the query peptide and its own database entry
- The improvement holds across all classifiers tested (the article reports the alignment step improved results for all classifiers)

## Limitations

- For Tyr phosphosites, the inference step was not as informative at predicting inferred positives compared to false positives, possibly due to higher label imbalance
- Label uncertainty and mislabeling is a common problem that is only partially addressed by this annotation step, since many phosphosites remain undetected by high-throughput phosphoproteomic approaches
- PhosBoost retains a precision-recall tradeoff with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos even after inference

## Evidence

- [results] Phosphosites matching any experimentally derived phosphosite, excluding self‐matches, were relabeled as inferred positives (IP): "Phosphosites matching any experimentally derived phosphosite, excluding self‐matches, were relabeled as inferred positives (IP)"
- [results] Using DIAMOND, we extracted peptide sequences of size 31 centered at the Ser/Thr phosphosites in the test dataset and conducted pairwise alignments: "Using DIAMOND, we extracted peptide sequences of size 31 centered at the Ser/Thr phosphosites in the test dataset and conducted pairwise alignments"
- [abstract] A sequence-based pairwise alignment step improved prediction results for all classifiers by effectively increasing the number of inferred positive phosphosites: "A sequence-based pairwise alignment step improved prediction results for all classifiers by effectively increasing the number of inferred positive phosphosites"
- [intro] To improve phosphosite annotation and reduce phosphosite label uncertainty, we supplemented PhosBoost predictions with a DIAMOND (Buchfink et al., 2021) pairwise alignment analysis step: "To improve phosphosite annotation and reduce phosphosite label uncertainty, we supplemented PhosBoost predictions with a DIAMOND (Buchfink et al., 2021) pairwise alignment analysis step"
