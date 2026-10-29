---
name: immunopeptidomics-fdr-filtering
description: Use when PSM or peptide results from immunopeptidomics database searches
  (MS-GF+, Comet, X!Tandem, MaxQuant) must be filtered at a controlled false discovery
  rate before downstream rescoring, RT model training, or peptide-count comparisons.
when_to_use_negative:
- Input is already a high-confidence peptide list that has passed 1% FDR filtering
  — re-filtering adds nothing
- The task requires binding-affinity evaluation, where the peptide-length restriction
  is 8-11 (not 8-25) and MHC binders are defined by a 2% percentile rank score instead
- Data are from enzymatic digestion (tryptic proteome) rather than immunopeptidomics,
  where the inflated unspecific-cleavage search space motivates strict FDR control
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PGA
  role: Implements the target-decoy strategy used to filter peptide identifications
    and control FDR at 1% at both PSM and peptide levels
- name: Percolator
  role: Produces q-value scores on rescoring results that are subsequently filtered
    at q-value <= 0.01
- name: DeepRescore
  role: Pipeline whose outputs (PSM/peptide tables) are filtered at 1% FDR; consumes
    the filtered high-quality peptides for AutoRT fine-tuning
  repo: https://github.com/bzhanglab/DeepRescore
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:d82695e74a26340e79b77c450994a2af70d6c6716395ae6edc6fd3ece09871cf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bzhanglab/DeepRescore
  tools:
  - AutoRT
  - DeepRescore
  - Docker
  - GPTime
  - MS-GF+
  - MaxQuant
  - NextFlow
  - PGA
  - Percolator
  - ProteoWizard
  - X!Tandem
  - pDeep2
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# Immunopeptidomics FDR filtering

## Summary

Applies target-decoy-based false discovery rate control at 1% to peptide-spectrum matches and peptide identifications from immunopeptidomics searches, using q-value <= 0.01 filtering and engine-agreement criteria to define high-confidence peptides.

## When to use

Use when PSM or peptide results from immunopeptidomics database searches (MS-GF+, Comet, X!Tandem, MaxQuant) must be filtered at a controlled false discovery rate before downstream rescoring, RT model training, or peptide-count comparisons.

## When NOT to use

- Input is already a high-confidence peptide list that has passed 1% FDR filtering — re-filtering adds nothing
- The task requires binding-affinity evaluation, where the peptide-length restriction is 8-11 (not 8-25) and MHC binders are defined by a 2% percentile rank score instead
- Data are from enzymatic digestion (tryptic proteome) rather than immunopeptidomics, where the inflated unspecific-cleavage search space motivates strict FDR control

## Inputs

- Search engine PSM/peptide output files with q-value scores (from MS-GF+, Comet, X!Tandem, MaxQuant)
- Target-decoy protein database (e.g., human RefSeq with 245 contaminant sequences appended)
- Peptide identifications from multiple search engines for cross-engine agreement filtering

## Outputs

- Peptide lists filtered at 1% FDR (q-value <= 0.01) at PSM and peptide levels
- High-quality peptide sets for AutoRT transfer-learning training (>=3 of 4 engines passing 1% FDR, or per-engine variants)
- Unique peptide counts per search engine at 1% FDR (e.g., D1: Comet 23400, MS-GF+ 22789, X!Tandem 20727, MaxQuant 17122)

## How to apply

Filter search engine results using the target-decoy strategy implemented in PGA, controlling FDR at 1% at both PSM and peptide levels; operationally this means selecting rows whose q-value column is <= 0.01. When defining training peptides for AutoRT run-specific RT models, apply the stricter cross-engine agreement criterion — peptides passing 1% FDR at both PSM and peptide levels according to at least three of the four search engines (section 3.2 definition) — or, for real-application fine-tuning, 1% FDR per search engine separately (section 3.3 definition). Restrict peptides to lengths between 8 and 25 during identification. The 1% FDR threshold also matches the confidence level used to define high-quality peptides for transfer learning, and the same threshold defines the acceptance criterion when comparing rescoring configurations.

## Related tools

- **PGA** (Implements the target-decoy strategy used to filter peptide identifications and control FDR at 1% at both PSM and peptide levels)
- **Percolator** (Produces q-value scores on rescoring results that are subsequently filtered at q-value <= 0.01)
- **DeepRescore** (Pipeline whose outputs (PSM/peptide tables) are filtered at 1% FDR; consumes the filtered high-quality peptides for AutoRT fine-tuning) — https://github.com/bzhanglab/DeepRescore

## Examples

```
df_filtered = df[df['q-value'] <= 0.01]  # per PGA target-decoy results; apply at both PSM and peptide levels
```

## Evaluation signals

- All retained rows satisfy q-value <= 0.01 at both PSM and peptide levels
- Reported unique peptide counts per engine fall in the paper's ranges (e.g., D1: 17122-23400 across Comet, MS-GF+, X!Tandem, MaxQuant)
- AutoRT training peptide set satisfies the cross-engine criterion: 1% FDR at PSM and peptide levels in at least three of four search engines
- All retained peptides have length between 8 and 25
- Peptide-level FDR is re-estimated after PSM-level filtering to confirm both levels are controlled

## Limitations

- Immunopeptidomics uses unspecific cleavage, inflating the search space and driving higher false positive rates, so FDR thresholds must be controlled at both PSM and peptide levels rather than PSM alone
- The high-quality peptide definition differs by context: at least three of four search engines (Section 3.2) versus each search engine separately (Section 3.3); using the wrong variant changes the training set
- Peptide length is restricted to 8-25 for identification and 8-11 for binding affinity evaluation, which excludes shorter and longer peptides from each respective analysis

## Evidence
