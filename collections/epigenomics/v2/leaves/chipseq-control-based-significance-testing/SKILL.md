---
name: chipseq-control-based-significance-testing
description: Use when you have paired ChIP and control mapped-read BED files for a
  broad histone mark (e.g. H3K27me3/H3K9me3, kb-to-Mb domains) and need to call integral
  domains and compare their functional association across methods.
when_to_use_negative:
- Calling sharp, point-source transcription-factor peaks — the method is designed
  for broad domains (kb to Mb) and most existing peak callers handle local narrow
  enrichment
- The input is raw sequencing reads rather than mapped-read BED files, or no matched
  control BED sample is available for the control-based FDR significance testing
- You need a single-domain guarantee where two fine-scale domains separated by gaps
  must never be merged — coarse-graining can falsely merge such domains
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3169
tools:
- name: RECOGNICER
  role: 'Primary broad-domain caller: coarse-graining via recursive block transformation
    with control-based significance (FDR 0.01) to produce ENCODE broadPeak domains'
  repo: https://github.com/zanglab/recognicer
- name: SICER
  role: Comparison method run with default parameters for the repressed-gene coverage
    benchmark
- name: RSEG
  role: HMM-based broad-domain comparison method run with default parameters for the
    benchmark
- name: MUSIC
  role: Multiscale-decomposition comparison method run with default parameters for
    the benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:193668d67e8ba53584c8b57ce9610ba4094aa4c54f1836df8d7eaeddf2d1595b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/zanglab/recognicer
  tools:
  - MUSIC
  - RECOGNICER
  - RSEG
  - SICER
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# chipseq-control-based-significance-testing

## Summary

Calls broad H3K27me3 enrichment domains from ChIP-seq mapped reads using RECOGNICER's coarse-graining algorithm, with input-matched control-based significance (FDR) determination, and benchmarks the resulting domains against SICER, RSEG, and MUSIC on repressed-gene gene-body coverage.

## When to use

Use when you have paired ChIP and control mapped-read BED files for a broad histone mark (e.g. H3K27me3/H3K9me3, kb-to-Mb domains) and need to call integral domains and compare their functional association across methods.

## When NOT to use

- Calling sharp, point-source transcription-factor peaks — the method is designed for broad domains (kb to Mb) and most existing peak callers handle local narrow enrichment
- The input is raw sequencing reads rather than mapped-read BED files, or no matched control BED sample is available for the control-based FDR significance testing
- You need a single-domain guarantee where two fine-scale domains separated by gaps must never be merged — coarse-graining can falsely merge such domains

## Inputs

- ChIP-seq mapped-read BED file for the ChIP signal sample
- ChIP-seq mapped-read BED file for the matched control sample
- Matched gene expression profile (microarray or RNA-seq) for repressed-gene annotation
- Genome version configuration (e.g. hg38, hg19, hg18, mm10, mm9) as supported in GenomeData.py

## Outputs

- $SAMPLE-fdr0.01_broadPeak.bed: significant domains in ENCODE broadPeak format (score = ChIP/control fold change, signalValue = ChIP read count)
- $SAMPLE.cgsummary: candidate domains with chrom, start, end, ChIP read count, control read count, P-value, fold change, Q-value
- $SAMPLE-W200.bedgraph: 200 bp-resolution pile-up track for genome browser visualization
- h3k27me3_domain_coverage_comparison.tsv: per-cell-type, per-method fraction of repressed genes fully covered by a single domain

## How to apply

Run RECOGNICER on the ChIP.bed and control.bed files with the default parameters (FRAGMENT_SIZE=150 bp mononucleosome, WINDOW_SIZE=200 bp mononucleosome+linker, FDR threshold 0.01), producing significant domains in ENCODE broadPeak format ($SAMPLE-fdr0.01_broadPeak.bed). The algorithm coarse-grains each chromosome partitioned into equal-sized non-overlapping windows by recursive block transformations (block transformation factor 3), computes the auto-correlation length at each round to decide termination, retrieves candidate domains with boundary determination, and assigns significance against the control sample. For benchmarking, run SICER, RSEG, and MUSIC on the same BED inputs with their default parameters, classify genes as repressed using matched microarray or RNA-seq expression data, intersect called domains with gene bodies, and compute the fraction of repressed genes whose entire gene body falls within a single domain. RECOGNICER achieves the highest coverage fraction in 8 of 9 ENCODE cell types, which is the expected validation outcome.

## Related tools

- **RECOGNICER** (Primary broad-domain caller: coarse-graining via recursive block transformation with control-based significance (FDR 0.01) to produce ENCODE broadPeak domains) — https://github.com/zanglab/recognicer
- **SICER** (Comparison method run with default parameters for the repressed-gene coverage benchmark)
- **RSEG** (HMM-based broad-domain comparison method run with default parameters for the benchmark)
- **MUSIC** (Multiscale-decomposition comparison method run with default parameters for the benchmark)

## Examples

```
sh RECOGNICER.sh ChIP.bed control.bed 0.01
```

## Evaluation signals

- The final output file $SAMPLE-fdr0.01_broadPeak.bed parses as valid ENCODE broadPeak format, with fold change in column 5 and ChIP read count in column 7
- Identified H3K27me3 domain sizes span kb to Mb, consistent with the expected broad-domain scale
- In the benchmark table, RECOGNICER attains the highest fraction of repressed genes fully covered by a single domain in 8 of 9 ENCODE cell types versus SICER, RSEG, and MUSIC
- Downsampling the CD4+ T cell dataset from 17.23 million to as few as 4 million reads should not destroy domain detection (robustness to sequencing depth)
- Each RECOGNICER instance must be run in a separate directory — parallel runs from the same directory yield absurd outcomes due to hard-coded temporary file names

## Limitations

- RECOGNICER could falsely merge two domains that are separated by gaps on the fine scale into one domain on the coarse scale; users should specifically monitor such cases and can incorporate featured or masked regions
- Implemented in python2 with a scipy prerequisite; only the listed genomes (hg38/hg19/hg18, mm10/mm9/mm8, rn4, dm2/dm3, sacCer1/pombe, tair8) are supported unless GenomeData.py is edited
- Do not run multiple RECOGNICER instances from the same directory in parallel; hard-coded temporary file names cause interference
- The whole run may take several hours per sample

## Evidence

- [results] RECOGNICER-identified H3K27me3 domains cover more repressed genes in their entire gene bodies than the other 3 methods for 8 of 9 ENCODE cell types: [evidence span withheld — anchor status: not_found]
