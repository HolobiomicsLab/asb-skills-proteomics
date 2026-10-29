---
name: omic-signature-table-formatting
description: Use when you have a muSignAl output file listing signatures with per-signature
  AUCs and need to extract the high-performing subset (e.g. AUC > 0.95) into a ranked,
  exported table.
when_to_use_negative:
- The input is raw proteomic or other omic expression data (e.g. AD.xlsx) rather than
  a muSignAl final output table — signature generation must happen first with the
  muSignAl package.
- You need to select signatures rather than format/filter an existing signature table
  — use the muSignAl algorithm itself, which bypasses exploring all feature combinations.
- Your signatures come from a tool other than muSignAl whose output lacks the signature-number/protein-set/AUC
  column structure described here.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: muSignAl
  role: Generates the final signature output table (sign_cvd2.xlsx) that this formatting
    skill filters and exports
  repo: https://github.com/ShuklaLab/muSignAl
- name: R
  role: Environment in which the output table is loaded, filtered, sorted, and exported
    (coded for R v3.6)
  repo: https://github.com/ShuklaLab/muSignAl
- name: pROC
  role: Provides the AUC values attached to each signature in the output table and
    used as the > 0.95 filtering threshold
  repo: https://github.com/ShuklaLab/muSignAl
- name: glmnet
  role: Required prerequisite package whose fitted models underlie the per-signature
    AUCs in the output
  repo: https://github.com/ShuklaLab/muSignAl
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202200252
    title: 'muSignAl: An algorithm to search for multiple omic signatures with similar
      predictive performance'
  source_content_hash: sha256:3faa2cf6e9be3d3cdc8cb192f3a279ccfdd03d15d6e9b11c7c9f585f61899c28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShuklaLab/muSignAl
  tools:
  - R
  - glmnet
  - muSignAl
  - pROC
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# omic-signature-table-formatting

## Summary

Post-processing of the muSignAl final output table (e.g. sign_cvd2.xlsx) to filter protein signatures by predictive performance threshold (AUC > 0.95), sort them, and export a publication-style table such as Table 1 of the muSignAl paper (47 of 158 signatures).

## When to use

Use when you have a muSignAl output file listing signatures with per-signature AUCs and need to extract the high-performing subset (e.g. AUC > 0.95) into a ranked, exported table.

## When NOT to use

- The input is raw proteomic or other omic expression data (e.g. AD.xlsx) rather than a muSignAl final output table — signature generation must happen first with the muSignAl package.
- You need to select signatures rather than format/filter an existing signature table — use the muSignAl algorithm itself, which bypasses exploring all feature combinations.
- Your signatures come from a tool other than muSignAl whose output lacks the signature-number/protein-set/AUC column structure described here.

## Inputs

- muSignAl final output spreadsheet (sign_cvd2.xlsx) with per-signature columns: signature number, constituent protein set, AUC (pROC AUC from glmnet-fitted models)

## Outputs

- Filtered and descending-AUC-sorted signature table (signature number, protein set, AUC) written as an .xlsx file (e.g. filtered_signatures_auc_gt_095.xlsx) reproducing Table 1

## How to apply

Load the muSignAl final output file (sign_cvd2.xlsx from the package's example folder) into R with readxl/openxlsx. Parse the per-signature columns: signature number, constituent protein set, and AUC as computed with pROC from the glmnet-fitted models. Filter the signature rows to those with AUC > 0.95, then sort by descending AUC. Write the result as a table with columns (signature number, protein set, AUC) to a spreadsheet such as filtered_signatures_auc_gt_095.xlsx. On the Olink CVD-II proteomics dataset (77 patients, 91 proteins, FI threshold k = 0.9) this reproduces Table 1: 47 of 158 unique signatures, topped by signature 150 (CCL17/DECR1/ADM/PARP-1/SORT1) at AUC 1.00.

## Related tools

- **muSignAl** (Generates the final signature output table (sign_cvd2.xlsx) that this formatting skill filters and exports) — https://github.com/ShuklaLab/muSignAl
- **R** (Environment in which the output table is loaded, filtered, sorted, and exported (coded for R v3.6)) — https://github.com/ShuklaLab/muSignAl
- **pROC** (Provides the AUC values attached to each signature in the output table and used as the > 0.95 filtering threshold) — https://github.com/ShuklaLab/muSignAl
- **glmnet** (Required prerequisite package whose fitted models underlie the per-signature AUCs in the output) — https://github.com/ShuklaLab/muSignAl

## Examples

```
sig <- readxl::read_excel("sign_cvd2.xlsx"); top <- sig[sig$AUC > 0.95, ]; top <- top[order(-top$AUC), ]; writexl::write_xlsx(top, "filtered_signatures_auc_gt_095.xlsx")
```

## Evaluation signals

- The filtered set contains exactly the rows with AUC > 0.95 (on the CVD-II dataset, 47 of 158 unique signatures).
- The top row is signature 150 (CCL17, DECR1, ADM, PARP-1, SORT1) with AUC = 1.00.
- Rows are sorted by descending AUC and the table carries the columns signature number, protein set, and AUC.
- The exported file (e.g. filtered_signatures_auc_gt_095.xlsx) reproduces Table 1 of the article.
- The source file matches the muSignAl final output format (sign_cvd2.xlsx) rather than the raw input dataset (AD.xlsx).

## Limitations

- The AUC threshold (> 0.95) and resulting count (47/158) are specific to the Olink CVD-II dataset with FI threshold k = 0.9; other datasets will yield different filtered sets.
- The AUC values must have been computed with pROC from glmnet-fitted models as produced by muSignAl; the skill does not itself recompute performance.
- Requires R v3.6-compatible tooling and pre-installed glmnet and pROC per the package requirements.

## Evidence

- [methods] muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed"
- [methods] A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder: "A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder"
- [results] 47 of the 158 unique signatures had AUC greater than 0.95, reported in Table 1 (with the top signature 150, CCL17/DECR1/ADM/PARP-1/SORT1, reaching AUC 1.00).: [evidence span withheld — anchor status: not_found]
- [readme] A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder.: "A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder."
