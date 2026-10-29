---
name: mascot-output-parsing
description: Use when raw Mascot search results from middle-down histone LC-MS/MS
  (ETD/CID-HCD) runs must be converted to Histone Coder/isoScale inputs, or when PTM-localized
  peptide-spectrum matches in Brno nomenclature (e.g. K4me3K14ac) are needed downstream.
when_to_use_negative:
- Input spectra were searched with a different search engine (e.g. MaxQuant, Proteome
  Discoverer) rather than Mascot — the tools specifically process Mascot .xml/.csv
  output formats.
- The goal is bottom-up peptide quantification from MS1 precursor intensities or spectral
  counting alone — isoScale is explicitly recommended for middle-down or top-down
  analyses of single sequences with scrambled combinatorial PTMs.
- PTMs outside the Histone Coder scope (e.g. ubiquitination, glycation on residues
  other than the supported S/T/Y, K, R methylation states) are present and cannot
  be handled by the built-in modification list.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Histone Coder
  role: Consumes Mascot .xml output; filters on spectrum score, fragment ion types
    and MS/MS tolerance; counts site-determining ions to validate PTM localization
    and converts output to Brno nomenclature.
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: isoScale
  role: Consumes the Histone Coder result file plus Mascot .csv data file; quantifies
    peptides from total MS/MS ion intensity and computes FIRR for isobaric co-fragmenting
    peptides.
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Mascot
  role: Database search engine whose PTM localization assignments and .xml/.csv exports
    are the parsed input to the tool chain.
- name: Java
  role: Runtime requirement to execute the Histone Coder and isoScale .jar tools.
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:3d1ca562a3a7cd2a99b16d3a3635b7e7a7c6b9cb56b7acb3052595f4760bf5fd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Iso Scale
  - Java
  - Mascot
  - isoScale
  - pandas
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# mascot-output-parsing

## Summary

Parsing Mascot (Matrix Science) database-search outputs into the formats consumed by the middle-down proteomics tools: Mascot .xml feeds Histone Coder for PTM-localization validation, and a Histone Coder result file plus Mascot .csv data file feed isoScale for MS/MS total-ion-intensity quantification of combinatorially modified histone peptides.

## When to use

Use when raw Mascot search results from middle-down histone LC-MS/MS (ETD/CID-HCD) runs must be converted to Histone Coder/isoScale inputs, or when PTM-localized peptide-spectrum matches in Brno nomenclature (e.g. K4me3K14ac) are needed downstream.

## When NOT to use

- Input spectra were searched with a different search engine (e.g. MaxQuant, Proteome Discoverer) rather than Mascot — the tools specifically process Mascot .xml/.csv output formats.
- The goal is bottom-up peptide quantification from MS1 precursor intensities or spectral counting alone — isoScale is explicitly recommended for middle-down or top-down analyses of single sequences with scrambled combinatorial PTMs.
- PTMs outside the Histone Coder scope (e.g. ubiquitination, glycation on residues other than the supported S/T/Y, K, R methylation states) are present and cannot be handled by the built-in modification list.

## Inputs

- Mascot database search output in .xml extension (input to Histone Coder)
- Mascot database search output in .csv extension (data file input to isoScale)
- Histone Coder result file (output format of Histone Coder, required input to isoScale)

## Outputs

- Histone Coder result file with PTM-localization statistics (number and type of site-determining ions) and PTMs converted to Brno nomenclature (e.g. K4me3K14ac)
- Validated, PTM-localized peptide-spectrum matches ready for isoScale total-ion-intensity quantification and FIRR-based deconvolution of isobaric peptides

## How to apply

Run the Mascot database search over the histone spectra, then export per the tool chain: Histone Coder requires the Mascot output in .xml extension, while isoScale requires a Mascot .csv output together with the result file produced by Histone Coder. In the Histone Coder interface, set the filters for spectrum score, fragment ion types, and MS/MS ion tolerance for the search; the software counts site-determining MS/MS ions between the Mascot-assigned PTM localization and the closest other amino acids that can host the modification (phosphorylation S/T/Y, acetylation K, mono-/dimethylation K/R, trimethylation K) to validate unambiguous localization. Histone Coder then converts the Mascot output to the standard Brno nomenclature for PTMs, producing the validated result file. That result file plus the Mascot .csv are the two inputs isoScale uses to quantify peptides by total ion intensity of MS/MS spectra and to compute the fragment ion relative ratio (FIRR) for isobaric co-fragmenting species, with fragmentation mode (CID/HCD vs ETD) and MS/MS ion tolerance selected from the isoScale interface.

## Related tools

- **Histone Coder** (Consumes Mascot .xml output; filters on spectrum score, fragment ion types and MS/MS tolerance; counts site-determining ions to validate PTM localization and converts output to Brno nomenclature.) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **isoScale** (Consumes the Histone Coder result file plus Mascot .csv data file; quantifies peptides from total MS/MS ion intensity and computes FIRR for isobaric co-fragmenting peptides.) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Mascot** (Database search engine whose PTM localization assignments and .xml/.csv exports are the parsed input to the tool chain.)
- **Java** (Runtime requirement to execute the Histone Coder and isoScale .jar tools.)

## Examples

```
java -jar "Histone Coder.jar"  # load Mascot .xml, set spectrum-score/fragment-ion/MS-MS-tolerance filters, export Brno-nomenclature result file for isoScale
```

## Evaluation signals

- Histone Coder output is in standard Brno nomenclature for PTMs (e.g. K4me3K14ac) rather than raw Mascot modification annotations.
- Each retained peptide-spectrum match carries site-determining-ion counts supporting the Mascot-assigned PTM localization versus the closest alternative amino acids that can host the modification.
- isoScale accepts exactly the two expected files — the Histone Coder result file and the Mascot .csv data file — without format errors.
- Downstream quantification covers known histone-tail peptide groups (e.g. the 22 H3 peptide groups with 0-3 acetyl and 0-9 methyl PTM equivalents), as in the isoScale vs spectral counting benchmark against XIC.
- Spectrum score, fragment ion type, and MS/MS tolerance filter settings match those documented for the experiment (ETD fragmentation in the Suz12-/- histone H3 study).

## Limitations

- Histone Coder only handles a fixed PTM set: phosphorylation (S,T,Y), acetylation (K), mono- and dimethylation (K,R) and trimethylation (K); other modifications are not parsed or validated.
- Histone Coder input is Mascot output in .xml extension, while isoScale requires Mascot .csv — using the wrong export format for either tool breaks the chain.
- The FIRR-based relative quantification of isobaric co-fragmenting peptides depends on the chosen MS/MS ion tolerance, and low-resolution data may degrade isobaric deconvolution.

## Evidence

- [readme] The input is Mascot output in .xml extension.: [evidence span withheld — anchor status: external_doc]
