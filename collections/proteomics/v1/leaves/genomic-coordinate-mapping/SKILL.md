---
name: genomic-coordinate-mapping
description: Use when predicted phosphosites given in protein residue coordinates
  must be projected onto genome coordinates and merged with DIAMOND inferred positives
  into a GFF3 genome-browser track.
when_to_use_negative:
- Prediction results are already expressed in genomic coordinates or already merged
  into a GFF3 track
- The proteins carrying predicted phosphosites are not present in the genomic databases
  or contain non-canonical/missing amino-acid sequences, so no genomic codon can be
  mapped
- The goal is to retrain or score PhosBoost models rather than to export existing
  prediction and DIAMOND results for browser display
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: PhosBoost
  role: Supplies the protein-coordinate phosphorylation predictions (probability scores)
    that are mapped onto genomic codons and annotated as GFF3 features
  repo: https://github.com/eporetsky/PhosBoost
- name: DIAMOND
  role: Pairwise alignment analysis whose inferred-positive phosphosites are merged
    with PhosBoost predictions in the GFF3 track
- name: JBrowse
  role: Target genome browser into which the generated '<genome>.phosboost.gff3' track
    is uploaded directly, with color/bold styling applied
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:f3ffc16a5b90d015f2962dbd9f5ce47cca355817ca434278921a16fc16316dcb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eporetsky/PhosBoost
  tools:
  - DIAMOND
  - JBrowse
  - PhosBoost
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Genomic coordinate mapping

## Summary

Converts protein-coordinate PhosBoost phosphorylation predictions and DIAMOND pairwise-alignment inferred positives into genomic-coordinate GFF3 features annotated onto an existing genome GFF3, producing a single '<genome>.phosboost.gff3' track loadable directly into JBrowse.

## When to use

Use when predicted phosphosites given in protein residue coordinates must be projected onto genome coordinates and merged with DIAMOND inferred positives into a GFF3 genome-browser track.

## When NOT to use

- Prediction results are already expressed in genomic coordinates or already merged into a GFF3 track
- The proteins carrying predicted phosphosites are not present in the genomic databases or contain non-canonical/missing amino-acid sequences, so no genomic codon can be mapped
- The goal is to retrain or score PhosBoost models rather than to export existing prediction and DIAMOND results for browser display

## Inputs

- Genomic GFF3 annotation file for the target genome
- PhosBoost prediction results TSV (from /data/preds/), with phosphosite IDs and predicted probability scores in protein coordinates
- DIAMOND pairwise-alignment inferred-positives CSV (from /data/inferred/)

## Outputs

- Merged GFF3 track '<genome>.phosboost.gff3' in /data/gff3/, with PhosBoost and DIAMOND annotations mapped to genomic coordinates and browser styling attributes

## How to apply

Load three inputs: the genomic GFF3 annotation, the PhosBoost prediction TSV (from /data/preds/), and the DIAMOND inferred-positives CSV (from /data/inferred/). For each predicted phosphosite, map the protein residue position onto the genomic coordinates of its encoding three-basepair codon, using the protein-to-genome correspondence in the genomic GFF3. Annotate the GFF3 records with PhosBoost metadata (phosphosite ID, predicted probability score) and DIAMOND pairwise-alignment match information, and encode visual styling: bold labels for probability above 0.9, blue for DIAMOND-inferred sites, red for predicted probability above 0.5, and pink for sites that are both. Write the merged track as '<genome>.phosboost.gff3' into /data/gff3/ so it can be uploaded directly to JBrowse. The rationale is that DIAMOND relabeling recovers inferred positives that would otherwise be counted as false positives, and browser integration makes genome-wide predictions accessible alongside genome annotation.

## Related tools

- **PhosBoost** (Supplies the protein-coordinate phosphorylation predictions (probability scores) that are mapped onto genomic codons and annotated as GFF3 features) — https://github.com/eporetsky/PhosBoost
- **DIAMOND** (Pairwise alignment analysis whose inferred-positive phosphosites are merged with PhosBoost predictions in the GFF3 track)
- **JBrowse** (Target genome browser into which the generated '<genome>.phosboost.gff3' track is uploaded directly, with color/bold styling applied)

## Evaluation signals

- Output file '<genome>.phosboost.gff3' exists in /data/gff3/ and is parseable GFF3
- The track contains records with both PhosBoost and DIAMOND annotation mapped to genomic coordinates
- Each phosphosite feature spans the genomic coordinates of its encoding three-basepair codon (3 bp feature length on the chromosome)
- Feature attributes include the phosphosite ID and predicted probability score, with DIAMOND match metadata where applicable
- Styling thresholds are correctly encoded: bold labels for probability > 0.9, blue for DIAMOND-inferred, red for predicted probability > 0.5, pink for both

## Limitations

- PhosBoost has a precision-recall tradeoff with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos, so the track will contain low-confidence sites (only partially mitigated by the DIAMOND annotation step)
- For Tyr phosphosites, PhosBoost was not as informative at predicting inferred positives, possibly due to higher label imbalance, so Y-site tracks may be less reliable
- Proteins not present in genomic databases or containing missing/non-canonical amino-acid sequences were discarded upstream, so no track features can be generated for them
- No software version numbers for DIAMOND are stated, complicating exact reproduction of the inferred-positives input

## Evidence

- [intro] allowing for the incorporation of PhosBoost prediction results directly in the genome browser to facilitate accessibility: "allowing for the incorporation of PhosBoost prediction results directly in the genome browser to facilitate accessibility"
- [discussion] DIAMOND conversion to browser file: "we developed a method that converts the prediction results and the pairwise DIAMOND alignment results into a file"
- [results] DIAMOND 31-residue pairwise alignments: "Using DIAMOND, we extracted peptide sequences of size 31 centered at the Ser/Thr phosphosites in the test dataset and conducted pairwise alignments"
- [results] Inferred positives relabeling: "Phosphosites matching any experimentally derived phosphosite, excluding self‐matches, were relabeled as inferred positives (IP)"
- [methods] Filtering unmappable proteins: "Proteins that are not present in genomic databases or contain missing or non‐canonical amino‐acid sequences were discarded"
