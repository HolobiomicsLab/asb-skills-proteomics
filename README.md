<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/brand/asb-logo-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="assets/brand/asb-logo.svg">
    <img alt="AgenticScienceBuilder" src="assets/brand/asb-logo.svg" width="320">
  </picture>
</p>

<h1 align="center">ASB Skills: Proteomics</h1>

<p align="center"><em>Evidence-grounded skills and tool records for computational proteomics: mass-spectrometry-based protein identification and quantification.</em></p>

<p align="center">
  <img src="https://img.shields.io/badge/skills-558-00284b" alt="558 skills">
  <img src="https://img.shields.io/badge/tools-163-00284b" alt="163 tools">
  <img src="https://img.shields.io/badge/workflows-2-00284b" alt="2 workflows">
  <img src="https://img.shields.io/badge/papers-28-00284b" alt="28 source papers">
  <img src="https://img.shields.io/badge/content-CC--BY--4.0-04919a" alt="content CC-BY-4.0">
  <img src="https://img.shields.io/badge/code-Apache--2.0-04919a" alt="code Apache-2.0">
  <img src="https://img.shields.io/badge/status-released-04919a" alt="status released">
</p>

---

## TL;DR

This repository holds the proteomics skill collection of AgenticScienceBuilder (ASB): 558 skills over 163 software tools and 2 composite workflows, distilled from 28 method papers and their code repositories. Each skill is a `SKILL.md` file that an agent can load. It names its source paper; 282 of the 558 also quote it or its repository README. The collection installs as a Claude Code plugin, and a script that needs only the Python standard library searches it offline.

## Collections

