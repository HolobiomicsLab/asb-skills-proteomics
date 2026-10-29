# isobaric-reporter-quantification-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **identify_psms** — identify and validate peptide-spectrum matches for isobaric quantification  →  `proteomics-database-search-and-psm-validation`, `proteomics-search-result-conversion`, `comet-database-searching`, `peptide-spectrum-matching-database-search`
2. **quantify_reporter_ions** — extract peptide-level reporter intensities and scan interference scores  →  `isobaric-reporter-ion-quantification`, `isobaric-tmt-peptide-quantification`, `isobaric-quantification-cross-tool-benchmarking`, `precursor-interference-qc-score-calculation`
3. **filter_interference** — remove peptide reporter measurements with unsupported co-isolation scores  →  `isobaric-peptide-qc-score-filtering`, `reporter-ion-interference-assessment`, `tmt-interference-qc-score-filtering`
4. **aggregate_proteins** — roll matched unfiltered and filtered peptide intensities up to proteins  →  `protein-inference-and-quantification-aggregation`, `protein-level-peptide-aggregation`
5. **test_differential_abundance** — compare protein significance before and after interference filtering  →  `differential-abundance-significance-testing`, `intra-protein-cv-precision-analysis`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
