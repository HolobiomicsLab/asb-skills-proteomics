---
name: protein-frequency-counting-across-signature-panels
description: Use when you have a multi-signature biomarker output (e.g., muSignAl's
  sign_cvd2.xlsx) and need to identify which constituent proteins appear most often
  across panels passing a performance cut-off such as AUC > 0.95.
when_to_use_negative:
- Input is raw proteomics intensity data (e.g., AD.xlsx-style Olink matrix) that has
  not yet been processed by muSignAl to produce candidate signatures
- The goal is to compare signature-level predictive performance (AUC) rather than
  protein recurrence across panels
- Signatures come from a single-panel biomarker method that outputs one model, not
  multiple equally performing signatures
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: R
  role: Environment in which the signature table is loaded, filtered by AUC > 0.95,
    and protein occurrences are tallied with vectorized counting (table()); muSignAl
    was coded in R v3.6
  repo: https://github.com/ShuklaLab/muSignAl
- name: muSignAl
  role: Generates the multiple omic signatures (and the sign_cvd2.xlsx example output)
    whose constituent proteins are counted across high-AUC panels
  repo: https://github.com/ShuklaLab/muSignAl
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202200252
    title: 'muSignAl: An algorithm to search for multiple omic signatures with similar
      predictive performance'
  source_content_hash: sha256:ee448506c8913fdd585bc2ace8ad46d8d0fc4ecee8f79657fb51d5069359f6c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShuklaLab/muSignAl
  tools:
  - R
  - muSignAl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# protein-frequency-counting-across-signature-panels

## Summary

Tallies how often each protein feature recurs across the set of high-performing biomarker signatures produced by muSignAl (here, the 47 of 158 signatures exceeding AUC > 0.95 on the Brunner et al. Olink proteomics dataset), yielding a ranked protein-by-count frequency table.

## When to use

Use when you have a multi-signature biomarker output (e.g., muSignAl's sign_cvd2.xlsx) and need to identify which constituent proteins appear most often across panels passing a performance cut-off such as AUC > 0.95.

## When NOT to use

- Input is raw proteomics intensity data (e.g., AD.xlsx-style Olink matrix) that has not yet been processed by muSignAl to produce candidate signatures
- The goal is to compare signature-level predictive performance (AUC) rather than protein recurrence across panels
- Signatures come from a single-panel biomarker method that outputs one model, not multiple equally performing signatures

## Inputs

- muSignAl signature output table (e.g., sign_cvd2.xlsx from the package example folder)
- AUC performance values per signature
- Constituent protein lists per signature

## Outputs

- protein_frequency_table.csv — protein-by-count frequency table ranked by recurrence across the AUC > 0.95 signatures

## How to apply

Load the reproduced muSignAl signature output table (e.g., sign_cvd2.xlsx from the package's example folder) into R. Filter signature rows to retain only panels whose predictive performance exceeds the AUC > 0.95 criterion — in the Brunner et al. Olink case this retains 47 of the 158 unique signatures. Iterate over the retained signatures and tally each protein feature (e.g., PARP-1, SORT1, IL16, STK4, CCL17, DECR1, IDUA, ADM) using R vectorized counting such as the table() function. Assemble the protein-by-count frequency table and rank proteins by the number of the 47 signatures in which they appear. Export as protein_frequency_table.csv, which supports muSignAl's stated goal of optimising biomarker panel development cost by revealing redundant, equally good signatures.

## Related tools

- **R** (Environment in which the signature table is loaded, filtered by AUC > 0.95, and protein occurrences are tallied with vectorized counting (table()); muSignAl was coded in R v3.6) — https://github.com/ShuklaLab/muSignAl
- **muSignAl** (Generates the multiple omic signatures (and the sign_cvd2.xlsx example output) whose constituent proteins are counted across high-AUC panels) — https://github.com/ShuklaLab/muSignAl

## Examples

```
install.packages(c("glmnet","pROC")); devtools::install_github("ShuklaLab/muSignAl")
```

## Evaluation signals

- No protein's recurrence count exceeds the number of retained signatures (47 in the Brunner et al. Olink case)
- The set of tallied proteins equals the union of constituent proteins across the AUC > 0.95 panels (e.g., PARP-1, SORT1, IL16, STK4, CCL17, DECR1, IDUA, ADM)
- Retained signature count matches the expected cut-off result (47 of 158 unique signatures above AUC 0.95, as in Figure 2/Table 1)
- Exported CSV parses cleanly and counts sum consistently with the per-signature protein list lengths
- Ranking is monotonic — proteins are ordered strictly by descending signature-membership count

## Limitations

- Counting is only meaningful after the AUC > 0.95 (or other) cut-off is applied; applying it to all 158 signatures dilutes recurrence signal with low-performing panels
- muSignAl output depends on pre-installed glmnet and pROC and the R version used (v3.6 in the paper); count inputs may differ across package versions or dataset choices
- Protein name spelling/synonyms in the signature output must be harmonized or the table() counts will fragment identical proteins

## Evidence

- [abstract] abstract: "we have developed an algorithm muSignAl (multiple signature algorithm) which selects multiple signatures with similar predictive performance"
- [methods] methods: "A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder"
- [readme] readme: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed"
