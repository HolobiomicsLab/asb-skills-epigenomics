---
name: multi-method-benchmark-comparison
description: Use when a new domain/peak-calling or prediction method needs head-to-head
  validation against established alternatives on shared real datasets, with a functional
  readout rather than only internal overlap metrics.
when_to_use_negative:
- The competing methods lack published default parameters or cannot run on the same
  input format (e.g. some require control BED files unavailable for a dataset), making
  a default-parameter comparison unfair.
- No functional ground truth (such as matched gene expression to define repressed
  genes) exists for the datasets, so only internal metrics like domain overlap would
  be possible instead of the functional coverage benchmark.
- The question is about sharp/point-source peaks rather than broad domains; RECOGNICER,
  SICER, RSEG, and MUSIC are broad-domain or multiscale callers, and the whole-gene-body
  coverage metric assumes kb-to-Mb domains.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: RECOGNICER
  role: Candidate method under evaluation; run with default parameters (fragment size
    150 bp, window size 200 bp, FDR 0.01) to produce H3K27me3 domains in ENCODE broadPeak
    format for benchmarking against competitors
  repo: github.com/zanglab/recognicer
- name: SICER
  role: Established broad-domain caller used as one of the three baseline competitors
    on the same ENCODE datasets with default parameters
- name: RSEG
  role: HMM-based broad epigenomic domain caller used as a baseline competitor with
    default parameters
- name: MUSIC
  role: Multiscale decomposition peak caller used as a baseline competitor with default
    parameters
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:ea4b9b7b99fca5f56819a155ce755c740034db62162a29fc25737522fcf883f5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MUSIC
  - RECOGNICER
  - RSEG
  - SICER
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# multi-method-benchmark-comparison

## Summary

A benchmarking skill for comparing a candidate method (here RECOGNICER) against competing tools (SICER, RSEG, MUSIC) on the same public datasets, using a biologically grounded functional metric — in this case, the fraction of repressed genes whose entire gene bodies are covered by a single called H3K27me3 domain across 9 ENCODE cell types.

## When to use

Use when a new domain/peak-calling or prediction method needs head-to-head validation against established alternatives on shared real datasets, with a functional readout rather than only internal overlap metrics.

## When NOT to use

- The competing methods lack published default parameters or cannot run on the same input format (e.g. some require control BED files unavailable for a dataset), making a default-parameter comparison unfair.
- No functional ground truth (such as matched gene expression to define repressed genes) exists for the datasets, so only internal metrics like domain overlap would be possible instead of the functional coverage benchmark.
- The question is about sharp/point-source peaks rather than broad domains; RECOGNICER, SICER, RSEG, and MUSIC are broad-domain or multiscale callers, and the whole-gene-body coverage metric assumes kb-to-Mb domains.

## Inputs

- ENCODE H3K27me3 ChIP-seq mapped-read BED files (treatment and matched control) for 9 cell types
- Matched gene expression profiles (microarray or RNA-seq) for the same 9 cell types
- Method outputs: RECOGNICER ENCODE broadPeak-format domains plus SICER, RSEG, and MUSIC called domains on the same datasets

## Outputs

- Per-cell-type, per-method fractions of repressed genes with entire gene bodies covered by a single called H3K27me3 domain
- Combined benchmark table: h3k27me3_domain_coverage_comparison.tsv
- Ranking of methods per cell type (which method achieves the highest coverage fraction)

## How to apply

Obtain matched inputs for all tested conditions — here ENCODE H3K27me3 ChIP-seq mapped-read BED files (treatment and matched control) plus matched gene expression profiles (microarray or RNA-seq) for the 9 cell types. Run the candidate method (RECOGNICER, default parameters: DNA fragment size 150 bp, window size 200 bp, FDR 0.01) and each competitor (SICER, RSEG, MUSIC with default parameters) on identical datasets to eliminate configuration confounds. Define a biologically interpretable metric: classify genes as repressed or not from the matched expression data, intersect each method's called domains with gene bodies, and compute the fraction of repressed genes whose entire gene body falls within a single domain. Assemble per-condition, per-method fractions into one comparison table (h3k27me3_domain_coverage_comparison.tsv) and validate against the expected benchmark outcome (RECOGNICER highest in 8 of 9 cell types). Running all tools with default parameters, as the paper does, keeps the comparison reproducible and fair.

## Related tools

- **RECOGNICER** (Candidate method under evaluation; run with default parameters (fragment size 150 bp, window size 200 bp, FDR 0.01) to produce H3K27me3 domains in ENCODE broadPeak format for benchmarking against competitors) — github.com/zanglab/recognicer
- **SICER** (Established broad-domain caller used as one of the three baseline competitors on the same ENCODE datasets with default parameters)
- **RSEG** (HMM-based broad epigenomic domain caller used as a baseline competitor with default parameters)
- **MUSIC** (Multiscale decomposition peak caller used as a baseline competitor with default parameters)

## Examples

```
sh RECOGNICER.sh GSM_sample_ChIP.bed GSM_sample_control.bed 0.01
```

## Evaluation signals

- The comparison table h3k27me3_domain_coverage_comparison.tsv contains a coverage fraction for every (cell type, method) pair: 9 cell types × 4 methods = 36 rows.
- RECOGNICER achieves the highest repressed-gene whole-gene-body coverage fraction among the four methods in 8 of the 9 ENCODE cell types.
- All methods were run on the same input BED files (treatment and matched control) with default parameters — no per-method parameter tuning is present.
- Domain outputs are in ENCODE broadPeak format (chrom, start, end, name, score as fold change, strand, signalValue as ChIP read count, pValue, qValue).
- Repressed-gene annotations derive from the matched expression data for each cell type, not from a single pan-dataset gene list.

## Limitations

- Default-parameter comparison may not reflect each tool's best achievable performance; the paper explicitly compares 'using default parameters'.
- RECOGNICER can falsely merge two domains separated by fine-scale gaps into one coarse-scale domain, which could inflate whole-gene-body coverage; users should monitor individual merging cases and may incorporate featured or masked regions.
- The benchmark depends on the definition of 'repressed' from microarray or RNA-seq expression data, which varies by cell type and thresholding choice.
- RECOGNICER v1.0 is implemented in python2 and only supports a fixed set of genomes (hg38/hg19/hg18, mm10/mm9/mm8, rn4, dm2/dm3, sacCer1, pombe, tair8); custom genomes require editing GenomeData.py.

## Evidence

- [results] Benchmark outcome: RECOGNICER wins in 8/9 cell types: [evidence span withheld — anchor status: not_found]
