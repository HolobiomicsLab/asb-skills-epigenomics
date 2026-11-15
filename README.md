<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/brand/asb-logo-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="assets/brand/asb-logo.svg">
    <img alt="AgenticScienceBuilder" src="assets/brand/asb-logo.svg" width="320">
  </picture>
</p>

<h1 align="center">ASB Skills: Epigenomics</h1>

<p align="center"><em>Evidence-grounded skills and tool records for computational epigenomics: DNA methylation, chromatin accessibility and histone-mark analysis.</em></p>

<p align="center">
  <img src="https://img.shields.io/badge/skills-2%2C295-00284b" alt="2,295 skills">
  <img src="https://img.shields.io/badge/tools-532-00284b" alt="532 tools">
  <img src="https://img.shields.io/badge/workflows-2-00284b" alt="2 workflows">
  <img src="https://img.shields.io/badge/papers-92-00284b" alt="92 source papers">
  <img src="https://img.shields.io/badge/content-CC--BY--4.0-04919a" alt="content CC-BY-4.0">
  <img src="https://img.shields.io/badge/code-Apache--2.0-04919a" alt="code Apache-2.0">
  <img src="https://img.shields.io/badge/status-released-04919a" alt="status released">
</p>

---

## TL;DR

This repository holds the epigenomics skill collection of AgenticScienceBuilder (ASB): 2,295 skills over 532 software tools and 2 composite workflows, distilled from 92 method papers and their code repositories. Each skill is a `SKILL.md` file that an agent can load. It names its source paper; 1,669 of the 2,295 also quote it or its repository README. The collection installs as a Claude Code plugin, and a script that needs only the Python standard library searches it offline.

## Collections

