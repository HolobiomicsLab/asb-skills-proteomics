---
name: proteomics-signature-table-parsing
description: Use when you have a muSignAl signature output table (xlsx with one row
  per candidate signature and its constituent proteins plus AUC) and need to filter
  by an AUC threshold and compute per-protein recurrence frequency across retained
  signature panels.
when_to_use_negative:
- Input is already a filtered feature-frequency table rather than a per-signature
  panel table with AUC columns
- The signature output has no predictive-performance (AUC) column, so the AUC > 0.95
  filtering step cannot be applied
- You need to generate the signatures themselves rather than parse and summarize an
  existing muSignAl output table
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: muSignAl
  role: R package whose example output (sign_cvd2.xlsx) is the signature table being
    parsed and filtered; installed from GitHub via devtools
  repo: https://github.com/ShuklaLab/muSignAl
- name: R
  role: Environment for loading the xlsx signature table, applying the AUC > 0.95
    filter, and counting protein occurrences with vectorized counting (table())
- name: devtools
  role: Used to install the muSignAl package whose output feeds this parsing skill
  repo: https://github.com/ShuklaLab/muSignAl
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202200252
    title: 'muSignAl: An algorithm to search for multiple omic signatures with similar
      predictive performance'
  source_content_hash: sha256:583610f972d431777b7d004e204ca9d63518b5579864a53dbd2f1e891cf993ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShuklaLab/muSignAl
  tools:
  - R
  - devtools
  - muSignAl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# proteomics-signature-table-parsing

## Summary

Parse the muSignAl signature output table (sign_cvd2.xlsx) produced from an Olink proteomics dataset, filter signature rows to panels exceeding an AUC > 0.95 cut-off, and tally protein-feature occurrence counts across the retained signatures (e.g., the 47 high-AUC signatures of 158 unique signatures on the Brunner et al. dataset).

## When to use

Use when you have a muSignAl signature output table (xlsx with one row per candidate signature and its constituent proteins plus AUC) and need to filter by an AUC threshold and compute per-protein recurrence frequency across retained signature panels.

## When NOT to use

- Input is already a filtered feature-frequency table rather than a per-signature panel table with AUC columns
- The signature output has no predictive-performance (AUC) column, so the AUC > 0.95 filtering step cannot be applied
- You need to generate the signatures themselves rather than parse and summarize an existing muSignAl output table

## Inputs

- muSignAl signature output table (xlsx, e.g. sign_cvd2.xlsx) with one row per unique signature, its constituent protein features, and AUC performance values
- AUC cut-off criterion (e.g. 0.95) defining the retained signature set

## Outputs

- Filtered table of signatures exceeding the AUC cut-off (e.g. 47 signatures)
- Protein-by-count frequency table (proteins ranked by number of signatures they appear in)
- CSV export (e.g. protein_frequency_table.csv)

## How to apply

Load the reproduced muSignAl signature output (sign_cvd2.xlsx from the package example folder) into R via a spreadsheet reader. Filter signature rows to retain only panels whose predictive performance exceeds the AUC > 0.95 criterion (in the Brunner et al. Olink example this retains 47 of 158 unique signatures). Iterate over the retained signatures and tally the occurrence count of each protein feature (e.g., PARP-1, SORT1, IL16, STK4, CCL17, DECR1, IDUA, ADM) using R vectorized counting such as the table() function. Assemble the protein-by-count frequency table and rank proteins by how many of the retained signatures they appear in. Export as protein_frequency_table.csv; note that the underlying muSignAl algorithm was designed to bypass exhaustive exploration of all feature combinations, so the output table is already a compact set of comparable signatures rather than a full combinatorial enumeration.

## Related tools

- **muSignAl** (R package whose example output (sign_cvd2.xlsx) is the signature table being parsed and filtered; installed from GitHub via devtools) — https://github.com/ShuklaLab/muSignAl
- **R** (Environment for loading the xlsx signature table, applying the AUC > 0.95 filter, and counting protein occurrences with vectorized counting (table()))
- **devtools** (Used to install the muSignAl package whose output feeds this parsing skill) — https://github.com/ShuklaLab/muSignAl

## Examples

```
devtools::install_github("ShuklaLab/muSignAl"); sign <- readxl::read_xlsx("example/sign_cvd2.xlsx"); freq <- sort(table(unlist(strsplit(sign$proteins[sign$AUC > 0.95], ","))), decreasing = TRUE)
```

## Evaluation signals

- The number of retained signatures matches the expected subset of the total unique signatures (e.g. 47 retained of 158 on the Brunner et al. Olink dataset)
- No protein's occurrence count exceeds the number of retained signatures (an invariant: each protein can appear at most once per signature)
- Every protein listed in the frequency table (e.g. PARP-1, SORT1, IL16, STK4, CCL17, DECR1, IDUA, ADM) traces back to at least one retained AUC > 0.95 signature row
- Exported protein_frequency_table.csv parses cleanly and contains a count column ranking proteins by signature membership
- Re-running the filter on the same sign_cvd2.xlsx reproduces the same retained-signature set (deterministic AUC cut-off)

## Limitations

- The filtering and counting are only meaningful on muSignAl-style signature output tables; muSignAl was coded in R v3.6 and requires glmnet and pROC, so version or package mismatches can affect reproduction of the signature output itself
- The skill depends on the completeness of the input signature table — it does not regenerate signatures or explore feature combinations, which muSignAl deliberately bypasses

## Evidence

- [results] 47 of the 158 unique signatures had AUC above the 0.95 cut-off: [evidence span withheld — anchor status: not_found]
- [readme] A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder: "A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder"
- [methods] muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed"
- [abstract] which selects multiple signatures with similar predictive performance while systematically bypassing the requirement of exploring all the combinations of features: "which selects multiple signatures with similar predictive performance while systematically bypassing the requirement of exploring all the combinations of features"
