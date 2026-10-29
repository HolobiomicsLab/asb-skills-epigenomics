---
name: paired-end-tag-fraction-computation
description: Use when processing ChIA-PET fastq data through ChIA-PET2 and you need
  to quantify per-step PET retention (pre-/post-alignment, post-duplicate-removal,
  intra-chromosomal >=4 PETs) for quality control.
when_to_use_negative:
- Data is not ChIA-PET (e.g., Hi-C or single-end ChIP-Seq reads); PET-pair statistics
  and bridge-linker/half-linker mode selection do not apply
- You need statistically significant loop calling rather than data quality metrics;
  use bedpe2Interaction or MICC2.R outputs instead of QCplots.R
- Raw reads have not yet been processed through ChIA-PET2 steps 1-5, so no per-step
  statistics files exist to aggregate
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PET2
  role: Pipeline whose steps 1-5 generate the per-step statistics files consumed for
    PET fraction computation
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: QCplots.R
  role: R/ggplot2 module (step 6) that plots the Quality Control figure from the stepwise
    statistics
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: R
  role: Runtime environment in which QCplots.R aggregates statistics into the multi-panel
    QC figure
- name: trimLinker
  role: Trims bridge-linker sequences (-l 15 minimum trimmed read length) to produce
    the trim statistics
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: removeDup
  role: Removes duplicate PETs to yield the post-duplicate-removal retention fraction
    (41.08% for GM12878 CTCF)
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: bwa
  role: Aligns trimmed reads with -Q 30 MAPQ cutoff to produce valid-PET statistics
  repo: https://github.com/lh3/bwa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:00be947b906b7ec0ed13d9134c75af171b26a3c52e7b974e5023e062a1d71f29
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
  - bwa
  - removeDup
  - trimLinker
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# paired-end-tag-fraction-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes stepwise paired-end tag (PET) retention fractions from ChIA-PET2 per-step statistics files and aggregates them into the multi-panel QC plot, quantifying how many PETs survive linker trimming, alignment (MAPQ 30), and duplicate removal.

## When to use

Use when processing ChIA-PET fastq data through ChIA-PET2 and you need to quantify per-step PET retention (pre-/post-alignment, post-duplicate-removal, intra-chromosomal >=4 PETs) for quality control.

## When NOT to use

- Data is not ChIA-PET (e.g., Hi-C or single-end ChIP-Seq reads); PET-pair statistics and bridge-linker/half-linker mode selection do not apply
- You need statistically significant loop calling rather than data quality metrics; use bedpe2Interaction or MICC2.R outputs instead of QCplots.R
- Raw reads have not yet been processed through ChIA-PET2 steps 1-5, so no per-step statistics files exist to aggregate

## Inputs

- Bridge-linker ChIA-PET paired-end fastq(.gz) files (e.g., GM12878 CTCF dataset)
- ChIA-PET2 per-step statistics files: prefixname.trim.stat, prefixname_1.valid.sam, prefixname.bedpe.stat

## Outputs

- Stepwise PET retention fractions: valid PETs pre-alignment, valid PETs post-alignment, PETs retained after duplicate removal, intra-chromosomal interactions at >=4 PETs
- prefixname.QCplot.pdf multi-panel quality-control figure

## How to apply

Run the GM12878 CTCF bridge-linker dataset through ChIA-PET2 steps 1-5 in bridge-linker mode (-m 1): trimLinker (-l 15 minimum trimmed read length), bwa alignment (-Q 30 MAPQ cutoff), buildBedpe, removeDup, and bedpe2Interaction, so that the per-step statistics files (prefixname.trim.stat, prefixname_1.valid.sam, prefixname.bedpe.stat) are generated. Then invoke the QC plotting module (step 6) via 'Rscript QCplots.R directory name' so ggplot2 aggregates the stepwise statistics into the multi-panel quality-control figure. From the generated statistics, compute the stepwise fractions: valid PETs before alignment, valid PETs after alignment, PETs remaining after duplicate removal, and the fraction of intra-chromosomal interactions supported by >=4 PETs. Correct application is judged by whether the computed fractions reproduce the reported values (89.95% valid PETs pre-alignment, 85% post-alignment, 41.08% after duplicate removal, 92.12% intra-chromosomal at >=4 PETs) and whether prefixname.QCplot.pdf is produced.

## Related tools

- **ChIA-PET2** (Pipeline whose steps 1-5 generate the per-step statistics files consumed for PET fraction computation) — https://github.com/GuipengLi/ChIA-PET2
- **QCplots.R** (R/ggplot2 module (step 6) that plots the Quality Control figure from the stepwise statistics) — https://github.com/GuipengLi/ChIA-PET2
- **R** (Runtime environment in which QCplots.R aggregates statistics into the multi-panel QC figure)
- **trimLinker** (Trims bridge-linker sequences (-l 15 minimum trimmed read length) to produce the trim statistics) — https://github.com/GuipengLi/ChIA-PET2
- **removeDup** (Removes duplicate PETs to yield the post-duplicate-removal retention fraction (41.08% for GM12878 CTCF)) — https://github.com/GuipengLi/ChIA-PET2
- **bwa** (Aligns trimmed reads with -Q 30 MAPQ cutoff to produce valid-PET statistics) — https://github.com/lh3/bwa

## Examples

```
Rscript QCplots.R directory name
```

## Evaluation signals

- prefixname.QCplot.pdf is produced by the QCplots.R invocation
- Valid PETs before alignment equals 89.95% for the GM12878 CTCF bridge-linker dataset
- PETs retained after filtering unmapped and low-MAPQ reads equals 85% of input PETs
- PETs remaining after duplicate removal equals 41.08% of PETs for MACS2 peak calling
- Intra-chromosomal interactions supported by >=4 PETs equals 92.12%

## Limitations

- QC fractions are dataset- and parameter-dependent (MAPQ 30 cutoff, -l 15 trim length); values will differ for other datasets or thresholds
- ChIA-PET2 loop calling depends on ChIP-Seq-like peak calling sensitivity, so post-duplicate-removal PET counts feed into a strategy that assumes loop anchors lie at ChIP-Seq-like peaks
- No software versions are stated for ChIA-PET2 or its dependencies (bwa, samtools, bedtools, MACS2, g++), so exact reproduction may vary

## Evidence

- [other] For the GM12878 CTCF dataset, after filtering unmapped and low-MAPQ reads, 85% of input PETs were retained; after duplicate removal, 41.08% of PETs were retained for MACS2 peak calling.: [evidence span withheld — anchor status: not_found]
- [other] verify that prefixname.QCplot.pdf is produced and that the computed stepwise fractions reproduce the reported values (89.95% valid PETs pre-alignment, 85% post-alignment, 41.08% after duplicate removal, 92.12% intra-chromosomal at >=4 PETs).: [evidence span withheld — anchor status: not_found]
