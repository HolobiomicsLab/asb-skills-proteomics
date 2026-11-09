---
name: cnv-alteration-frequency-analysis
description: Use when a task asks for pancancer CNV alteration frequency of a specific
  gene across cancer types in patient-derived cancer cell lines, or to replicate the
  IMOPAC 'Cancer Types Summary' CNV module from DepMap CNV data.
when_to_use_negative:
- The analysis goal is continuous-omics (e.g. mRNA) pancancer comparison — IMOPAC
  uses a boxplot for continuous settings in 'Cancer Types Summary', not a discrete
  alteration-frequency histogram.
- The task concerns drug-omics/pathway correlation, fusion query, cancer subtyping,
  or drug response prediction rather than gene-level CNV frequency summarization.
- The dataset is GDSC (992 cell lines, 28 cancer types) or NCI60 (60 cell lines, 9
  cancer types) where the reported CTNNB1 CNV frequency result does not apply.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3053
tools:
- name: IMOPAC
  role: Web server whose 'Cancer Types Summary' pancancer CNV module is being reproduced;
    source code available in the IMOPAC repository
  repo: https://github.com/wolfgangsk07/IMOPAC
- name: DepMap
  role: Source of the CNV omics dataset (1773 cell lines from 34 cancer types) over
    which alteration frequencies are computed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.023
    title: 'IMOPAC: A web server for interactive multiomics and pharmacological analyses
      of patient-derived cancer cell lines'
  source_content_hash: sha256:42a4ac44313a115c4acdb9d4d09facf01c99fe80a50b109bbaa05932cdb9af92
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wolfgangsk07/IMOPAC
  repo_url: https://github.com/wolfgangsk07/IMOPAC
  tools:
  - DepMap
  - IMOPAC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cnv-alteration-frequency-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the IMOPAC pancancer 'Cancer Types Summary' CNV display by computing, for a single input gene (e.g. CTNNB1), the fraction of cell lines carrying a CNV alteration within each cancer type of the DepMap CNV omics dataset (1773 cell lines, 34 cancer types), ranking cancer types by frequency, and plotting the per-cancer-type histogram.

## When to use

Use when a task asks for pancancer CNV alteration frequency of a specific gene across cancer types in patient-derived cancer cell lines, or to replicate the IMOPAC 'Cancer Types Summary' CNV module from DepMap CNV data.

## When NOT to use

- The analysis goal is continuous-omics (e.g. mRNA) pancancer comparison — IMOPAC uses a boxplot for continuous settings in 'Cancer Types Summary', not a discrete alteration-frequency histogram.
- The task concerns drug-omics/pathway correlation, fusion query, cancer subtyping, or drug response prediction rather than gene-level CNV frequency summarization.
- The dataset is GDSC (992 cell lines, 28 cancer types) or NCI60 (60 cell lines, 9 cancer types) where the reported CTNNB1 CNV frequency result does not apply.

## Inputs

- DepMap CNV omics profile for a single gene (discrete CNV alteration calls) covering 1773 cell lines from 34 cancer types
- Cell-line to cancer-type annotation mapping for the DepMap dataset

## Outputs

- Ranked per-cancer-type CNV alteration frequency table (e.g. ctnnb1_cnv_pancancer_frequency.csv)
- Histogram of CNV alteration frequency per cancer type replicating the IMOPAC 'Cancer Types Summary' pancancer CNV display

## How to apply

Download the DepMap CNV profile for the target gene (IMOPAC's DepMap database covers 1773 cell lines from 34 cancer types, with 12 omics including CNV). Treat CNV as a discrete omics setting: for each cancer type, compute alteration frequency as the number of cell lines carrying any CNV alteration divided by the total number of cell lines of that cancer type. Rank the cancer types in descending order of frequency and write the per-cancer-type table to a CSV file (e.g. ctnnb1_cnv_pancancer_frequency.csv). Plot a histogram of alteration frequency per cancer type to replicate the IMOPAC 'Cancer Types Summary' pancancer CNV figure for the input gene. Validate by confirming the top-ranked cancer types match the reported result that CTNNB1 CNV alteration frequencies are highest in head and neck carcinoma, kidney carcinoma, and oral cavity carcinoma.

## Related tools

- **IMOPAC** (Web server whose 'Cancer Types Summary' pancancer CNV module is being reproduced; source code available in the IMOPAC repository) — https://github.com/wolfgangsk07/IMOPAC
- **DepMap** (Source of the CNV omics dataset (1773 cell lines from 34 cancer types) over which alteration frequencies are computed)

## Examples

```
Rscript -e 'cnv<-read.csv("ctnnb1_cnv.csv"); f<-aggregate(altered~cancer_type,cnv,mean); f<-f[order(-f$altered),]; write.csv(f,"ctnnb1_cnv_pancancer_frequency.csv",row.names=FALSE)'
```

## Evaluation signals

- Top-ranked cancer types by CTNNB1 CNV alteration frequency match the reported result: head and neck carcinoma, kidney carcinoma, and oral cavity carcinoma.
- The output CSV contains one row per cancer type (34 cancer types for the DepMap dataset) with frequencies bounded in [0, 1].
- Per-cancer-type frequencies are reproducible as the count of cell lines with any CNV alteration divided by total cell lines of that cancer type.
- The histogram reproduces the visual layout of the IMOPAC 'Cancer Types Summary' pancancer CNV display (Fig. 2A style).
- Ranking is strictly in descending order of alteration frequency.

## Limitations

- The reference finding is demonstrated for CTNNB1 in the DepMap database; results for other genes or other databases (GDSC with 992 cell lines/28 cancer types, NCI60 with 60 cell lines/9 cancer types) will differ.
- IMOPAC is a web application without a stated software version number, and the GitHub repository cannot host the large *.RData datasets — they must be downloaded separately via urls.txt.
- Treating CNV as a discrete omics setting discards amplitude information; only presence/absence of alteration per cell line is used.

## Evidence

- [results] CTNNB1 CNV alteration frequencies are the highest in head and neck carcinoma, kidney carcinoma, and oral cavity carcinoma: [evidence span withheld — anchor status: not_found]
