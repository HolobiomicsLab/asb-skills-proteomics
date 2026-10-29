---
name: site-determining-ion-counting
description: Use when Mascot (.xml) results from ETD middle-down analysis of hypermodified
  histone tails must have PTM sites unambiguously localized before downstream quantification
  (isoScale) or mark counting.
when_to_use_negative:
- 'The next pipeline step is peptide quantification: isoScale consumes Mascot output
  in .csv extension plus the Histone Coder result file, so use the isoScale/FIRR quantification
  skill instead of re-running ion counting'
- The PTMs of interest fall outside the script's supported set (phosphorylation on
  S,T,Y; acetylation on K; mono- and dimethylation on K,R; trimethylation on K)
- Spectra lack ETD-type c/z fragment ions flanking the modification, so no site-determining
  ions can be counted and the PTM cannot be unambiguously localized by this method
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Histone Coder
  role: Java tool that counts site-determining MS/MS ions, applies spectrum-score/fragment-ion-type/MS-MS-tolerance
    filters, and converts Mascot output to Brno nomenclature
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database-search engine whose assigned PTM localizations and .xml output are
    the validation target
- name: Java
  role: Runtime requirement to activate Histone Coder
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:475e5cd6ae3d9ab3f56635519b3bd75f846f387bf9c38a650dac630d0f119df7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Java
  - Mascot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# site-determining-ion-counting

## Summary

Histone Coder validates Mascot-assigned PTM localizations in middle-down histone MS/MS by counting site-determining fragment ions on both sides of each modified site, eliminating false-positive PTM placements before quantification.

## When to use

Use when Mascot (.xml) results from ETD middle-down analysis of hypermodified histone tails must have PTM sites unambiguously localized before downstream quantification (isoScale) or mark counting.

## When NOT to use

- The next pipeline step is peptide quantification: isoScale consumes Mascot output in .csv extension plus the Histone Coder result file, so use the isoScale/FIRR quantification skill instead of re-running ion counting
- The PTMs of interest fall outside the script's supported set (phosphorylation on S,T,Y; acetylation on K; mono- and dimethylation on K,R; trimethylation on K)
- Spectra lack ETD-type c/z fragment ions flanking the modification, so no site-determining ions can be counted and the PTM cannot be unambiguously localized by this method

## Inputs

- Mascot database-search output in .xml extension (peptide-spectrum matches with Mascot-assigned PTM localizations)
- Histone tail sequences from middle-down ETD LC-MS/MS runs (RP trap column coupled to WCX-HILIC)
- User-selected interface filters: spectrum score threshold, fragment ion types (c/z), and MS/MS ion tolerance

## Outputs

- Histone Coder result file listing each combinatorial histone mark with the number and type of site-determining ions found between the assigned PTM localization and the closest alternative host residues
- PTM assignments converted to standard Brno nomenclature (e.g. K4me3K14ac)

## How to apply

Load the Mascot database-search output in .xml format with Histone Coder (a Java application). For each Mascot-assigned PTM, locate the closest other amino acids in the histone sequence that can host the same modification, then count site-determining c and z fragment ions in the spectrum between the assigned position and each alternative host residue, using the interface filters for spectrum score, fragment ion types and MS/MS tolerance. Retain a PTM assignment only when at least one site-determining ion is found on each side of the assigned position; the supported modifications are phosphorylation (S,T,Y), acetylation (K), mono- and dimethylation (K,R) and trimethylation (K). Convert the validated peptide identifications to standard Brno nomenclature (e.g. K4me3K14ac) and write a Histone Coder result file listing each combinatorial mark with the number and type of site-determining ions found.

## Related tools

- **Histone Coder** (Java tool that counts site-determining MS/MS ions, applies spectrum-score/fragment-ion-type/MS-MS-tolerance filters, and converts Mascot output to Brno nomenclature) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database-search engine whose assigned PTM localizations and .xml output are the validation target)
- **Java** (Runtime requirement to activate Histone Coder)

## Examples

```
java -jar "Histone Coder.jar"
```

## Evaluation signals

- Reproduce the Fig 2B reference output: the H3K27me2 example reports 2 site-determining ions on the left side and 8 on the right side
- Every retained PTM assignment has at least one site-determining ion on each side of the assigned position; assignments lacking flanking ions are rejected as ambiguous
- Reported counts match the number and type of c/z fragment ions falling within the selected MS/MS tolerance and passing the spectrum-score filter
- Output identifiers conform to standard Brno nomenclature (e.g. K4me3K14ac)
- Counts are reported relative to the closest alternative amino acid capable of hosting the same modification, not merely the nearest residue

## Limitations

- Histone Coder requires Java and its filters (spectrum score, fragment ion types, MS/MS tolerance) must be set manually through the interface
- Only phosphorylation (S,T,Y), acetylation (K), mono- and dimethylation (K,R) and trimethylation (K) are included in the script
- A spectrum without site-determining ions on both sides of the assigned PTM cannot resolve localization against nearby alternative sites, so those identifications are discarded

## Evidence
