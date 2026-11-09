---
name: tandem-mass-spectrometry-peptide-identification
description: Use when MS/MS spectra must be identified against a peptide sequence
  database, especially when mutations or unexpected post-translational modifications
  may be present or spectra are of low quality.
when_to_use_negative:
- No MS/MS spectrum is available or the goal is identification from a different data
  type (e.g. intact protein or proteomics feature tables).
- Peptides are expected to be unmodified and fully standard — the tool is specifically
  aimed at characterizing mutations or unexpected post-translational modifications,
  so a conventional search tool may suffice otherwise.
- 'A maintained, supported tool is required: the README states the author ''will not
  make any new development nor will I perform any maintenance/support on the tool.'''
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Popitam
  role: 'Core tool: builds a spectrum-graph representation of the MS/MS spectrum,
    parses it with an Ant Colony Optimization algorithm against theoretical peptide
    sequences, and ranks candidate peptides by correlation score.'
  repo: https://github.com/chernan/popitam
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.200300402
    title: 'Popitam: Towards new heuristic strategies to improve protein identification
      from tandem mass spectrometry data'
  source_content_hash: sha256:a36875ce49fe393c98ec2ca2dae6ebc37c3461b1b262b2a69f7774db99db64e8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chernan/popitam
  tools:
  - Popitam
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# tandem-mass-spectrometry-peptide-identification

## Summary

Identifies peptides from tandem mass spectrometry (MS/MS) data by matching theoretical peptide sequences from a database against a spectrum-graph representation of the source spectrum, using Popitam's Ant Colony Optimization parsing to produce peptides ranked by a correlation score. It is designed to handle peptides with mutations or unexpected post-translational modifications.

## When to use

Use when MS/MS spectra must be identified against a peptide sequence database, especially when mutations or unexpected post-translational modifications may be present or spectra are of low quality.

## When NOT to use

- No MS/MS spectrum is available or the goal is identification from a different data type (e.g. intact protein or proteomics feature tables).
- Peptides are expected to be unmodified and fully standard — the tool is specifically aimed at characterizing mutations or unexpected post-translational modifications, so a conventional search tool may suffice otherwise.
- A maintained, supported tool is required: the README states the author 'will not make any new development nor will I perform any maintenance/support on the tool.'

## Inputs

- Sample MS/MS spectrum data (spectrum-graph input files, e.g. the repository's shipped example MS/MS spectrum)
- A peptide sequence database providing theoretical peptide sequences

## Outputs

- Ranked peptide identification list (peptides ranked by correlation score), e.g. popitam_ranked_results.txt with a correlation-score column

## How to apply

Obtain the improved Popitam source from github.com/chernan/popitam (the version migrated from code.google.com/p/popitam, made 1.4 times more efficient by Florent Glück) and compile it with the repository's build script. Run the Popitam executable on the MS/MS spectrum input, which the software converts into a structured spectrum-graph representation similar to those used by de novo sequencing software. The Ant Colony Optimization bio-inspired algorithm parses this graph — a highly combinatorial task — by matching theoretical peptide sequences issued from a database and emphasizing relevant sections of the graph for each candidate. Collect the resulting list of peptides ranked by correlation score and save it (e.g. as popitam_ranked_results.txt). Verify the run script completed successfully and that the output matches the repository's shipped reference results, or at minimum contains a correlation-score column.

## Related tools

- **Popitam** (Core tool: builds a spectrum-graph representation of the MS/MS spectrum, parses it with an Ant Colony Optimization algorithm against theoretical peptide sequences, and ranks candidate peptides by correlation score.) — https://github.com/chernan/popitam

## Evaluation signals

- The output file popitam_ranked_results.txt exists after the run and the run script completed successfully.
- The ranked results contain a correlation-score column, since the process 'leads to a list of peptides ranked by a correlation score'.
- Results match the repository's shipped reference results (or at minimum contain a correlation-score column).
- The Popitam executable compiled successfully from the github.com/chernan/popitam source using the repository's build script.
- The spectrum input was converted into a spectrum-graph representation before candidate matching, mirroring the paper's described pipeline.

## Limitations

- The GitHub repository is an archive: the author will not make new development nor perform maintenance/support on the tool.
- The algorithm's strength (handling modifications/mutations, large databases, and low quality data) is described as 'promising properties' in the abstract, not a guaranteed performance claim.
- Graph parsing is described as a potentially highly combinatorial task, handled heuristically by the Ant Colony Optimization algorithm rather than exhaustively.

## Evidence
