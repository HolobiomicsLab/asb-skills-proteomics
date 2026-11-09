---
name: auc-based-signature-performance-thresholding
description: Use when you have a muSignAl output table of candidate signatures (e.g.
  sign_cvd2.xlsx) with per-signature AUC values and need to select the high-performing
  subset above a specified AUC threshold such as 0.95.
when_to_use_negative:
- Input is raw proteomic or omic intensity data (e.g. AD.xlsx) rather than an already-generated
  muSignAl signature output file — run the muSignAl algorithm first.
- The goal is to discover new signatures rather than to rank/filter existing ones;
  use the muSignAl algorithm itself instead of this thresholding step.
- Signatures lack AUC values computed with pROC from glmnet-fitted models, so the
  threshold cannot be applied meaningfully.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0203
tools:
- name: muSignAl
  role: Generates the multiple candidate signatures with similar predictive performance
    whose output file is thresholded by AUC in this skill
  repo: https://github.com/ShuklaLab/muSignAl
- name: pROC
  role: Provides the AUC values used for thresholding, computed from the glmnet-fitted
    models per signature
- name: glmnet
  role: Fits the regularized models from which each signature's AUC is derived prior
    to thresholding
- name: R
  role: Language in which the thresholding workflow (loading sign_cvd2.xlsx, filtering
    AUC > 0.95, writing filtered_signatures_auc_gt_095.xlsx) is implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202200252
    title: 'muSignAl: An algorithm to search for multiple omic signatures with similar
      predictive performance'
  source_content_hash: sha256:5a90af0155d4056ea3a9ee04c1ee535602da00ebea4adde7cca0a5948b4283f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShuklaLab/muSignAl
  tools:
  - R
  - glmnet
  - muSignAl
  - pROC
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# auc-based-signature-performance-thresholding

## Summary

Filters multi-omic signatures produced by muSignAl by predictive performance, keeping only those whose AUC (computed with pROC from glmnet-fitted models) exceeds a fixed cutoff (e.g. AUC > 0.95), reproducing the paper's Table 1 selection of 47 of 158 signatures on the Olink CVD-II dataset.

## When to use

Use when you have a muSignAl output table of candidate signatures (e.g. sign_cvd2.xlsx) with per-signature AUC values and need to select the high-performing subset above a specified AUC threshold such as 0.95.

## When NOT to use

- Input is raw proteomic or omic intensity data (e.g. AD.xlsx) rather than an already-generated muSignAl signature output file — run the muSignAl algorithm first.
- The goal is to discover new signatures rather than to rank/filter existing ones; use the muSignAl algorithm itself instead of this thresholding step.
- Signatures lack AUC values computed with pROC from glmnet-fitted models, so the threshold cannot be applied meaningfully.

## Inputs

- muSignAl final output spreadsheet (e.g. sign_cvd2.xlsx) with per-signature columns: signature number, constituent protein set, AUC
- AUC threshold cutoff (e.g. 0.95)

## Outputs

- Filtered, AUC-sorted table of signatures above the cutoff (e.g. filtered_signatures_auc_gt_095.xlsx) with columns signature number, protein set, AUC

## How to apply

Load the muSignAl final output file (e.g. sign_cvd2.xlsx from the package's example folder) into R with readxl or openxlsx, and parse the per-signature columns: signature number, constituent protein set, and AUC computed with the pROC package from the glmnet-fitted models. Apply the threshold filter by retaining only signature rows with AUC greater than the chosen cutoff (the paper uses AUC > 0.95). Sort the filtered rows by descending AUC and write them out (signature number, protein set, AUC) to a results file such as filtered_signatures_auc_gt_095.xlsx. The rationale is that muSignAl deliberately enumerates multiple signatures with similar predictive performance, so thresholding on AUC isolates the biomarker-panel candidates worth reporting (in the CVD-II example, 47 of 158 unique signatures, with the top signature 150 — CCL17/DECR1/ADM/PARP-1/SORT1 — reaching AUC 1.00).

## Related tools

- **muSignAl** (Generates the multiple candidate signatures with similar predictive performance whose output file is thresholded by AUC in this skill) — https://github.com/ShuklaLab/muSignAl
- **pROC** (Provides the AUC values used for thresholding, computed from the glmnet-fitted models per signature)
- **glmnet** (Fits the regularized models from which each signature's AUC is derived prior to thresholding)
- **R** (Language in which the thresholding workflow (loading sign_cvd2.xlsx, filtering AUC > 0.95, writing filtered_signatures_auc_gt_095.xlsx) is implemented)

## Examples

```
sig <- openxlsx::read.xlsx("sign_cvd2.xlsx"); top <- sig[sig$AUC > 0.95,]; top <- top[order(-top$AUC),]; openxlsx::write.xlsx(top, "filtered_signatures_auc_gt_095.xlsx")
```

## Evaluation signals

- The filtered set contains exactly 47 signature rows when applied to the paper's 158-signature CVD-II output with AUC > 0.95.
- The top row is signature 150 (CCL17, DECR1, ADM, PARP-1, SORT1) with AUC = 1.00.
- The filtered table reproduces Table 1 of the article when compared row by row.
- Every retained row has an AUC value strictly greater than the 0.95 cutoff.
- Output file (filtered_signatures_auc_gt_095.xlsx) has the expected columns: signature number, protein set, AUC, sorted by descending AUC.

## Limitations

- The threshold AUC > 0.95 was demonstrated on a single proteomics dataset (Olink CVD-II, 77 patients, 91 proteins, FI threshold k = 0.9); other datasets may require different cutoffs.
- Thresholding on AUC alone ignores biomarker panel development cost considerations that muSignAl is designed to help optimise via equally good alternative signatures.
- The AUC values depend on the glmnet-fitted models and pROC computation; re-running the algorithm may yield slightly different AUCs, so the 47/158 count is specific to the provided sign_cvd2.xlsx output.

## Evidence

- [abstract] which selects multiple signatures with similar predictive performance while systematically bypassing the requirement of exploring all the combinations of features: "which selects multiple signatures with similar predictive performance while systematically bypassing the requirement of exploring all the combinations of features"
- [methods] A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder: "A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder"
- [methods] muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed"
- [abstract] discovery and development of biomarker panels while providing the opportunity of optimising their development cost with the help of equally good multiple signatures: "discovery and development of biomarker panels while providing the opportunity of optimising their development cost with the help of equally good multiple signatures"
