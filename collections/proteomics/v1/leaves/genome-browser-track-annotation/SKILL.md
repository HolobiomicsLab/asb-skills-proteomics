---
name: genome-browser-track-annotation
description: Use when PhosBoost per-site prediction tsv files and DIAMOND inferred-positives
  csv files exist for a proteome and a genomic gff3 is available, and you need a JBrowse-compatible
  phosphosite track.
when_to_use_negative:
- Only a single input (e.g. just prediction tsv) is available without the genomic
  gff3 needed to map protein coordinates to genomic codon coordinates
- The goal is model training or prediction itself rather than visualization of existing
  PhosBoost/DIAMOND results
- The target browser does not accept GFF3 tracks or protein-to-genome coordinate mapping
  cannot be established for the proteome/genome pair
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3510
- http://edamontology.org/topic_0092
tools:
- name: PhosBoost
  role: Source of per-site phosphorylation probability scores (tsv) that are annotated
    onto genomic gff3 records
  repo: https://github.com/eporetsky/PhosBoost
- name: DIAMOND
  role: Provides pairwise-alignment inferred-positive phosphosite annotations merged
    into the GFF3 track metadata
- name: JBrowse
  role: Target genome browser into which the '<genome>.phosboost.gff3' track is uploaded
    (GrainGenes, MaizeGDB)
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:77e4461a327f93c28f1cf3586b5516150fb52a09d89362d070c34e0710814118
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eporetsky/PhosBoost
  tools:
  - CatBoost
  - DIAMOND
  - JBrowse
  - PhosBoost
  - ProtT5-XL-U50
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3510
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# genome-browser-track-annotation

## Summary

Convert genome-wide PhosBoost phosphorylation predictions and DIAMOND inferred-positive alignments into a GFF3 genome-browser track ('<genome>.phosboost.gff3') by mapping protein-coordinate phosphosites to genomic coordinates of their encoding codons, enabling direct upload to JBrowse (e.g. GrainGenes, MaizeGDB).

## When to use

Use when PhosBoost per-site prediction tsv files and DIAMOND inferred-positives csv files exist for a proteome and a genomic gff3 is available, and you need a JBrowse-compatible phosphosite track.

## When NOT to use

- Only a single input (e.g. just prediction tsv) is available without the genomic gff3 needed to map protein coordinates to genomic codon coordinates
- The goal is model training or prediction itself rather than visualization of existing PhosBoost/DIAMOND results
- The target browser does not accept GFF3 tracks or protein-to-genome coordinate mapping cannot be established for the proteome/genome pair

## Inputs

- Genomic gff3 annotation file for the target genome
- PhosBoost prediction tsv file (per-site probability scores) from /data/preds/
- DIAMOND inferred-positives csv file (pairwise alignment annotations) from /data/inferred/

## Outputs

- Merged GFF3 track named '<genome>.phosboost.gff3' in /data/gff3/ with phosphosite features carrying phosphosite ID, predicted probability score, and DIAMOND match metadata
- JBrowse genome-browser phosphosite track (e.g. GrainGenes wheat, barley, oat browsers; MaizeGDB)

## How to apply

Load three inputs: the genomic gff3 annotation, the PhosBoost prediction tsv from /data/preds/, and the DIAMOND inferred-positives csv from /data/inferred/. For each predicted phosphosite, map its protein coordinate to the genomic coordinates of the encoding three-basepair codon in the genome gff3. Annotate the genomic records with the PhosBoost predicted probability score, the phosphosite ID, and the DIAMOND pairwise-alignment matches; apply browser styling such as bold labels for probability above .9, blue for DIAMOND-inferred sites, red for predicted probability above .5, and pink for both. Write the merged track as '<genome>.phosboost.gff3' into /data/gff3/ so it can be uploaded directly to JBrowse (as demonstrated for GrainGenes Triticum aestivum IWGSC v2.1, Hordeum vulgare Morex v3, and Avena sativa Sang v1).

## Related tools

- **PhosBoost** (Source of per-site phosphorylation probability scores (tsv) that are annotated onto genomic gff3 records) — https://github.com/eporetsky/PhosBoost
- **DIAMOND** (Provides pairwise-alignment inferred-positive phosphosite annotations merged into the GFF3 track metadata)
- **JBrowse** (Target genome browser into which the '<genome>.phosboost.gff3' track is uploaded (GrainGenes, MaizeGDB))

## Examples

```
sbatch predict.sh; python convert_to_gff3.py --gff3 genome.gff3 --preds data/preds/genome.tsv --inferred data/inferred/genome.csv --out data/gff3/genome.phosboost.gff3
```

## Evaluation signals

- Output '<genome>.phosboost.gff3' file exists in /data/gff3/ and is parseable as valid GFF3
- Output contains PhosBoost/DIAMOND annotation records mapped to genomic coordinates of the encoding three-basepair codons
- Each phosphosite feature carries the phosphosite ID, predicted probability score, and DIAMOND match metadata fields
- Visual styling rules hold in browser view: bold labels for probability > .9, blue for DIAMOND-inferred, red for probability > .5, pink for both
- Tracks upload successfully to JBrowse (e.g. GrainGenes Triticum aestivum IWGSC v2.1, Hordeum vulgare Morex v3, Avena sativa Sang v1)

## Limitations

- Coordinate mapping depends on correctly matching predicted phosphosite protein coordinates to genomic codons; proteome-genome inconsistencies break the track
- PhosBoost has a precision-recall tradeoff with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos, inflating the number of red-styled sites in the track
- For Tyr phosphosites, PhosBoost was not as informative at predicting inferred positives, so Y-site DIAMOND styling may be sparse

## Evidence

- [other] Map each predicted phosphosite's protein coordinate to the genomic coordinates of its encoding three-basepair codon.: [evidence span withheld — anchor status: not_found]
- [other] GFF3 file suitable for direct integration within a JBrowse genome browser: [evidence span withheld — anchor status: not_found]
- [other] visual styling applied in the browser view: [evidence span withheld — anchor status: not_found]
- [intro] allowing for the incorporation of PhosBoost prediction results directly in the genome browser: "allowing for the incorporation of PhosBoost prediction results directly in the genome browser to facilitate accessibility"
- [discussion] we developed a method that converts the prediction results and the pairwise DIAMOND alignment results into a file: "To facilitate the accessibility to the prediction data, we developed a method that converts the prediction results and the pairwise DIAMOND alignment results into a file"
