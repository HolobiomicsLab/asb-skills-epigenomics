---
name: cross-tool-chromatin-loop-reproducibility-comparison
description: Use when comparing chromatin loop callers on identical ChIA-PET replicates
  — e.g. quantifying whether one pipeline's loop calls are more reproducible between
  replicates or recover another tool's calls.
when_to_use_negative:
- Data are bridge-linker ChIA-PET protocol reads (Mango was designed only for half-linker
  ChIA-PET data processing)
- You need allele-specific interaction calling rather than cross-tool reproducibility
  benchmarking
- Only one replicate is available — the Jaccard inter-replicate metric is undefined
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0607
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PET2
  role: Primary pipeline run in A/B-linker (-m 0) half-linker mode through MICC loop
    calling
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: Mango
  role: Comparison pipeline for significant half-linker ChIA-PET loop calling on the
    same replicates
- name: bwa
  role: Read alignment step inside the ChIA-PET2 workflow
  repo: https://github.com/lh3/bwa
- name: samtools
  role: SAM/BAM handling within the ChIA-PET2 pipeline
  repo: https://github.com/samtools/samtools
- name: bedtools
  role: bed/bedpe file manipulation; chromosome-size file prepared for it
  repo: https://github.com/arq5x/bedtools2
- name: MACS2
  role: Peak calling on tagAlign input with default '-q 0.05'
  repo: https://github.com/macs3-project/MACS
- name: R
  role: Implementation language for MICC2.R significance estimation and QC plots
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:586a1625ee21f6cc36d6caf721339c93bfc4b0c3a3c892877e59a4808e8145ac
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
  - MACS2
  - Mango
  - R
  - bedtools
  - bwa
  - samtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0607
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# cross-tool-chromatin-loop-reproducibility-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark two ChIA-PET loop-calling pipelines (ChIA-PET2 vs Mango) on the same half-linker dataset by running each end-to-end on identical replicates and quantifying inter-replicate reproducibility (Jaccard index) and cross-tool recovery of called loops.

## When to use

Use when comparing chromatin loop callers on identical ChIA-PET replicates — e.g. quantifying whether one pipeline's loop calls are more reproducible between replicates or recover another tool's calls.

## When NOT to use

- Data are bridge-linker ChIA-PET protocol reads (Mango was designed only for half-linker ChIA-PET data processing)
- You need allele-specific interaction calling rather than cross-tool reproducibility benchmarking
- Only one replicate is available — the Jaccard inter-replicate metric is undefined

## Inputs

- Raw paired-end FASTQ files for two K562 POL2 half-linker ChIA-PET replicates (GEO GSE72816)
- bwa genome index
- Tab-delimited chromosome-size file for bedtools

## Outputs

- chia_pet2_vs_mango_comparison.tsv with per-pipeline loop counts, inter-replicate Jaccard indices, and Mango-loop recovery rates
- ChIA-PET2 outputs per replicate: interactions.intra.bedpe, interactions.inter.bedpe, interactions.MICC
- Mango significant loop calls per replicate

## How to apply

Run each pipeline end-to-end on the same raw paired-end FASTQ replicates so the comparison is isolated to loop calling. For ChIA-PET2 use A/B-linker mode (-m 0) with linker trimming, bwa alignment, PET construction at MAPQ cutoff 30, removeDup PCR duplicate removal, MACS2 peak calling with default '-q 0.05', and MICC significance estimation (default CUTOFFPET=2), yielding interactions.intra.bedpe, interactions.inter.bedpe, and interactions.MICC files. Run Mango on the same replicates for half-linker significant loop calling. Count significant loops per replicate per pipeline, compute the inter-replicate Jaccard index of called loop sets per pipeline, and compute the fraction of Mango loops recovered by ChIA-PET2 per replicate; write results to a TSV (e.g. chia_pet2_vs_mango_comparison.tsv). Rationale: Mango is 'too conservative' at significant loop calling, so reproducibility (Jaccard) and sensitivity (recovery rate) on the same inputs directly expose this divergence.

## Related tools

- **ChIA-PET2** (Primary pipeline run in A/B-linker (-m 0) half-linker mode through MICC loop calling) — https://github.com/GuipengLi/ChIA-PET2
- **Mango** (Comparison pipeline for significant half-linker ChIA-PET loop calling on the same replicates)
- **bwa** (Read alignment step inside the ChIA-PET2 workflow) — https://github.com/lh3/bwa
- **samtools** (SAM/BAM handling within the ChIA-PET2 pipeline) — https://github.com/samtools/samtools
- **bedtools** (bed/bedpe file manipulation; chromosome-size file prepared for it) — https://github.com/arq5x/bedtools2
- **MACS2** (Peak calling on tagAlign input with default '-q 0.05') — https://github.com/macs3-project/MACS
- **R** (Implementation language for MICC2.R significance estimation and QC plots)

## Examples

```
ChIA-PET2.sh -f rep1.fastq.gz -z bwa_index --chromsize chrom.sizes -m 0 --mapq 30 --macs2_q 0.05 --cutoffpet 2 -o rep1_chiapet2
```

## Evaluation signals

- ChIA-PET2 inter-replicate Jaccard index of loop sets reaches ~52.5% vs ~25.6% for Mango (within reasonable tolerance)
- ChIA-PET2 reproduces >9,000 significant loops per replicate versus <300 for Mango
- Fraction of Mango-called loops recovered by ChIA-PET2 matches ~64% for replicate 1 and ~74% for replicate 2
- Each ChIA-PET2 run produces the expected outputs: interactions.intra.bedpe, interactions.inter.bedpe, interactions.MICC
- Both pipelines consumed identical raw FASTQ replicates so differences are attributable to the loop-calling step, not inputs

## Limitations

- Mango is too conservative at the significant loop calling step, giving low reproducibility between replicates (~25.6% Jaccard)
- Mango was designed only for half-linker ChIA-PET data processing and cannot handle bridge-linker/long-read protocol datasets
- ChIA-PET2 loop calling depends on ChIP-Seq-like peak calling sensitivity (anchor-peak assumption)
- No software version stated for ChIA-PET2 or its dependencies (bwa, samtools, bedtools, MACS2, g++), complicating exact reproduction

## Evidence
