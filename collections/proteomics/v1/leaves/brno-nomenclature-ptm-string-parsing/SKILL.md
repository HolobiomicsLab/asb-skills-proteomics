---
name: brno-nomenclature-ptm-string-parsing
description: Use when Mascot .xml/.csv output from middle-down histone MS/MS runs
  must be converted to standardized PTM strings and parsed to enumerate combinatorial
  marks (e.g. K4me3K14ac, H3K27me3K36me3) per peptide.
when_to_use_negative:
- The data are bottom-up tryptic histone peptides with few PTMs per peptide rather
  than hypermodified middle-down N-terminal tails, where isoScale/FIRR-based isobaric
  deconvolution is unnecessary
- The input is not Mascot search output (.xml for Histone Coder, .csv for isoScale),
  e.g. raw spectra or mzML files directly
- PTM strings are already in Brno nomenclature from a prior Histone Coder run and
  only downstream quantification is needed
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Histone Coder
  role: Counts site-determining MS/MS ions to validate PTM localization and converts
    Mascot output into Brno-nomenclature PTM strings (e.g. K4me3K14ac); its result
    file is the parsed input for isoScale
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: isoScale
  role: Consumes the Histone Coder result file plus Mascot .csv to quantify peptides
    by total MS/MS ion intensity and deconvolute isobaric co-fragmented PTM species
    via FIRR
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Mascot
  role: Database search engine whose assigned PTM localizations (.xml/.csv output)
    are validated, converted to Brno nomenclature, and quantified
- name: Java
  role: Runtime requirement for executing the Histone Coder and isoScale .jar tools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:24872c46c77d57a9913d79a2be998d03c599ef7f236f20c6a97b516d32755d97
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Java
  - Mascot
  - isoScale
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# brno-nomenclature-ptm-string-parsing

## Summary

Parse PTM strings encoded in the standard Brno nomenclature (e.g. K4me3K14ac) that Histone Coder produces from Mascot search output, so that combinatorial histone marks can be counted, paired, and quantified downstream (e.g. for PTM co-existence statistics with isoScale).

## When to use

Use when Mascot .xml/.csv output from middle-down histone MS/MS runs must be converted to standardized PTM strings and parsed to enumerate combinatorial marks (e.g. K4me3K14ac, H3K27me3K36me3) per peptide.

## When NOT to use

- The data are bottom-up tryptic histone peptides with few PTMs per peptide rather than hypermodified middle-down N-terminal tails, where isoScale/FIRR-based isobaric deconvolution is unnecessary
- The input is not Mascot search output (.xml for Histone Coder, .csv for isoScale), e.g. raw spectra or mzML files directly
- PTM strings are already in Brno nomenclature from a prior Histone Coder run and only downstream quantification is needed

## Inputs

- Mascot output in .xml extension (input to Histone Coder)
- Mascot .csv output file (data file input to isoScale)
- Histone Coder result file (Histone Coder output format, input to isoScale)

## Outputs

- Peptide records with PTM strings in standard Brno nomenclature (e.g. K4me3K14ac) plus site-determining ion counts per spectrum
- Quantified combinatorial histone mark table (relative abundance from total MS/MS ion intensity, FIRR-deconvoluted isobaric species)
- Co-existence statistics per PTM pair (e.g. wt vs Suz12-/- percentages, as in ptm_coexistence_stats.csv)

## How to apply

Run Histone Coder on the Mascot output (.xml), which counts site-determining MS/MS ions to validate PTM localization and converts each identified peptide to a Brno-nomenclature PTM string such as K4me3K14ac. The supported modification set is phosphorylation (S,T,Y), acetylation (K), mono- and dimethylation (K,R), and trimethylation (K); the interface allows filtering by spectrum score, fragment ion types, and MS/MS ion tolerance. Parse each resulting PTM string by tokenizing residues plus modification tags (me1/me2/me3 for K/R, ac for K, ph for S/T/Y) to derive per-tail modification combinations. Feed the Histone Coder result file together with the Mascot .csv data file into isoScale, which quantifies peptides by total MS/MS ion intensity and deconvolutes isobaric co-fragmented species via the fragment ion relative ratio (FIRR); PTM pairs (e.g. H3K9me3K27me3) are then counted over quantified spectra. Rationale: unambiguous localization from site-determining ions is required before combinatorial marks can be trusted for cross-talk statistics.

