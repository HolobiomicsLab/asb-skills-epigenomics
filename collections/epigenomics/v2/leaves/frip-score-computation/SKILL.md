---
name: frip-score-computation
description: Use when you need a quality/robustness metric for broad-domain ChIP-seq
  calls — e.g. verifying that total aggregate domain length and FRIP remain invariant
  under sequencing-depth downsampling (17.23M → 4M reads) of H3K27me3 data.
when_to_use_negative:
- No called peak/domain set exists yet — FRIP is computed relative to RECOGNICER's
  fdr0.01 broadPeak output, so the domain-calling pipeline must run first
- The ChIP target is narrow/punctate enrichment rather than broad kb-to-Mb domains
  like H3K27me3/H3K9me3, where this skill's invariance rationale is grounded
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3308
tools:
- name: RECOGNICER
  role: Calls broad H3K27me3 domains from ChIP and control BED inputs at FDR 0.01;
    its broadPeak output defines the intervals over which FRIP is computed, and it
    is rerun per downsampled depth condition
  repo: github.com/zanglab/recognicer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:64b91cdac5bacbbe018da9d8b33d853748747c06fc5ebe6271a3f1b0e1a06388
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - RECOGNICER
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# frip-score-computation

## Summary

Computes the FRIP (fraction of reads in peaks) score for RECOGNICER broad-domain ChIP-seq calls, using the fraction of ChIP reads falling inside significant H3K27me3 domains relative to total mapped reads. In the RECOGNICER paper it serves as a robustness metric: FRIP stays at the same level when the CD4+ T cell H3K27me3 dataset is downsampled from 17.23M to 4M reads.

## When to use

Use when you need a quality/robustness metric for broad-domain ChIP-seq calls — e.g. verifying that total aggregate domain length and FRIP remain invariant under sequencing-depth downsampling (17.23M → 4M reads) of H3K27me3 data.

## When NOT to use

- No called peak/domain set exists yet — FRIP is computed relative to RECOGNICER's fdr0.01 broadPeak output, so the domain-calling pipeline must run first
- The ChIP target is narrow/punctate enrichment rather than broad kb-to-Mb domains like H3K27me3/H3K9me3, where this skill's invariance rationale is grounded

## Inputs

- Mapped ChIP-seq reads in BED format (human CD4+ T cell H3K27me3, 17.23M full-depth and 4M downsampled)
- Control sample mapped reads in BED format
- RECOGNICER significant domains in ENCODE broadPeak format ($SAMPLE-fdr0.01_broadPeak.bed, FDR threshold 0.01)

## Outputs

- FRIP score per sequencing-depth condition (17.23M and 4M reads)
- Depth invariance summary (FRIP and total aggregate domain length per depth, e.g. depth_invariance_summary.tsv)

## How to apply

Run RECOGNICER on the ChIP and control BED files (e.g. via `sh RECOGNICER.sh ChIP.bed control.bed 0.01`, with default WINDOW_SIZE=200 bp and FRAGMENT_SIZE=150 bp) to obtain significant domains in `$SAMPLE-fdr0.01_broadPeak.bed` at FDR 0.01. Then intersect the mapped ChIP reads with these broadPeak intervals and divide the number of reads inside domains by the total number of mapped ChIP reads to obtain FRIP. For the depth-robustness check, randomly downsample the mapped-read dataset (17.23 million → 4 million reads), rerun the full coarse-graining pipeline (recursive block transformation with termination when fitted auto-correlation length λ drops below 1, domain retrieval, and local Poisson significance testing with Benjamini-Hochberg FDR), and recompute FRIP per run. Correctness is judged by FRIP staying at the same level across depths, alongside total aggregate domain length remaining largely unchanged.

## Related tools

- **RECOGNICER** (Calls broad H3K27me3 domains from ChIP and control BED inputs at FDR 0.01; its broadPeak output defines the intervals over which FRIP is computed, and it is rerun per downsampled depth condition) — github.com/zanglab/recognicer

## Examples

```
sh RECOGNICER.sh ChIP.bed control.bed 0.01; bedtools intersect -a ChIP.bed -b Sample-fdr0.01_broadPeak.bed -u | wc -l
```

## Evaluation signals

- FRIP score stays at the same level between the 17.23M-read and 4M-read runs (no systematic degradation)
- Total aggregate length of identified H3K27me3 domains remains largely unchanged across downsampling conditions
- broadPeak output schema holds: score (5th column) is fold change of ChIP vs control read counts, signalValue (7th column) is ChIP read count
- Domains were called with the default RECOGNICER parameters (window size 200 bp, fragment size 150 bp, FDR threshold 0.01) before FRIP is computed
- Coarse-graining terminated automatically when the fitted auto-correlation characteristic length λ dropped below 1 on the next coarse scale

## Limitations

- RECOGNICER could falsely merge two domains separated by fine-scale gaps into one domain on the coarse scale; individual cases should be monitored and may require masked/featured regions
- RECOGNICER v1.0 is implemented in Python 2 and its script instances must not run in parallel from the same directory because temporary files have hard-coded names
- GENOME_FRACTION default (0.74 for the human genome) depends on read length, which affects background expectations underlying significance calls feeding FRIP

## Evidence
