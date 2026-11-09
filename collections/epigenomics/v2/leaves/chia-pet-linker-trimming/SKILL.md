---
name: chia-pet-linker-trimming
description: Use when raw ChIA-PET paired-end FASTQ reads contain bridge-linker sequences
  at the 5' ends and you need linker-trimmed reads before alignment, or when quantifying
  how much the valid PET ratio improves with mismatch tolerance.
when_to_use_negative:
- The reads are already linker-trimmed and available as SAM/BAM or bedpe (proceed
  with buildBedpe instead)
- The goal is loop calling from an existing deduplicated bedpe (use bedpe2Interaction
  and MICC2.R rather than re-trimming)
- The input is single-end or non-ChIA-PET data without the bridge-linker structure
  at read 5' ends
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PET2
  role: Provides trimLinker (and buildBedpe, removeDup, buildTagAlign, bedpe2Interaction,
    MICC2.R) for trimming linkers and the rest of the pipeline
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: bwa
  role: Aligns the linker-trimmed reads to the reference genome in the next pipeline
    step
  repo: https://github.com/lh3/bwa
- name: MACS2
  role: Calls anchor peaks (q-value 0.05) on buildTagAlign output derived from the
    trimmed, deduplicated PETs
  repo: https://github.com/macs3-project/MACS
- name: bedtools
  role: Bed/bedpe manipulation used to detect interactions between peak anchors (pairToBed
    -type both)
  repo: https://github.com/arq5x/bedtools2
- name: samtools
  role: SAM file handling for the aligned trimmed reads
  repo: https://github.com/samtools/samtools
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:e8810aacc697dc20de8eccd9349fd43840f623498781f3636d538c6ae5d43006
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
  - R
  - bedtools
  - bwa
  - samtools
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# chia-pet-linker-trimming

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trims bridge-linker sequences from ChIA-PET paired-end FASTQ reads using ChIA-PET2's trimLinker, including a 1-mismatch rescue mode that recovers tens of millions of PETs lost under strict 0-mismatch matching.

## When to use

Use when raw ChIA-PET paired-end FASTQ reads contain bridge-linker sequences at the 5' ends and you need linker-trimmed reads before alignment, or when quantifying how much the valid PET ratio improves with mismatch tolerance.

## When NOT to use

- The reads are already linker-trimmed and available as SAM/BAM or bedpe (proceed with buildBedpe instead)
- The goal is loop calling from an existing deduplicated bedpe (use bedpe2Interaction and MICC2.R rather than re-trimming)
- The input is single-end or non-ChIA-PET data without the bridge-linker structure at read 5' ends

## Inputs

- Raw paired-end ChIA-PET FASTQ files (.fastq or .fastq.gz), e.g. GM12878 CTCF from GSE72816/GSM970213
- Bridge-linker sequence (ACGCGATATCTTATCTGACT), split into first-15-base halves linkerA and linkerB

## Outputs

- Linker-trimmed paired-end FASTQ reads ready for bwa alignment
- output.trim.stat summary files with per-step PET counts, rescued PET counts and valid PET ratios

## How to apply

Run ChIA-PET2 trimLinker in bridge-linker mode (-m 1) supplying the first 15 bases of each 5' end of the bridge linker (ACGCGATATCTTATCTGACT) as linkerA=ACGCGATATCTTATC and linkerB=AGTCAGATAAGATAT, with minimum trimmed read length -l 15 and keep-empty -k 0 so reads lacking a linker are discarded. Run it twice for benchmarking: once strict (-e 0) and once allowing one mismatch (-e 1) in the linker-searching step, since allowing one mismatch rescues >16 million PETs and yields a 12.85% improvement in the valid PET ratio on the GM12878 CTCF dataset (GSE72816/GSM970213). Parse the output.trim.stat summary files from both runs to extract rescued PET counts and valid PET ratios. Downstream, the trimmed reads feed into bwa alignment, buildBedpe, removeDup, buildTagAlign, MACS2 peak calling (q-value 0.05) and bedpe2Interaction loop detection with MICC2.R significance estimation (PET cutoff 2).

## Related tools

- **ChIA-PET2** (Provides trimLinker (and buildBedpe, removeDup, buildTagAlign, bedpe2Interaction, MICC2.R) for trimming linkers and the rest of the pipeline) — https://github.com/GuipengLi/ChIA-PET2
- **bwa** (Aligns the linker-trimmed reads to the reference genome in the next pipeline step) — https://github.com/lh3/bwa
- **MACS2** (Calls anchor peaks (q-value 0.05) on buildTagAlign output derived from the trimmed, deduplicated PETs) — https://github.com/macs3-project/MACS
- **bedtools** (Bed/bedpe manipulation used to detect interactions between peak anchors (pairToBed -type both)) — https://github.com/arq5x/bedtools2
- **samtools** (SAM file handling for the aligned trimmed reads) — https://github.com/samtools/samtools

## Examples

```
ChIA-PET2 trimLinker -f reads_1.fastq.gz -r reads_2.fastq.gz -m 1 --linkerA ACGCGATATCTTATC --linkerB AGTCAGATAAGATAT -e 1 -k 0 -l 15
```

## Evaluation signals

- The -e 1 trim.stat summary shows >16 million rescued PETs relative to -e 0 on the GM12878 CTCF dataset, a 12.85% improvement in the valid PET ratio
- Every kept read in the trimmed FASTQ satisfies the minimum trimmed read length of 15 (-l 15)
- Reads without a detectable linker are discarded under the default keep-empty setting (-k 0), and counts in output.trim.stat reconcile with total input PETs
- Downstream buildBedpe with MAPQ 30 and removeDup yields a deduplicated bedpe consistent with the trim.stat valid PET counts

## Limitations

- The mismatch tolerance decision (-e 0 vs -e 1) trades off sensitivity and specificity; users must choose thresholds to balance the trade-off between sensitivity and specificity of downstream loop calling
- The paper does not state a ChIA-PET2 software version, and no changelog was found in the repository, so reproducibility of exact trim.stat counts may depend on the code revision used
- trimLinker's bridge-linker mode as benchmarked assumes the specific bridge linker ACGCGATATCTTATCTGACT; other protocols require appropriate linker definitions

## Evidence