## Related tools

- **Histone Coder** (Counts site-determining MS/MS ions to validate PTM localization and converts Mascot output into Brno-nomenclature PTM strings (e.g. K4me3K14ac); its result file is the parsed input for isoScale) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **isoScale** (Consumes the Histone Coder result file plus Mascot .csv to quantify peptides by total MS/MS ion intensity and deconvolute isobaric co-fragmented PTM species via FIRR) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Mascot** (Database search engine whose assigned PTM localizations (.xml/.csv output) are validated, converted to Brno nomenclature, and quantified)
- **Java** (Runtime requirement for executing the Histone Coder and isoScale .jar tools)

## Examples

```
java -jar "Histone Coder.jar"  # GUI: load Mascot .xml output, set spectrum score / fragment ion type / MS/MS tolerance filters, export Brno-nomenclature result file for isoScale
```

## Evaluation signals

- Every parsed PTM string conforms to Brno nomenclature with supported modifications only: phosphorylation on S/T/Y, acetylation on K, mono-/dimethylation on K/R, trimethylation on K
- Each retained spectrum has unambiguous PTM localization supported by counted site-determining ions (Histone Coder lists number and type of site-determining ions per spectrum)
- Cross-talk sanity check: co-existence of mutually exclusive repressive pairs should be near zero, e.g. H3K9me3K27me3 never co-detected and H3K27me3K36me3 below 0.5% (wt) and 0.2% (Suz12-/-), while permissive pairs such as H3K18ac/H3K23ac co-detected in ~54% of relevant peptides and H4S1acK20me2 at 54.1% (wt) / 60.2% (Suz12-/-)
- Isobaric peptides sharing a sequence but differing in PTM localization are deconvoluted by FIRR before counting, so summed relative abundances per histone type remain consistent across wt and Suz12-/- datasets
- Output table contains the expected columns (histone type, PTM pair, wt co-existence percentage, Suz12-/- co-existence percentage) with no duplicate PTM-string variants from unsplit isobaric counts

## Limitations

- Histone Coder only handles the predefined PTM set (phosphorylation S/T/Y, acetylation K, mono-/dimethylation K/R, trimethylation K); other modification types will not be encoded into Brno strings
- Requires Java and specific Mascot output formats (.xml for Histone Coder, .csv for isoScale) rather than generic search-engine exports
- Co-fragmenting isobaric species with distinct PTM localizations must be deconvoluted via FIRR, and results depend on the chosen MS/MS ion tolerance and fragment ion type filters (CID/HCD vs ETD) in the isoScale interface

## Evidence

- [intro] Histone Coder converts the Mascot output to the standard Brno nomenclature for PTMs (e.g. K4me3K14ac).: "Histone Coder converts the Mascot output to the standard Brno nomenclature for PTMs (e.g. K4me3K14ac)."
- [intro] Histone Coder counts the number of MS/MS ions in a given spectrum to determine the unambiguous localization of a post-translational modification (PTM).: "Histone Coder counts the number of MS/MS ions in a given spectrum to determine the unambiguous localization of a post-translational modification (PTM)."
- [methods] The software input is the result file, which is the output format of Histone Coder, and a data file, which is the Mascot .csv output: "The software input is the result file, which is the output format of Histone Coder, and a data file, which is the Mascot .csv output"
- [readme] The PTMs included in the script are phosphorylation (S,T,Y), acetylation (K), mono- and dimethylation (K,R) and trimethylation (K).: "The PTMs included in the script are phosphorylation (S,T,Y), acetylation (K), mono- and dimethylation (K,R) and trimethylation (K)."
- [readme] The software provides also relative quantification of isobaric peptides co-fragmented in MS/MS spectra which share the same sequence but have distinct localizations of post-translational: "The software provides also relative quantification of isobaric peptides co-fragmented in MS/MS spectra which share the same sequence but have distinct localizations of post-translational"
