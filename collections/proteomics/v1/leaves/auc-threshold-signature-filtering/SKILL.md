---
name: auc-threshold-signature-filtering
description: Use when a signature-search output (e.g. muSignAl's sign_cvd2.xlsx) lists
  many feature panels with performance scores and you must retain only those above
  a defined AUC threshold and profile feature recurrence.
when_to_use_negative:
- The input is raw proteomics measurement data (e.g. AD.xlsx sample dataset) rather
  than an already-generated signature table from muSignAl
- The goal is to generate the signatures themselves (running muSignAl's search) rather
  than post-filter an existing signature output by AUC
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: muSignAl
  role: Source algorithm whose signature output table is filtered by the AUC threshold
  repo: https://github.com/ShuklaLab/muSignAl
- name: R
  role: Environment in which the AUC filtering and protein-count tallying are performed
- name: glmnet
  role: Prerequisite package muSignAl depends on for the signature models whose AUCs
    are filtered
- name: pROC
  role: Prerequisite package used to compute the AUC values being filtered against
    the cut-off
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202200252
    title: 'muSignAl: An algorithm to search for multiple omic signatures with similar
      predictive performance'
  source_content_hash: sha256:0567cfc423eea93ae49011be05bac091405849666d4f3cd92c647742cb22e4c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShuklaLab/muSignAl
  tools:
  - R
  - glmnet
  - muSignAl
  - pROC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# auc-threshold-signature-filtering

## Summary

Filters the multiple biomarker signatures produced by muSignAl down to those exceeding a fixed predictive-performance cut-off (AUC > 0.95 in the Brunner et al. Olink proteomics example) and tallies how often each protein feature appears across the retained signatures.

## When to use

Use when a signature-search output (e.g. muSignAl's sign_cvd2.xlsx) lists many feature panels with performance scores and you must retain only those above a defined AUC threshold and profile feature recurrence.

## When NOT to use

- The input is raw proteomics measurement data (e.g. AD.xlsx sample dataset) rather than an already-generated signature table from muSignAl
- The goal is to generate the signatures themselves (running muSignAl's search) rather than post-filter an existing signature output by AUC

## Inputs

- muSignAl signature output table (sign_cvd2.xlsx, Excel format) with one row per signature and its constituent proteins
- AUC threshold value (e.g. 0.95) applied as the filtering criterion

## Outputs

- Filtered subset of signatures above the AUC cut-off (47 of 158 in the example)
- Protein-by-count frequency table (e.g. protein_frequency_table.csv) ranking proteins by number of retained signatures they appear in

## How to apply

Load the reproduced muSignAl signature output table (sign_cvd2.xlsx from the package example folder) in R and filter signature rows to retain only panels whose predictive performance exceeds the AUC > 0.95 criterion. Iterate over the retained signatures and tally each protein feature's occurrence count using R vectorized counting such as the table() function (e.g. proteins such as PARP-1, SORT1, IL16, STK4, CCL17, DECR1, IDUA, ADM). Assemble the protein-by-count frequency table and rank proteins by how many of the retained signatures they appear in. Export the result (e.g. protein_frequency_table.csv) and sanity-check that no protein's count exceeds the number of retained signatures. The rationale is that muSignAl deliberately returns many signatures with similar predictive performance, so a fixed AUC cut-off is the decision point separating the top-performing panels (47 of 158 unique signatures in the published example) from the rest.

## Related tools

- **muSignAl** (Source algorithm whose signature output table is filtered by the AUC threshold) — https://github.com/ShuklaLab/muSignAl
- **R** (Environment in which the AUC filtering and protein-count tallying are performed)
- **glmnet** (Prerequisite package muSignAl depends on for the signature models whose AUCs are filtered)
- **pROC** (Prerequisite package used to compute the AUC values being filtered against the cut-off)

## Examples

```
sign <- readxl::read_excel("sign_cvd2.xlsx"); keep <- sign[sign$AUC > 0.95, ]; freq <- table(unlist(keep$proteins)); write.csv(freq, "protein_frequency_table.csv")
```

## Evaluation signals

- Only signatures with AUC strictly above the chosen cut-off (0.95) remain after filtering
- No protein's recurrence count exceeds the number of retained signatures
- The filtered signature count matches the reference (47 of 158 unique signatures in the Brunner et al. Olink example)
- Exported protein_frequency_table.csv rows correspond one-to-one with proteins observed in the retained signatures
- Retained signatures match the peaks above the dotted AUC cut-off line in Figure 2 and the panels listed in Table 1

## Limitations

- The 0.95 AUC cut-off is a threshold choice; the count of retained signatures depends on it and may not transfer to other datasets
- Requires glmnet and pROC to be pre-installed since muSignAl was coded in R v3.6 and depends on them

## Evidence

- [abstract] muSignAl selects multiple signatures with similar predictive performance: "which selects multiple signatures with similar predictive performance while systematically bypassing the requirement of exploring all the combinations of features"
- [methods] Output and sample files in the package example folder: "A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder"
- [results] 47 of 158 signatures exceed the 0.95 AUC cut-off: [evidence span withheld — anchor status: not_found]
- [readme] Dependencies needed to reproduce the AUC values: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed"
