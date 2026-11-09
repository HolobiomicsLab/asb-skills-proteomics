---
name: multiple-omic-signature-auc-comparison
description: Use when a labeled omics dataset (e.g. 77 samples x 91 Olink protein
  features) needs multiple equally predictive feature signatures enumerated and ranked
  by AUC, rather than a single model.
when_to_use_negative:
- You need a single optimal biomarker panel with unique features rather than multiple
  signatures of similar predictive performance
- The feature space is small enough that exhaustive enumeration of all feature combinations
  is feasible and preferable
- Input is not a labeled feature table (e.g. unlabeled proteomics matrices without
  case/control status), since AUC scoring with pROC requires class labels
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: muSignAl
  role: Core R package that enumerates multiple candidate signatures with similar
    predictive performance using the k = 0.9 feature-importance threshold
  repo: https://github.com/ShuklaLab/muSignAl
- name: glmnet
  role: Fits Elastic Net (alpha = 0.9) regularized models used to rank protein features
    by importance
- name: pROC
  role: Computes per-signature AUC values used to compare the returned signatures
- name: devtools
  role: Installs muSignAl from the ShuklaLab GitHub repository
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202200252
    title: 'muSignAl: An algorithm to search for multiple omic signatures with similar
      predictive performance'
  source_content_hash: sha256:fa937a07b20ad022354315ccb66889b2eca37caa36e92d5b69483784544128e5
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# multiple-omic-signature-auc-comparison

## Summary

Reproduce the muSignAl workflow that enumerates multiple candidate biomarker signatures from omics data and scores each with pROC-derived AUC, so that several signatures of similar predictive performance can be compared without exploring all feature combinations.

## When to use

Use when a labeled omics dataset (e.g. 77 samples x 91 Olink protein features) needs multiple equally predictive feature signatures enumerated and ranked by AUC, rather than a single best model.

## When NOT to use

- You need a single optimal biomarker panel with unique features rather than multiple signatures of similar predictive performance
- The feature space is small enough that exhaustive enumeration of all feature combinations is feasible and preferable
- Input is not a labeled feature table (e.g. unlabeled proteomics matrices without case/control status), since AUC scoring with pROC requires class labels

## Inputs

- Labeled omics feature table in xlsx format (e.g. AD.xlsx: Brunner et al. Olink CVD-II proteomics, 77 samples x 91 protein features)

## Outputs

- Signature set with per-signature AUC values (e.g. sign_cvd2.xlsx)
- Count of total and unique signatures generated (e.g. 1984 signatures, 158 unique)
- Ranked list of signatures meeting an AUC cutoff (e.g. 47 unique signatures with AUC > 0.95)

## How to apply

Install the prerequisites glmnet and pROC under R v3.6, then install muSignAl from GitHub with devtools::install_github("ShuklaLab/muSignAl"). Load a labeled feature table such as the example AD.xlsx (Brunner et al. Olink CVD-II proteomics, 77 samples x 91 protein features) and run muSignAl() with a feature-importance threshold k = 0.9 and Elastic Net alpha = 0.9, letting glmnet fit regularized models and rank protein features by importance. The k = 0.9 importance cutoff enumerates many candidate signatures while bypassing exhaustive exploration of all feature combinations. Score every returned signature with pROC to obtain per-signature AUC values, and export the signature set with its AUCs (e.g. as sign_cvd2.xlsx). Validate by comparing against the reference output in the repository's example folder; in the Brunner CVD-II run, 1984 signatures (158 unique) were generated and 47 unique signatures achieved AUC > 0.95.

## Related tools

- **muSignAl** (Core R package that enumerates multiple candidate signatures with similar predictive performance using the k = 0.9 feature-importance threshold) — https://github.com/ShuklaLab/muSignAl
- **glmnet** (Fits Elastic Net (alpha = 0.9) regularized models used to rank protein features by importance)
- **pROC** (Computes per-signature AUC values used to compare the returned signatures)
- **devtools** (Installs muSignAl from the ShuklaLab GitHub repository)

## Examples

```
install.packages(c("glmnet","pROC")); install.packages("devtools"); devtools::install_github("ShuklaLab/muSignAl"); Rscript test.R
```

## Evaluation signals

- Exported signature file matches the reference output sign_cvd2.xlsx in the repository's example folder
- Signature count falls in the expected range for the Brunner CVD-II run: 1984 signatures generated, 158 unique
- Number of unique signatures with AUC greater than 0.95 matches the reported value of 47
- Per-signature AUC values are computed by pROC for every returned signature, not a subset
- Feature-importance cutoff k = 0.9 and Elastic Net alpha = 0.9 are recorded as the run parameters

## Limitations

- muSignAl was coded in R v3.6 and requires glmnet and pROC to be pre-installed; other R versions are not attested
- The number of signatures grows with the FI threshold choice; k = 0.9 on the example data yields 1984 (158 unique) signatures, so results depend on cutoff selection
- AUCs above 0.95 do not guarantee unique biological insight — the algorithm deliberately returns multiple signatures with similar predictive performance

## Evidence

- [abstract] muSignAl: An algorithm to search for multiple omic signatures with similar predictive performance: "muSignAl: An algorithm to search for multiple omic signatures with similar predictive performance"
- [methods] requires glmnet and pROC: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed"
- [methods] installation via devtools: "devtools::install_github("ShuklaLab/muSignAl")"
- [methods] example test script and dataset: "A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder"
- [readme] README requirements statement: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed, which can be installed as below:"
