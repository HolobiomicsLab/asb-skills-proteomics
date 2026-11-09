---
name: biomarker-panel-prioritization
description: Use when a muSignAl run yields many candidate biomarker signatures (e.g.
  158 panels from Olink CVD-II in sign_cvd2.xlsx) and you must rank or threshold them
  by AUC to shortlist panels worth developing.
when_to_use_negative:
- The candidate signatures still need to be generated from the raw omics matrix —
  that is the upstream muSignAl run (test.R on the input .xlsx); this skill only prioritizes
  an existing final output like sign_cvd2.xlsx
- Only one candidate panel is under consideration — ranking by AUC adds no value without
  multiple alternative signatures
- The output has no per-signature performance metric (the AUC column computed with
  pROC is absent) — there is nothing to threshold or rank on
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: muSignAl
  role: Generates the multiple candidate signatures whose AUC-ranked final output
    (sign_cvd2.xlsx) is the object of prioritization
  repo: https://github.com/ShuklaLab/muSignAl
- name: R
  role: Runtime (R v3.6) in which the output parsing, AUC > 0.95 filtering and descending
    sort are scripted
- name: glmnet
  role: Fits the predictive models underlying each signature; the models' discriminative
    performance determines the ranking metric
- name: pROC
  role: Computes the per-signature AUC used as the prioritization threshold (>0.95)
    and the sort key
- name: devtools
  role: Installs the muSignAl package from GitHub before the run (devtools::install_github("ShuklaLab/muSignAl"))
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202200252
    title: 'muSignAl: An algorithm to search for multiple omic signatures with similar
      predictive performance'
  source_content_hash: sha256:a4c6a40382c634f7a866eb154afd32877c91662d967ce2129e915bcef8248262
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShuklaLab/muSignAl
  tools:
  - R
  - devtools
  - glmnet
  - muSignAl
  - pROC
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# biomarker-panel-prioritization

## Summary

Prioritizes candidate biomarker panels by ranking muSignAl-generated omic signatures on their predictive performance (AUC computed with pROC from glmnet-fitted models), e.g. isolating the 47 of 158 CVD-II proteomics signatures with AUC > 0.95. Because equally good multiple signatures let biomarker panel development cost be optimised, the ranked shortlist (Table 1, led by signature 150 at AUC 1.00) is the actionable artifact.

## When to use

Use when a muSignAl run yields many candidate biomarker signatures (e.g. 158 panels from Olink CVD-II in sign_cvd2.xlsx) and you must rank or threshold them by AUC to shortlist panels worth developing.

## When NOT to use

- The candidate signatures still need to be generated from the raw omics matrix — that is the upstream muSignAl run (test.R on the input .xlsx); this skill only prioritizes an existing final output like sign_cvd2.xlsx
- Only one candidate panel is under consideration — ranking by AUC adds no value without multiple alternative signatures
- The output has no per-signature performance metric (the AUC column computed with pROC is absent) — there is nothing to threshold or rank on

## Inputs

- Proteomics expression matrix as .xlsx (e.g. the Olink CVD-II input of 77 patients × 91 proteins, such as AD.xlsx in the example folder)
- muSignAl final output sign_cvd2.xlsx with per-signature columns: signature number, protein set, AUC

## Outputs

- AUC-filtered, descending-sorted signature table (signature number, protein set, AUC), e.g. filtered_signatures_auc_gt_095.xlsx
- Shortlist of 47 biomarker panels with AUC > 0.95 reproducing Table 1, led by signature 150 (CCL17/DECR1/ADM/PARP-1/SORT1, AUC 1.00)

## How to apply

Run muSignAl in R v3.6 with glmnet and pROC pre-installed on the omics input — the demonstration used an Olink CVD-II proteomics matrix of 77 patients × 91 proteins with FI threshold k = 0.9 — to enumerate multiple signatures with similar predictive performance. Load the final output sign_cvd2.xlsx from the package's example folder into R using readxl/openxlsx and parse the per-signature columns: signature number, constituent protein set, and the AUC computed with pROC from the glmnet-fitted models. Filter the signature rows to those with AUC > 0.95 (47 of 158 unique signatures in the CVD-II case) and sort by descending AUC. Write the ranked table (signature number, protein set, AUC) to filtered_signatures_auc_gt_095.xlsx. Validate that the top row is signature 150 (CCL17/DECR1/ADM/PARP-1/SORT1) at AUC = 1.00 and that the filtered rows reproduce Table 1 of the article — this shortlist is the cost-optimisation deliverable the paper motivates.

## Related tools

- **muSignAl** (Generates the multiple candidate signatures whose AUC-ranked final output (sign_cvd2.xlsx) is the object of prioritization) — https://github.com/ShuklaLab/muSignAl
- **R** (Runtime (R v3.6) in which the output parsing, AUC > 0.95 filtering and descending sort are scripted)
- **glmnet** (Fits the predictive models underlying each signature; the models' discriminative performance determines the ranking metric)
- **pROC** (Computes the per-signature AUC used as the prioritization threshold (>0.95) and the sort key)
- **devtools** (Installs the muSignAl package from GitHub before the run (devtools::install_github("ShuklaLab/muSignAl")))

## Examples

```
f <- subset(readxl::read_excel("sign_cvd2.xlsx"), AUC > 0.95); f[order(-f$AUC), ]
```

## Evaluation signals

- On sign_cvd2.xlsx with threshold AUC > 0.95, the filtered table contains exactly 47 rows out of the 158 unique signatures
- The top-ranked row is signature 150 with protein set CCL17/DECR1/ADM/PARP-1/SORT1 at AUC = 1.00
- Every output row satisfies AUC > 0.95 and the AUC column is monotonically non-increasing (descending sort)
- Row-by-row comparison of the output table against Table 1 of the article matches on signature number, protein set and AUC
- Output file filtered_signatures_auc_gt_095.xlsx carries the three expected columns: signature number, protein set, AUC

## Limitations

- Counts are dataset-specific: 158 unique signatures and 47 with AUC > 0.95 come from the Olink CVD-II example (77 patients, 91 proteins, FI threshold k = 0.9) and must be re-derived on other data
- muSignAl was coded in R v3.6 and requires glmnet and pROC pre-installed; sign_cvd2.xlsx in the example folder is the reference output for validating the pipeline
- The README documents only installation and a sample run — the AUC > 0.95 prioritization threshold is an analysis-layer choice, not a package default

## Evidence

- [abstract] discovery and development of biomarker panels while providing the opportunity of optimising their development cost with the help of equally good multiple signatures: "discovery and development of biomarker panels while providing the opportunity of optimising their development cost with the help of equally good multiple signatures"
- [abstract] we have developed an algorithm muSignAl (multiple signature algorithm) which selects multiple signatures with similar predictive performance: "we have developed an algorithm muSignAl (multiple signature algorithm) which selects multiple signatures with similar predictive performance"
- [methods] muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed"
- [readme] A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder.: "A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder."
- [other] 47 of the 158 unique signatures had AUC greater than 0.95, reported in Table 1 (with the top signature 150, CCL17/DECR1/ADM/PARP-1/SORT1, reaching AUC 1.00).: [evidence span withheld — anchor status: not_found]
