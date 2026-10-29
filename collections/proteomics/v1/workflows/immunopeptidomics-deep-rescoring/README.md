# immunopeptidomics-deep-rescoring-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **search_psms** — search immunopeptidomics spectra against a target-decoy protein database  →  `comet-database-searching`, `mgf-spectra-conversion-and-database-searching`, `cross-search-engine-peptide-overlap-analysis`
2. **select_training_psms** — select high-confidence peptides for run-specific feature models  →  `immunopeptidomics-fdr-filtering`, `psm-target-decoy-confidence-filtering`, `target-decoy-fdr-filtering`, `target-decoy-fdr-estimation`
3. **generate_rt_features** — derive run-specific retention-time error features for every PSM  →  `retention-time-prediction-feature-generation`, `retention-time-prediction-transfer-learning`, `rt-feature-distribution-significance-testing`
4. **generate_spectral_features** — derive predicted-versus-observed fragment-spectrum features for every PSM  →  `spectral-prediction-feature-generation`, `msms-spectrum-prediction-spectral-angle-calculation`, `spectral-angle-similarity-scoring`, `immunopeptidomics-psm-feature-extraction`
5. **rescore_and_control_fdr** — rescore PSMs and emit final PSM and peptide sets at one-percent FDR  →  `psm-rescoring-with-percolator`, `psm-rescoring-with-predicted-spectral-features`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
