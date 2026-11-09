---
name: chia-pet-quality-control-metrics
description: Use when processing bridge-linker or half-linker ChIA-PET FASTQ data
  through ChIA-PET2 and you need stepwise PET-retention statistics (valid PETs pre/post-alignment,
  post-duplicate-removal fraction, intra-chromosomal >=4 PET fraction) to judge dataset
  quality.
when_to_use_negative:
- The data are not ChIA-PET raw reads/pipeline outputs — e.g. already-called loops
  or a feature table — since QCplots.R expects ChIA-PET2 per-step statistics files
- You need statistically scored chromatin loops rather than QC plotting; use bedpe2Interaction/MICC2.R
  instead, as ChiaSig and MICC 'focus on significant loop calling which is only a
  part of the data analysis workflow'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PET2
  role: Pipeline producing the per-step statistics (trimLinker, buildBedpe, removeDup,
    bedpe2Interaction) that feed the QC plot
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: QCplots.R
  role: QC plotting module (step 6) that aggregates stepwise statistics into the multi-panel
    QC figure
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: R
  role: Runtime for QCplots.R; ChIA-PET2 is implemented using C++, R and shell scripts
- name: bwa
  role: Read alignment step whose MAPQ cutoff (-Q 30) determines post-alignment valid
    PET retention
  repo: https://github.com/lh3/bwa
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:0f675ce5d5eb8f9eb9e58c92e500381ed5c8527984f1bf19431fb212b5d4fcd7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/GuipengLi/ChIA-PET2
  repo_url: https://github.com/GuipengLi/ChIA-PET2
  tools:
  - ChIA-PET2
  - QCplots.R
  - R
  - bedtools
  - bwa
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# chia-pet-quality-control-metrics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates and interprets stepwise quality-control metrics for ChIA-PET data processed with ChIA-PET2, aggregating per-step statistics (linker trimming, alignment, duplicate removal, interaction calling) into a multi-panel QC figure via the QCplots.R module.

## When to use

Use when processing bridge-linker or half-linker ChIA-PET FASTQ data through ChIA-PET2 and you need stepwise PET-retention statistics (valid PETs pre/post-alignment, post-duplicate-removal fraction, intra-chromosomal >=4 PET fraction) to judge dataset quality.

## When NOT to use

- The data are not ChIA-PET raw reads/pipeline outputs — e.g. already-called loops or a feature table — since QCplots.R expects ChIA-PET2 per-step statistics files
- You need statistically scored chromatin loops rather than QC plotting; use bedpe2Interaction/MICC2.R instead, as ChiaSig and MICC 'focus on significant loop calling which is only a part of the data analysis workflow'

## Inputs

- Per-step ChIA-PET2 statistics files: prefixname.trim.stat, prefixname_1.valid.sam, prefixname.bedpe.stat
- Bridge-linker ChIA-PET paired-end FASTQ(.gz) reads (e.g. GM12878 CTCF, GSM970213)

## Outputs

- prefixname.QCplot.pdf — multi-panel ggplot2 quality-control figure
- Stepwise PET-retention fractions (valid PETs pre-alignment, post-alignment, post-duplicate-removal, intra-chromosomal >=4 PET fraction)

## How to apply

Run the GM12878 CTCF bridge-linker fastq files through ChIA-PET2 steps 1-5 in bridge-linker mode (-m 1): trimLinker (-l 15 minimum trimmed read length), bwa alignment with a -Q 30 MAPQ cutoff, buildBedpe to construct PETs, removeDup for PCR duplicate removal, and bedpe2Interaction for interaction detection, so the per-step statistics files (prefixname.trim.stat, prefixname_1.valid.sam, prefixname.bedpe.stat) are generated. Then invoke the QC plotting module (step 6) with 'Rscript QCplots.R directory name', which uses ggplot2/R to aggregate the stepwise statistics into the multi-panel quality-control figure (prefixname.QCplot.pdf). Compute the stepwise fractions from the statistics: valid PETs before alignment, valid PETs after alignment, PETs remaining after duplicate removal, and the fraction of intra-chromosomal interactions supported by >=4 PETs. Validation compares these fractions against published reference values (89.95% valid pre-alignment, 85% post-alignment, 41.08% after duplicate removal, 92.12% intra-chromosomal at >=4 PETs).

## Related tools

- **ChIA-PET2** (Pipeline producing the per-step statistics (trimLinker, buildBedpe, removeDup, bedpe2Interaction) that feed the QC plot) — https://github.com/GuipengLi/ChIA-PET2
- **QCplots.R** (QC plotting module (step 6) that aggregates stepwise statistics into the multi-panel QC figure) — https://github.com/GuipengLi/ChIA-PET2
- **R** (Runtime for QCplots.R; ChIA-PET2 is implemented using C++, R and shell scripts)
- **bwa** (Read alignment step whose MAPQ cutoff (-Q 30) determines post-alignment valid PET retention) — https://github.com/lh3/bwa

## Examples

```
Rscript QCplots.R directory name
```

## Evaluation signals

- prefixname.QCplot.pdf is produced without error from 'Rscript QCplots.R directory name'
- Valid PETs pre-alignment fraction reproduces the reported 89.95%
- Post-alignment retention reproduces 85% after filtering unmapped and low-MAPQ reads
- Post-duplicate-removal retention reproduces 41.08% of PETs retained for MACS2 peak calling
- Intra-chromosomal interactions supported by >=4 PETs reproduce 92.12%

## Limitations

- QC reference fractions are tied to the GM12878 CTCF bridge-linker dataset with specific parameters (-l 15, -Q 30, -m 1); other datasets will differ
- No software version of ChIA-PET2 is stated, and no versions stated for required dependencies (bwa, samtools, bedtools, MACS2, g++)
- Only a few examples of the transitivity, redundancy and robustness of chromatin loops were provided

## Evidence

- [other] after filtering unmapped and low-MAPQ reads, 85% of input PETs were retained; after duplicate removal, 41.08% of PETs were retained for MACS2 peak calling: [evidence span withheld — anchor status: not_found]