| Path | Title | Skills | Tools | Released | DOI |
|---|---|---|---|---|---|
| `collections/proteomics/v1` | ASB Proteomics Skill Collection v1 | 558 | 163 | 2026-11-14 | [10.5281/zenodo.24224624](https://doi.org/10.5281/zenodo.24224624) |

## Status

Collection `v1`, built from corpus cut `c0107b2c545e`, is released on `main` at tag `proteomics-v1.1.0` and archived on Zenodo under the version DOI [10.5281/zenodo.24224624](https://doi.org/10.5281/zenodo.24224624).

| Item | State |
|---|---|
| Release gate | `receipts/c0107b2c545e/gate_report.json` reports `pass`: 14 of 16 checks pass and 2 are not applicable (`cut_manifest`, and `peer_review_claim` because the corpus holds no preprint or deposit DOI). [`receipts/c0107b2c545e/VERIFY.md`](receipts/c0107b2c545e/VERIFY.md) gives the command that re-verifies it. |
| Plugin manifests | `claude plugin validate` passes for `.claude-plugin/marketplace.json` and for `collections/proteomics/v1` |
| Collection IRI | `https://w3id.org/holobiomicslab/asb-skill/collection/proteomics/v1` |
| IRI resolution (2026-11-14) | Registered at w3id.org. It redirects (HTTP 302) to the collection landing page in the asb-skill-collections registry (HTTP 200), which names this repository and the Zenodo record. |
| DOI | [10.5281/zenodo.24224624](https://doi.org/10.5281/zenodo.24224624) (this version). The Zenodo deposit metadata is in [`.zenodo.json`](.zenodo.json). |
| Concept DOI | [10.5281/zenodo.24093113](https://doi.org/10.5281/zenodo.24093113) (all versions) |

## What is inside

| Item | Count | Path under `collections/proteomics/v1/` |
|---|--:|---|
| Leaf skills (`SKILL.md`) | 558 | `leaves/<slug>/SKILL.md` |
| Skill router | 1 | `skills/_router/` |
| Composite workflows | 2 | `workflows/<slug>/` |
| Workflow router | 1 | `workflows/_workflow_router/` |
| Software-tool records | 163 | `tools/` |
| Source papers | 28 | `corpus.yaml` |
| Indexes | 4 | `skills_index.json`, `tools_index.json`, `workflows/workflows_index.json`, `kb_bundle.json` |
| Offline search script | 1 | `bin/search_skills.py` |

The build and gate receipts are in `receipts/c0107b2c545e/` at the repository root.

Composite workflows chain leaf skills into a typed pipeline:

| Workflow | Use when | Stages |
|---|---|--:|
| `immunopeptidomics-deep-rescoring` | MGF immunopeptidomics spectra must be searched without enzyme specificity, augmented with run-specific retention-time and predicted-spectrum features, rescored, and controlled at both PSM and peptide false-discovery levels | 5 |
| `isobaric-reporter-quantification` | TMT, TMTpro, or iTRAQ mzML runs must be searched, quantified at the reporter-ion level, filtered for co-isolation interference, aggregated to proteins, and tested for condition-level abundance changes | 5 |

## Licence tiers

Each skill carries a `license_tier` that answers one question: what may you do with the underlying tool?
The tiers are defined in the registry's [licence-tier policy](https://github.com/HolobiomicsLab/asb-skill-collections/blob/main/governance/LICENSE_TIERS.md).
The tier is advice about the tool. It does not change the licence of this collection's content.

| Tier | Skills |
|---|--:|
| `open` | 224 |
| `noncommercial` | 69 |
| `restricted` | 0 |
| `unknown` | 265 |

## Install

Claude Code, from the directory that holds your clone of this repository:

```bash
claude plugin marketplace add ./asb-skills-proteomics
claude plugin install proteomics@asb-skills-proteomics
```

Other agents: every skill is one Markdown file with YAML front matter. Point the agent at `collections/proteomics/v1/leaves/`, or let it call the search script below.

### Only a few skills

Each leaf directory holds a single `SKILL.md`, which does not refer to other files of the collection. Copy the leaves you need, from the directory that holds your clone. Claude Code reads the skills of a project from `<project>/.claude/skills/`:

```bash
mkdir -p <project>/.claude/skills
cp -R asb-skills-proteomics/collections/proteomics/v1/leaves/isobaric-reporter-ion-quantification <project>/.claude/skills/
```

The search script below prints the slug of each match.

## Search offline

The search script needs only the Python standard library.

```bash
cd asb-skills-proteomics
python3 collections/proteomics/v1/bin/search_skills.py --query "TMT reporter ion quantification" -k 3
python3 collections/proteomics/v1/bin/search_skills.py --target workflows --query "TMT reporter ion quantification" -k 2
```

Each result prints its rank and slug, then its description, tools, licence tier and the path of the file to read. The ranked slugs:

```text
1. isobaric-reporter-ion-quantification
2. reporter-ion-interference-assessment
3. isobaric-quantification-cross-tool-benchmarking

1. isobaric-reporter-quantification
```

## How it was built

The AgenticScienceBuilder pipeline read each paper and its code repository, wrote the skills, and passed the collection through a strict release gate. `receipts/c0107b2c545e/source.json` names the pipeline commit `da8abd15`, the collections-tooling commit `b8c22a6bc` and the corpus commit `9f15448`.

The gate checks that every included paper has an allowed access type, caps the length and similarity of verbatim quotes, scans quote spans for personal data and dual-use content, requires provenance and a licence tag on every skill, resolves each composite workflow to real leaves, and binds the receipt to the hashes of the collection files.

22 of the 28 included papers are open access according to `corpus.yaml` (`access.is_oa`). The other 6 are closed access and listed as `link-only`.

Each leaf names its source paper in `provenance.source_papers` (558 of 558 leaves). 282 of the 558 quote it, or its repository README, in their `## Evidence` section. In the other 276, the gate parsed no quote span (`gate_report.json`, `strip_verbatim_similarity`, `coverage.files_without_spans`). In 273 of them the section is empty or its quotes are withheld because they could not be anchored in the source. In 2 the section holds a quote in a form the gate parser does not read. 1 has no `## Evidence` section.

## Related repositories

- [ASB skill collections](https://github.com/HolobiomicsLab/asb-skill-collections): the registry of every domain, the shared tooling, the governance documents, the contribution loop and the earlier metabolomics v2 tree. The current metabolomics collection (v3) is in the `asb-skills-metabolomics` repository.
- AgenticScienceBuilder: the pipeline that generated this collection. Repository follows.
- `indicium-adapters-proteomics`: the Indicium domain adapter for the same domain. Repository follows.

## Citation

Cite the collection with [`CITATION.cff`](CITATION.cff). Cite also the source paper of each skill you use: its DOI is in `provenance.source_papers` in the skill's front matter. The collection's version DOI is [10.5281/zenodo.24224624](https://doi.org/10.5281/zenodo.24224624); its concept DOI, which covers all versions, is [10.5281/zenodo.24093113](https://doi.org/10.5281/zenodo.24093113).

## Licence

Code (the search script and other scripts): Apache-2.0, see [`LICENSE`](LICENSE). Collection content (`SKILL.md` files, tool records, indexes, metadata): CC-BY-4.0, see [`LICENSING.md`](LICENSING.md). Short verbatim quotes from source papers are reproduced for attribution under quotation right. A rights holder can ask for removal through an issue.
