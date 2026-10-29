---
name: classifier-prediction-count-comparison
description: Use when a classifier's predicted false positives need to be relabeled
  as inferred positives via sequence alignment against experimentally derived phosphosites,
  and the improvement must be quantified per classifier.
when_to_use_negative:
- The goal is Tyr phosphosite inference — the article reports PhosBoost was not as
  informative at predicting inferred positives for Tyr, possibly due to higher label
  imbalance
- Input already contains experimentally verified positive labels for every site, so
  no inferred-positive relabeling is needed
- Only model quality metrics (AUPRC/AUROC) on held-out test data are needed, not phosphosite
  annotation or false-positive reduction counts
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3510
tools:
- name: PhosBoost
  role: Produces the S/T and Y phosphorylation predictions whose false positives are
    relabeled as inferred positives
  repo: https://github.com/eporetsky/PhosBoost
- name: DIAMOND
  role: Performs pairwise sequence alignment of 31-residue phosphosite peptide windows
    against the complete qPTMplants database (--masking none --ultra-sensitive --max-target-seqs
    100) to identify matches with experimentally derived phosphosites
- name: ProtT5-XL-U50
  role: Pre-trained protein language model used upstream to generate residue and protein
    embedding vectors for PhosBoost prediction
  repo: https://github.com/agemagician/ProtTrans
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:35c6d2b7b6fc9b0802295470e6a7b95cfc9b00b2ad636d8c83a692526040e50e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eporetsky/PhosBoost
  tools:
  - DIAMOND
  - PhosBoost
  - ProtT5-XL-U50
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3510
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# classifier-prediction-count-comparison

## Summary

Tabulates and compares per-classifier counts of predicted true positives, inferred positives (IP), and false positives before and after a DIAMOND pairwise-alignment relabeling step, to quantify whether alignment-based inference of positives improves phosphorylation prediction results.

## When to use

Use when a classifier's predicted false positives need to be relabeled as inferred positives via sequence alignment against experimentally derived phosphosites, and the improvement must be quantified per classifier.

## When NOT to use

- The goal is Tyr phosphosite inference — the article reports PhosBoost was not as informative at predicting inferred positives for Tyr, possibly due to higher label imbalance
- Input already contains experimentally verified positive labels for every site, so no inferred-positive relabeling is needed
- Only model quality metrics (AUPRC/AUROC) on held-out test data are needed, not phosphosite annotation or false-positive reduction counts

## Inputs

- Classifier prediction results (e.g. PhosBoost, PhosphoLingo, DeepPhos) on Ser/Thr or Tyr test or genome-wide sites
- Query protein FASTA with candidate phosphosites (e.g. Arabidopsis.qPTMplants.fasta)
- Complete qPTMplants protein FASTA (qPTMplants.fasta) for the DIAMOND subject database
- Experimentally derived positive phosphosite list (data/sites/qPTMplants.sites)

## Outputs

- Inferred-positive site list CSV saved to data/inferred/ (e.g. <query>_qPTMplants_15.csv) with supporting annotation
- Per-classifier comparison table of predicted true positives, inferred positives, and remaining false positives (e.g. Figure 4c-d outcome: S/T TP+IP 16,185 to 23,728, FP 38,770 to 31,227, precision .43, recall .84)

## How to apply

First run the classification stage (e.g. PhosBoost stacking classifier of two CatBoost base classifiers) and collect predicted positive and negative phosphosites per residue class. Next, extract 31-residue peptide windows centered on each candidate Ser/Thr/Tyr phosphosite, build a DIAMOND subject database from the complete qPTMplants protein FASTA plus its positive phosphosite list (data/sites/qPTMplants.sites), and align query peptides with DIAMOND v2.1.6 using --masking none --ultra-sensitive --max-target-seqs 100. Test all pairwise alignment results for each query to find matches with experimentally derived phosphosites, excluding self-matches, and relabel matching predicted false positives as inferred positives (IP) with supporting annotation. Finally, compile the matched sites into an inferred-positive site list CSV (e.g. <query>_qPTMplants_15.csv) and tabulate inferred positives and remaining false positives per classifier, comparing against classification-only counts to verify the alignment step improved prediction results for all classifiers.

## Related tools

- **PhosBoost** (Produces the S/T and Y phosphorylation predictions whose false positives are relabeled as inferred positives) — https://github.com/eporetsky/PhosBoost
- **DIAMOND** (Performs pairwise sequence alignment of 31-residue phosphosite peptide windows against the complete qPTMplants database (--masking none --ultra-sensitive --max-target-seqs 100) to identify matches with experimentally derived phosphosites)
- **ProtT5-XL-U50** (Pre-trained protein language model used upstream to generate residue and protein embedding vectors for PhosBoost prediction) — https://github.com/agemagician/ProtTrans

## Examples

```
sbatch infer.sh <query> qPTMplants 15
```

## Evaluation signals

- The DIAMOND inference step increases inferred positive counts and reduces false positive counts relative to classification alone for each classifier (article: improved prediction results for all classifiers)
- Reproduced benchmark numbers: PhosBoost S/T predicted+inferred true positives rise from 16,185 to 23,728 and false positives drop from 38,770 to 31,227, achieving precision .43 and recall .84
- No query phosphosite is labeled an inferred positive via a self-match (self-matches must be excluded)
- Inferred-positive CSV exists in data/inferred/ with expected filename pattern <query>_qPTMplants_15.csv and supporting annotation columns
- Each inferred positive corresponds to a peptide-window alignment hit whose subject phosphosite is present in qPTMplants.sites

## Limitations

- For Tyr phosphosites, the DIAMOND inference step was not as informative at predicting inferred positives compared to false positives, possibly due to higher label imbalance
- Label uncertainty and mislabeling is only partially addressed by the DIAMOND annotation step; many phosphosites remain undetected by high-throughput phosphoproteomics, limiting negative label reliability
- PhosBoost has a precision-recall tradeoff with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos, which the alignment step mitigates but does not eliminate

## Evidence

- [results] Relabeling predicted false positives matching experimentally derived phosphosites as inferred positives: [evidence span withheld — anchor status: not_found]
- [results] DIAMOND alignment parameters and peptide windows: "Using DIAMOND, we extracted peptide sequences of size 31 centered at the Ser/Thr phosphosites in the test dataset and conducted pairwise alignments"
- [abstract] Findings: alignment improved results for all classifiers: "A sequence-based pairwise alignment step improved prediction results for all classifiers by effectively increasing the number of inferred positive phosphosites"
- [readme] README: infer.sh command pattern: [evidence span withheld — anchor status: not_found]
- [discussion] Limitation: Tyr inference not informative: "For Tyr phosphosites, PhosBoost was not as informative at predicting inferred positives compared to false positives (Figure S7C), possibly due to higher label imbalance"