| Path | Title | Skills | Tools | Released | DOI |
|---|---|---|---|---|---|
| `collections/epigenomics/v1` | ASB Epigenomics Skill Collection v1 (legacy) | 203 | 68 | 2026-06-15 | not deposited |
| `collections/epigenomics/v2` | ASB Epigenomics Skill Collection v2 | 2295 | 532 | 2026-11-14 | [10.5281/zenodo.24224653](https://doi.org/10.5281/zenodo.24224653) |

## Status

Collection `v2`, built from corpus cut `d7359e4dd5f0`, is released on `main` at tag `epigenomics-v2.1.0` and archived on Zenodo under the version DOI [10.5281/zenodo.24224653](https://doi.org/10.5281/zenodo.24224653). The earlier flat-layout collection `v1` stays in the repository as a legacy collection; it is not part of this release.

| Item | State |
|---|---|
| Release gate | `receipts/d7359e4dd5f0/gate_report.json` reports `pass`: 14 of 16 checks pass and 2 are not applicable (`cut_manifest`, and `peer_review_claim` because the corpus holds no preprint or deposit DOI). [`receipts/d7359e4dd5f0/VERIFY.md`](receipts/d7359e4dd5f0/VERIFY.md) gives the command that re-verifies it. |
| Plugin manifests | `claude plugin validate` passes for `.claude-plugin/marketplace.json` and for `collections/epigenomics/v2` |
| Collection IRI | `https://w3id.org/holobiomicslab/asb-skill/collection/epigenomics/v2` |
| IRI resolution (2026-11-14) | Registered at w3id.org. It redirects (HTTP 302) to the collection landing page in the asb-skill-collections registry (HTTP 200), which names this repository and the Zenodo record. |
| DOI | [10.5281/zenodo.24224653](https://doi.org/10.5281/zenodo.24224653) (this version). The Zenodo deposit metadata is in [`.zenodo.json`](.zenodo.json). |
| Concept DOI | [10.5281/zenodo.24093106](https://doi.org/10.5281/zenodo.24093106) (all versions) |

## What is inside

| Item | Count | Path under `collections/epigenomics/v2/` |
|---|--:|---|
| Leaf skills (`SKILL.md`) | 2,295 | `leaves/<slug>/SKILL.md` |
| Skill router | 1 | `skills/_router/` |
| Composite workflows | 2 | `workflows/<slug>/` |
| Workflow router | 1 | `workflows/_workflow_router/` |
| Software-tool records | 532 | `tools/` |
| Source papers | 92 | `corpus.yaml` |
| Indexes | 4 | `skills_index.json`, `tools_index.json`, `workflows/workflows_index.json`, `kb_bundle.json` |
| Offline search script | 1 | `bin/search_skills.py` |

The build and gate receipts are in `receipts/d7359e4dd5f0/` at the repository root.

Composite workflows chain leaf skills into a typed pipeline:

| Workflow | Use when | Stages |
|---|---|--:|
| `bisulfite-dmr-analysis` | paired-end whole-genome bisulfite sequencing reads must be mapped, reduced to six-column per-CpG methylated and unmethylated count tables, aggregated into regional total and methylated counts, and tested between externally defined groups | 4 |
| `chipseq-peak-motif-discovery` | paired ChIP-seq immunoprecipitation and control FASTQ reads must be aligned and quality-assessed, converted into control-aware peaks, and analyzed for over-represented sequence motifs against an explicit matched background | 4 |

## Licence tiers

Each skill carries a `license_tier` that answers one question: what may you do with the underlying tool?
The tiers are defined in the registry's [licence-tier policy](https://github.com/HolobiomicsLab/asb-skill-collections/blob/main/governance/LICENSE_TIERS.md).
The tier is advice about the tool. It does not change the licence of this collection's content.

| Tier | Skills |
|---|--:|
| `open` | 1,571 |
| `noncommercial` | 577 |
| `restricted` | 17 |
| `unknown` | 130 |

## Install

Claude Code, from the directory that holds your clone of this repository:

```bash
claude plugin marketplace add ./asb-skills-epigenomics
claude plugin install epigenomics@asb-skills-epigenomics
```

Other agents: every skill is one Markdown file with YAML front matter. Point the agent at `collections/epigenomics/v2/leaves/`, or let it call the search script below.

### Only a few skills

Each leaf directory holds a single `SKILL.md`, which does not refer to other files of the collection. Copy the leaves you need, from the directory that holds your clone. Claude Code reads the skills of a project from `<project>/.claude/skills/`:

```bash
mkdir -p <project>/.claude/skills
cp -R asb-skills-epigenomics/collections/epigenomics/v2/leaves/chip-seq-peak-calling--8607454362b9598f <project>/.claude/skills/
```

The search script below prints the slug of each match.

## Search offline

The search script needs only the Python standard library.

```bash
cd asb-skills-epigenomics
python3 collections/epigenomics/v2/bin/search_skills.py --query "ChIP-seq peak calling" -k 3
python3 collections/epigenomics/v2/bin/search_skills.py --target workflows --query "ChIP-seq peak calling" -k 2
```

Each result prints its rank and slug, then its description, tools, licence tier and the path of the file to read. The ranked slugs:

```text
1. chip-seq-peak-calling--8607454362b9598f
2. chip-seq-peak-calling--9572d2e9701b16d4
3. chip-seq-peak-calling-with-control

1. chipseq-peak-motif-discovery
```

## How it was built

The AgenticScienceBuilder pipeline read each paper and its code repository, wrote the skills, and passed the collection through a strict release gate. `receipts/d7359e4dd5f0/source.json` names the pipeline commit `da8abd15`, the collections-tooling commit `b8c22a6bc` and the corpus commit `9f15448`.

The gate checks that every included paper has an allowed access type, caps the length and similarity of verbatim quotes, scans quote spans for personal data and dual-use content, requires provenance and a licence tag on every skill, resolves each composite workflow to real leaves, and binds the receipt to the hashes of the collection files.

All 92 included papers are open access according to `corpus.yaml` (`access.is_oa`).

Each leaf names its source paper in `provenance.source_papers` (2,295 of 2,295 leaves). 1,669 of the 2,295 quote it, or its repository README, in their `## Evidence` section. In the other 626, the gate parsed no quote span (`gate_report.json`, `strip_verbatim_similarity`, `coverage.files_without_spans`). In 621 of them the section is empty or its quotes are withheld because they could not be anchored in the source. In 3 the section holds a quote in a form the gate parser does not read. 2 have no `## Evidence` section.

## Related repositories

- [ASB skill collections](https://github.com/HolobiomicsLab/asb-skill-collections): the registry of every domain, the shared tooling, the governance documents, the contribution loop and the earlier metabolomics v2 tree. The current metabolomics collection (v3) is in the `asb-skills-metabolomics` repository.
- AgenticScienceBuilder: the pipeline that generated this collection. Repository follows.
- [`indicium-adapters-epigenomics`](https://github.com/HolobiomicsLab/indicium-adapters-epigenomics): the Indicium domain adapter for the same domain.

## Citation

Cite the collection with [`CITATION.cff`](CITATION.cff). Cite also the source paper of each skill you use: its DOI is in `provenance.source_papers` in the skill's front matter. The collection's version DOI is [10.5281/zenodo.24224653](https://doi.org/10.5281/zenodo.24224653); its concept DOI, which covers all versions, is [10.5281/zenodo.24093106](https://doi.org/10.5281/zenodo.24093106).

## Licence

Code (the search script and other scripts): Apache-2.0, see [`LICENSE`](LICENSE). Collection content (`SKILL.md` files, tool records, indexes, metadata): CC-BY-4.0, see [`LICENSING.md`](LICENSING.md). Short verbatim quotes from source papers are reproduced for attribution under quotation right. A rights holder can ask for removal through an issue.
