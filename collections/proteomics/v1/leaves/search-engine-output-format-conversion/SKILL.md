---
name: search-engine-output-format-conversion
description: Use when raw search engine results (e.g., MaxQuant msms.txt/txt output
  from PRIDE deposits like PXD010871) need to be reformatted for a downstream rescoring
  pipeline that requires a standardized internal schema and ProForma-compliant peptide
  sequences.
when_to_use_negative:
- The search results are already in Oktoberfest's internal format or ProForma notation
  (re-conversion would risk corrupting modification labels)
- The goal is DIA spectral library generation from FASTA digestion (use pp.digest/pp.gen_lib
  instead of search-result conversion)
- The search output is from a search engine not supported by pp.convert_search (conversion
  would silently mis-map columns)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Oktoberfest
  role: Provides pp.convert_search, pp.load_search, and pp.split_search to translate
    search engine results into the internal rescoring format
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: spectrum_fundamentals
  role: Performs conversion between search engine-specific modstrings and the ProForma
    standard used internally
  repo: https://github.com/wilhelm-lab/spectrum_fundamentals
- name: spectrum_io
  role: File/data conversion for mass spec data within the Oktoberfest ecosystem (complementary
    spectra conversion)
  repo: https://github.com/wilhelm-lab/spectrum_io
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:a96769db3737bbc70ca00de865aefe60591787b517ac35763ba7ec0b2369c601
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Oktoberfest
  - Python
  - mokapot
  - spectrum_fundamentals
  - spectrum_io
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# search-engine-output-format-conversion

## Summary

Convert native search engine outputs (e.g., unfiltered MaxQuant/Andromeda results) into Oktoberfest's internal format, including translating search engine-specific modification strings (modstrings) into the ProForma standard via spectrum_fundamentals, so that PSMs can be used downstream for Prosit-based rescoring.

## When to use

Use when raw search engine results (e.g., MaxQuant msms.txt/txt output from PRIDE deposits like PXD010871) need to be reformatted for a downstream rescoring pipeline that requires a standardized internal schema and ProForma-compliant peptide sequences.

## When NOT to use

- The search results are already in Oktoberfest's internal format or ProForma notation (re-conversion would risk corrupting modification labels)
- The goal is DIA spectral library generation from FASTA digestion (use pp.digest/pp.gen_lib instead of search-result conversion)
- The search output is from a search engine not supported by pp.convert_search (conversion would silently mis-map columns)

## Inputs

- Unfiltered MaxQuant/Andromeda search results (e.g., from PRIDE accession PXD010871, downloaded as txt/msms tables)
- Search results from multiple search spaces (Swiss-Prot Human, Swiss-Prot Human + Bacteria, Swiss-Prot Human + IGC)
- Search engine-specific modification strings (modstrings)

## Outputs

- Oktoberfest internal-format search result tables (per search space)
- Split search tables (pp.split_search output) keyed by scan for downstream feature generation
- ProForma-standardized peptide/modification sequences

## How to apply

Load the search engine's unfiltered results with oktoberfest.pp.load_search after converting them with pp.convert_search, which maps search engine-specific columns (e.g., MaxQuant/Andromeda scores, sequence, modifications) onto the internal format. Use pp.split_search to partition the converted search into the per-scan PSM tables needed for feature generation. Modification strings are handled by spectrum_fundamentals, which provides 'conversion between search engine-specific modstrings and the ProForma standard' and 'calculation of theoretical peptide / ion masses'. In the metaproteomics use case, this conversion is repeated per search space (Swiss-Prot Human, +Bacteria, +IGC) before pp.filter_peptides_for_model prepares peptides for Prosit prediction. Rationale: standardized, ProForma-compliant sequences are required so that theoretical fragment masses used during peak annotation with re.generate_features (massTolerance 20 for FTMS, 'ppm') match the predicted Prosit spectra.

## Related tools

- **Oktoberfest** (Provides pp.convert_search, pp.load_search, and pp.split_search to translate search engine results into the internal rescoring format) — https://github.com/wilhelm-lab/oktoberfest
- **spectrum_fundamentals** (Performs conversion between search engine-specific modstrings and the ProForma standard used internally) — https://github.com/wilhelm-lab/spectrum_fundamentals
- **spectrum_io** (File/data conversion for mass spec data within the Oktoberfest ecosystem (complementary spectra conversion)) — https://github.com/wilhelm-lab/spectrum_io

## Examples

```
from oktoberfest import pp as pr_pp; pr_pp.convert_search(search_results_type='Maxquant', search_results='/data/PXD010871/msms.txt', out_dir='/data/internal/')
```

## Evaluation signals

- All peptide sequences in the converted tables parse as valid ProForma notation with modification masses consistent with UniMod identifiers (retrievable from unimod.org)
- Row counts of converted PSM tables match the original MaxQuant output per search space (no silent PSM loss during conversion/splitting)
- Downstream pp.filter_peptides_for_model yields peptides whose theoretical masses agree with Prosit-predicted fragment ions within the annotation tolerance (20 ppm for FTMS)
- The converted per-search-space tables support the expected Figure 3C-D pattern: MaxQuant PSM counts collapse from 11,724 to 434 as search space grows, confirming conversion preserved search-engine identification semantics

## Limitations

- Conversion depends on the search engine type specified (search_results_type 'Maxquant'); unsupported engines or renamed columns will break the mapping
- Modification handling is bounded by UniMod IDs and masses; non-standard or unspecified modifications may not convert to ProForma correctly
- The README does not document a change log for format-mapping changes ('No changelog found'), so column-mapping behavior may shift between versions without notice

## Evidence

- [methods] pp.convert_search / pp.load_search / pp.split_search API listing: "pp.convert_search
   pp.load_search
   pp.split_search"
- [readme] spectrum_fundamentals modstring conversion: [evidence span withheld — anchor status: not_found]
- [readme] Oktoberfest end-to-end pipeline description: [evidence span withheld — anchor status: external_doc]
- [methods] UniMod modification mass reference: "The UNIMOD_ID and the corresponding monoisotopic modification mass can be retrieved from `unimod.org <https://unimod.org/>`_"
- [other] Task 005 conversion workflow step: [evidence span withheld — anchor status: not_found]
