---
name: maldi-mz-to-neutral-mass-conversion
description: Use when you have MALDI imaging peak lists of monoisotopic [M+H]+ m/z
  values (positive mode) and an LC-MS/MS peptide mass table (peptides.txt), and need
  to identify which MALDI peaks correspond to proteolytic peptides.
when_to_use_negative:
- Input m/z values are already neutral peptide masses (subtracting 1 Da would double-shift
  the masses)
- The goal is direct identification of MALDI peaks without an accompanying LC-MS/MS
  peptide mass table, since this skill only correlates MALDI masses against LC-MS/MS-derived
  peptide masses
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MALDImID
  role: R package implementing the m/z-to-neutral-mass conversion (subtract 1 Da)
    and the ±0.2 Da matching against LC-MS/MS peptide masses
- name: MALDImIDshiny
  role: Shiny app exposing the same MALDI-to-LC-MS/MS peak matching workflow interactively
  repo: budczies/MALDImIDshiny
- name: R
  role: Language in which the mass correlation (Tasks 3-4 of MALDImID) is implemented
    and executed
- name: Shiny
  role: Web framework hosting the MALDImID app for interactive use of the matching
    step
  repo: https://biosciences.shinyapps.io/Maldimid
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:3bce8495f618fa4a5c6e36bd9575cf0c2d32848d8e560205b04b0c13d3f22c07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/budczies/MALDImIDshiny
  tools:
  - MALDImID
  - MALDImIDshiny
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# maldi-mz-to-neutral-mass-conversion

## Summary

Converts monoisotopic [M+H]+ m/z values from MALDI imaging peak lists into neutral peptide masses by subtracting 1 Da (the proton/cation hydrogen), then matches them against LC-MS/MS peptide masses within a ±0.2 Da tolerance, as implemented in the MALDImID R package (Tasks 3-4).

## When to use

Use when you have MALDI imaging peak lists of monoisotopic [M+H]+ m/z values (positive mode) and an LC-MS/MS peptide mass table (peptides.txt), and need to identify which MALDI peaks correspond to proteolytic peptides.

## When NOT to use

- Input m/z values are already neutral peptide masses (subtracting 1 Da would double-shift the masses)
- The goal is direct identification of MALDI peaks without an accompanying LC-MS/MS peptide mass table, since this skill only correlates MALDI masses against LC-MS/MS-derived peptide masses

## Inputs

- MALDI imaging peak list of monoisotopic [M+H]+ m/z values (positive mode)
- LC-MS/MS peptide mass table from peptides.txt

## Outputs

- matched_masses.csv containing MALDI m/z, converted neutral mass, matched LC-MS peptide mass, and mass difference

## How to apply

Load the MALDI imaging peak list of monoisotopic [M+H]+ m/z values (positive mode) and the LC-MS/MS peptide mass table from peptides.txt into R using MALDImID. Convert each MALDI m/z value to a neutral mass by subtracting 1 Da, correcting for the proton of the cation (e.g., MALDI m/z 865.39 becomes neutral mass 864.39). For each neutral mass, search the LC-MS/MS peptide masses and retain all matches falling within a ±0.2 Da tolerance window. Assemble the matched entries (MALDI m/z, neutral mass, matched LC-MS/MS peptide mass, mass difference) into a table and write it to matched_masses.csv. The ±0.2 Da window reflects the mass accuracy attainable when correlating MALDI imaging peaks with LC-MS/MS-derived proteomic biomarker discovery data, as described in the MALDImID article (DOI 10.1002/pmic.202300005).

## Related tools

- **MALDImID** (R package implementing the m/z-to-neutral-mass conversion (subtract 1 Da) and the ±0.2 Da matching against LC-MS/MS peptide masses)
- **MALDImIDshiny** (Shiny app exposing the same MALDI-to-LC-MS/MS peak matching workflow interactively) — budczies/MALDImIDshiny
- **R** (Language in which the mass correlation (Tasks 3-4 of MALDImID) is implemented and executed)
- **Shiny** (Web framework hosting the MALDImID app for interactive use of the matching step) — https://biosciences.shinyapps.io/Maldimid

## Examples

```
matched <- peptides[abs((mz_peaks - 1) - peptides$mass) <= 0.2, ]; write.csv(matched, "matched_masses.csv")
```

## Evaluation signals

- Every reported match has an absolute mass difference between the converted MALDI neutral mass and the LC-MS peptide mass of at most 0.2 Da
- The output table matched_masses.csv exists and includes the required mass-difference column
- Spot-check a known conversion: MALDI m/z 865.39 must yield neutral mass 864.39 before matching
- All matched entries carry the four expected columns: MALDI m/z, neutral mass, matched LC-MS peptide mass, mass difference
- No MALDI m/z value retains its proton (i.e., all values in the matched table are reduced by 1 Da relative to the input peaks)

## Limitations

- Matching tolerance is fixed at ±0.2 Da, so peptides outside this window around the proton-corrected neutral mass will be missed; this is a coarse window relative to high-resolution MS tolerances
- Requires positive-mode monoisotopic [M+H]+ MALDI imaging peak lists; other adducts or charge states are not handled by this conversion
- Matches are mass-only correlations; they are not unique peptide identifications and depend entirely on the quality of the LC-MS/MS peptide mass table from peptides.txt

## Evidence

- [abstract] we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data for more specific identification of peaks: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data for more specific identification of peaks"
- [abstract] The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid: "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid."
- [abstract] The package is made available under the GPL 3 license.: "The package is made available under the GPL 3 license."
