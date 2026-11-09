---
name: gff3-annotation-merging
description: Use when PhosBoost prediction TSVs and DIAMOND inferred-positive CSVs
  must be projected from protein coordinates onto a genomic GFF3 to build a JBrowse
  visualization track.
when_to_use_negative:
- Target proteins are absent from the genomic GFF3 or contain non-canonical/missing
  amino acids, so no protein-to-genome codon projection is possible (the paper discards
  such sequences)
- The goal is PhosBoost model training, Bayesian hyperparameter tuning, or benchmarking
  on the 60%-20%-20% stratified splits rather than visualizing finished predictions
- You need a model-readable feature table or per-residue ProtT5 embeddings, not a
  browser-facing genomic track
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: PhosBoost
  role: Supplies the genome-wide, probability-scored phosphosite predictions (TSVs
    in /data/preds/) that are converted into GFF3 features
  repo: https://github.com/eporetsky/PhosBoost
- name: DIAMOND
  role: Supplies the pairwise-alignment inferred-positive annotations (CSV in /data/inferred/)
    merged as metadata onto the GFF3 phosphosite features
- name: JBrowse
  role: Target genome browser into which '<genome>.phosboost.gff3' is uploaded for
    styled visualization of predicted and DIAMOND-inferred phosphosites
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:88d79ee4b21d3d45e3cbfef41b56afbd27beedcc10140e13c7a1828eed7100be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eporetsky/PhosBoost
  tools:
  - DIAMOND
  - JBrowse
  - PhosBoost
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# gff3-annotation-merging

## Summary

Merges genome-wide PhosBoost phosphorylation predictions (TSV) and DIAMOND pairwise-alignment inferred positives (CSV) into a single JBrowse-ready GFF3 track by projecting each phosphosite's protein residue coordinate onto the genomic coordinates of its encoding three-basepair codon.

## When to use

Use when PhosBoost prediction TSVs and DIAMOND inferred-positive CSVs must be projected from protein coordinates onto a genomic GFF3 to build a JBrowse visualization track.

## When NOT to use

- Target proteins are absent from the genomic GFF3 or contain non-canonical/missing amino acids, so no protein-to-genome codon projection is possible (the paper discards such sequences)
- The goal is PhosBoost model training, Bayesian hyperparameter tuning, or benchmarking on the 60%-20%-20% stratified splits rather than visualizing finished predictions
- You need a model-readable feature table or per-residue ProtT5 embeddings, not a browser-facing genomic track

## Inputs

- Genomic GFF3 annotation file for the target genome (provides the CDS/mRNA features needed for protein-to-genome coordinate mapping)
- PhosBoost genome-wide prediction TSV from /data/preds/ (phosphosite ID, protein residue coordinate, predicted probability score)
- DIAMOND inferred-positives CSV from /data/inferred/ (phosphosites relabeled as inferred positives via pairwise alignment to experimentally derived sites)

## Outputs

- '<genome>.phosboost.gff3' written to /data/gff3/: GFF3 features spanning each phosphosite's 3-bp codon with attributes for phosphosite ID, predicted probability, DIAMOND inferred-positive status, and JBrowse styling (bold >0.9; blue/red/pink category colors)
- JBrowse-uploadable genome-browser track suitable for direct integration (as exposed via GrainGenes and MaizeGDB)

## How to apply

Load the target genome's GFF3 annotation, the PhosBoost prediction TSV from /data/preds/, and the DIAMOND inferred-positives CSV from /data/inferred/. For each predicted phosphosite, translate its protein residue coordinate into the genomic start/end of its encoding three-basepair codon using the CDS/mRNA features in the genomic GFF3 (sites on proteins absent from the genomic annotation, or with non-canonical residues, cannot be mapped and are dropped). Annotate each resulting GFF3 record with the phosphosite ID, the PhosBoost predicted probability score, and the DIAMOND pairwise-alignment inferred-positive match, so prediction and alignment evidence co-locate on one feature. Style the features for the browser: bold labels when predicted probability exceeds 0.9; blue when the site is DIAMOND-inferred, red when predicted probability exceeds 0.5, and pink when both conditions hold. Write the merged track as '<genome>.phosboost.gff3' into /data/gff3/ for direct JBrowse upload, as deployed through GrainGenes and MaizeGDB.

## Related tools

- **PhosBoost** (Supplies the genome-wide, probability-scored phosphosite predictions (TSVs in /data/preds/) that are converted into GFF3 features) — https://github.com/eporetsky/PhosBoost
- **DIAMOND** (Supplies the pairwise-alignment inferred-positive annotations (CSV in /data/inferred/) merged as metadata onto the GFF3 phosphosite features)
- **JBrowse** (Target genome browser into which '<genome>.phosboost.gff3' is uploaded for styled visualization of predicted and DIAMOND-inferred phosphosites)

## Evaluation signals

- Output file '<genome>.phosboost.gff3' exists in /data/gff3/ and parses as valid GFF3 (9 tab-separated columns per feature, valid directives)
- Each phosphosite feature's genomic start-end spans exactly three base pairs, matching its encoding codon
- Every annotated record carries the phosphosite ID, predicted probability score, and DIAMOND match status in its attribute column
- Browser styling follows the thresholds: bold label when probability > 0.9; blue = DIAMOND-inferred only, red = probability > 0.5 only, pink = both
- Counts of annotated records reconcile with the PhosBoost predictions and DIAMOND inferred positives that have valid genomic mappings

## Limitations

- Proteins not present in genomic databases, or with missing/non-canonical amino acids, are discarded and cannot be projected to genomic coordinates
- PhosBoost has a precision-recall tradeoff with more predicted false positives than PhosphoLingo or DeepPhos, so the track prioritizes recall over precision
- For Tyr phosphosites, the DIAMOND inferred-positive step was not as informative, possibly due to higher label imbalance

## Evidence

- [discussion] To facilitate the accessibility to the prediction data, we developed a method that converts the prediction results and the pairwise DIAMOND alignment results into a file: "To facilitate the accessibility to the prediction data, we developed a method that converts the prediction results and the pairwise DIAMOND alignment results into a file"
- [intro] allowing for the incorporation of PhosBoost prediction results directly in the genome browser to facilitate accessibility: "allowing for the incorporation of PhosBoost prediction results directly in the genome browser to facilitate accessibility"
- [intro] To improve phosphosite annotation and reduce phosphosite label uncertainty, we supplemented PhosBoost predictions with a DIAMOND ... pairwise alignment analysis step: [evidence span withheld — anchor status: not_found]
- [results] These results are now accessible through the GrainGenes (Yao et al., 2022) and MaizeGDB databases: "These results are now accessible through the GrainGenes (Yao et al., 2022) and MaizeGDB databases"
