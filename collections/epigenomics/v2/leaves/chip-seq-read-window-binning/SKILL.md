---
name: chip-seq-read-window-binning
description: Use when preparing mapped ChIP-seq (or control) BED files for broad-domain
  calling with RECOGNICER, i.e. when the analysis requires converting raw read coordinates
  into nucleosome-resolution (default 200 bp) binary blocks before recursive block
  transformation.
when_to_use_negative:
- The task is peak calling for sharp, punctate transcription-factor signals rather
  than broad domains (kb-to-Mb H3K27me3/H3K9me3-style enrichment), where local-statistical
  peak callers are the article's stated contrast
- Input data are not mapped reads in BED format (e.g. alignment BAM files that have
  not been converted, or already-binned signal tracks), since RECOGNICER v1.0 expects
  two BED files
- Multiple RECOGNICER instances must run from the same directory, since temporary
  files have hard-coded names and parallel runs from one directory would produce absurd
  outcomes — use separate directories instead
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: RECOGNICER
  role: Tool whose preprocessing stage performs the 200 bp window partitioning, 75
    bp 5'-end shift, redundancy filtering, and binary block designation
  repo: github.com/zanglab/recognicer
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:42190d55dfa7cccdd608018079f11c2c9c8c61d2eb1604653165b8d829990f1b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - RECOGNICER
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# chip-seq-read-window-binning

## Summary

Preprocessing step of RECOGNICER that converts mapped ChIP-seq reads in BED format into binary ('0'/'1') 200 bp genomic blocks by shifting single-end read 5' ends by half the DNA fragment size and thresholding window read counts against the random-placement average, providing the finest-scale input for downstream coarse-graining.

## When to use

Use when preparing mapped ChIP-seq (or control) BED files for broad-domain calling with RECOGNICER, i.e. when the analysis requires converting raw read coordinates into nucleosome-resolution (default 200 bp) binary blocks before recursive block transformation.

## When NOT to use

- The task is peak calling for sharp, punctate transcription-factor signals rather than broad domains (kb-to-Mb H3K27me3/H3K9me3-style enrichment), where local-statistical peak callers are the article's stated contrast
- Input data are not mapped reads in BED format (e.g. alignment BAM files that have not been converted, or already-binned signal tracks), since RECOGNICER v1.0 expects two BED files
- Multiple RECOGNICER instances must run from the same directory, since temporary files have hard-coded names and parallel runs from one directory would produce absurd outcomes — use separate directories instead

## Inputs

- ChIP-seq mapped reads file in BED format (ChIP sample)
- Control sample mapped reads file in BED format
- Configuration parameters in RECOGNICER.sh: SAMPLE name, SPECIES/genome version (e.g. hg18), OUT_DIR, optional WINDOW_SIZE (default 200), FRAGMENT_SIZE (default 150), GENOME_FRACTION (default 0.74 for human), CHIPTHRESHOLD (default 1)

## Outputs

- Binary '0'/'1' block map per chromosome at 200 bp resolution (input to recursive block transformation)
- $SAMPLE-nonredundant.bed (non-redundant reads filtered from the input ChIP dataset)
- $SAMPLE-W200.bedgraph (read pile-up track at 200 bp window resolution in bedGraph format for genome browser visualization)

## How to apply

Load the ChIP and control mapped-read BED files and partition each chromosome into equal-sized non-overlapping 200 bp windows (the WINDOW_SIZE default, matching mononucleosome+linker). Shift each single-end read from its 5' start location downstream by half the DNA fragment size (FRAGMENT_SIZE default 150 bp for mono-nucleosomal fragments, so a 75 bp shift) to place the read at the center of the DNA fragment it represents. Place reads into windows and allow at most CHIPTHRESHOLD copies of identical reads (default 1) to remove redundancy. Designate windows whose read counts exceed the average expected under random placement as '1' blocks; all others are '0' blocks. The resulting binary block representation at the finest scale, plus the non-redundant read file $SAMPLE-nonredundant.bed and the 200 bp-resolution pile-up track $SAMPLE-W200.bedgraph, feed directly into RECOGNICER's recursive block transformation (factor L=3, majority rule).

## Related tools

- **RECOGNICER** (Tool whose preprocessing stage performs the 200 bp window partitioning, 75 bp 5'-end shift, redundancy filtering, and binary block designation) — github.com/zanglab/recognicer

## Examples

```
sh RECOGNICER.sh ChIP.bed control.bed 0.01
```

## Evaluation signals

- The non-redundant file $SAMPLE-nonredundant.bed exists and contains no more than CHIPTHRESHOLD (default 1) copies of any identical read
- $SAMPLE-W200.bedgraph parses as valid bedGraph and its signal bins are 200 bp wide, matching the WINDOW_SIZE default
- Read coordinates in intermediate files reflect a downstream shift of half FRAGMENT_SIZE (75 bp for the 150 bp default) from the 5' ends of single-end reads
- '1' block counts per chromosome are consistent with windows above the random-placement average threshold, and the binary blocks at the finest scale are recovered/used by the subsequent L=3 block transformation rounds
- Subsequent RECOGNICER stages run to completion, yielding $SAMPLE.cgsummary and $SAMPLE-fdr0.01_broadPeak.bed without path or genome errors, indicating the binning stage populated expected chromosome lengths from GenomeData.py

## Limitations

- Supported genomes are limited to those in GenomeData.py (human hg38/hg19/hg18, mouse mm10/mm9/mm8, rat rn4, drosophila dm2/dm3, yeast sacCer1/pombe, Arabidopsis tair8); other species require manual edits to add chrom lists and lengths
- GENOME_FRACTION (effective mappable genome fraction, default 0.74 for human) depends on read length and must be adjusted for non-default read lengths
- Parallel RECOGNICER instances from the same directory interfere through hard-coded temporary file names
- Python 2 implementation with scipy prerequisite; tuning parameters beyond WINDOW_SIZE, FRAGMENT_SIZE, GENOME_FRACTION, and CHIPTHRESHOLD is strongly not recommended

## Evidence

- [methods] partition each chromosome into equal-sized non-overlapping 200 bp windows, shift single-end read 5' coordinates downstream by half the DNA fragment size: [evidence span withheld — anchor status: not_found]
