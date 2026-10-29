---
name: genomic-interval-aggregation
description: Use when a broad-domain ChIP-seq analysis needs mapped BED reads binned
  into fixed windows (e.g., 200 bp) with 5'-end shifting (e.g., 75 bp for 150 bp fragments)
  before multi-scale enrichment detection.
when_to_use_negative:
- 'The goal is sharp/pointed peak calling rather than broad domains: existing peak
  callers based on local statistical models without multi-scale features are the usual
  choice, and RECOGNICER''s windowed coarse-graining targets kb-to-Mb domains like
  H3K27me3/H3K9me3'
- The input is raw sequencing data (FASTQ) rather than mapped reads in BED format;
  RECOGNICER requires mapped-read BED files
- Two RECOGNICER instances would share the same working directory, since temporary
  files with hard-coded names would interfere and yield absurd outcomes
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: RECOGNICER
  role: Implements chromosome window partitioning, 5'-end fragment shifting, and windowed
    read pile-up as the front end of the coarse-graining broad-domain caller
  repo: github.com/zanglab/recognicer
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:3f64f964bc3c3f97c335fdb48e365dd54d0513a449ee77f0881c5bed21fced3b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - RECOGNICER
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# genomic-interval-aggregation

## Summary

Partitions each chromosome into equal-sized, non-overlapping windows and aggregates mapped ChIP-seq reads into them, shifting each single-end read from its 5' end by half the DNA fragment size; this windowed pile-up is the substrate for RECOGNICER's coarse-graining of broad domains such as H3K27me3.

## When to use

Use when a broad-domain ChIP-seq analysis needs mapped BED reads binned into fixed windows (e.g., 200 bp) with 5'-end shifting (e.g., 75 bp for 150 bp fragments) before multi-scale enrichment detection.

## When NOT to use

- The goal is sharp/pointed peak calling rather than broad domains: existing peak callers based on local statistical models without multi-scale features are the usual choice, and RECOGNICER's windowed coarse-graining targets kb-to-Mb domains like H3K27me3/H3K9me3
- The input is raw sequencing data (FASTQ) rather than mapped reads in BED format; RECOGNICER requires mapped-read BED files
- Two RECOGNICER instances would share the same working directory, since temporary files with hard-coded names would interfere and yield absurd outcomes

## Inputs

- ChIP-seq mapped-read BED file (ChIP signal sample)
- ChIP-seq mapped-read BED file (control sample)
- WINDOW_SIZE parameter in bp (default 200)
- FRAGMENT_SIZE parameter in bp (default 150)
- GENOME_FRACTION and CHIPTHRESHOLD configuration values

## Outputs

- $SAMPLE-nonredundant.bed (non-redundant reads after copy filtering)
- $SAMPLE-W200.bedgraph (windowed read pile-up track in bedGraph format)
- $SAMPLE.cgsummary (candidate domains with ChIP/control counts, P-value, fold change, Q-value)
- $SAMPLE-fdr0.01_broadPeak.bed (final significant domains in ENCODE broadPeak format)

## How to apply

Partition each chromosome into equal-sized, non-overlapping windows of WINDOW_SIZE (RECOGNICER default 200 bp, i.e., mononucleosome+linker resolution; the paper's parameter sweep extends up to 1 kb). For each single-end read, determine the shift from the 5' start location toward the center of the represented DNA fragment using FRAGMENT_SIZE (default 150 bp, so shift = 75 bp), then place the shifted read into the overlapping window to build a read pile-up. Optionally restrict to non-redundant reads first, honoring CHIPTHRESHOLD (default 1 copy of identical reads) and GENOME_FRACTION (default 0.74 for human). Emit the aggregation as a bedGraph-format pile-up track at window-size resolution, which then feeds RECOGNICER's recursive block transformation (factor L=3 with phase optimization), domain retrieval, and Poisson-model significance testing with Benjamini-Hochberg FDR control.

## Related tools

- **RECOGNICER** (Implements chromosome window partitioning, 5'-end fragment shifting, and windowed read pile-up as the front end of the coarse-graining broad-domain caller) — github.com/zanglab/recognicer

## Examples

```
sh RECOGNICER.sh ChIP.bed control.bed 0.01
```

## Evaluation signals

- The pile-up bedGraph has exactly window-size (e.g., 200 bp) resolution and spans the chromosomes listed in GenomeData.py for the chosen genome version
- In the parameter sweep, total identified domain length and total read count within domains remain approximately stable across fragment-size (150 bp default) and window-size (200 bp to 1 kb) settings (Fig. 3E, 3F)
- After downsampling from 17.23M to 4M reads, total aggregate domain length and the FRIP score remain approximately invariant
- The $SAMPLE.cgsummary file contains chrom, start, end, ChIP count, control count, P-value, fold change, Q-value columns, and the final broadPeak.bed is valid ENCODE broadPeak format with fold change as score and ChIP count as signalValue
- Auto-correlation length behaves as expected during coarse-graining: ~100 kb constant for synthetic Chromosome 1 data, growing for real ChIP-seq data

## Limitations

- RECOGNICER may falsely merge two domains separated by fine-scale gaps into one coarse-scale domain; users should monitor individual cases and may incorporate featured or masked regions
- Running multiple RECOGNICER instances from the same directory in parallel causes hard-coded temporary files to interfere, resulting in absurd outcomes
- The result resolution is tied to WINDOW_SIZE (default 200 bp), so sub-window localization of boundaries is not available; only listed genome versions in GenomeData.py are supported unless customized

## Evidence

- [results] RECOGNICER is robust to the DNA fragment size and Initial window size parameters: the total length of identified domains and total read count in domains do not change appreciably when these parameters are varied: [evidence span withheld — anchor status: not_found]
