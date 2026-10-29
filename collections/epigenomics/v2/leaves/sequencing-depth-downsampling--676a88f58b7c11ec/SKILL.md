---
name: sequencing-depth-downsampling
description: Use when validating that a broad-domain caller (e.g., RECOGNICER on H3K27me3/H3K9me3
  ChIP-seq) is robust to sequencing depth by downsampling mapped reads and comparing
  aggregate domain length and FRIP across depths.
when_to_use_negative:
- Calling peaks on the downsampled data for discovery purposes — downsampling here
  is strictly a robustness/replication test, not a depth-normalization or peak-calling
  strategy
- The input is raw sequencing data (FASTQ) — the method operates on already-mapped
  reads (BED); remapping each downsampled subset is a different question not covered
  here
- Peak-caller robustness to narrow, punctate marks — the article frames RECOGNICER
  and this test around broad domains from kb to Mb such as H3K27me3 and H3K9me3
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3308
tools:
- name: RECOGNICER
  role: Broad-domain caller re-run at each sequencing depth (full and downsampled)
    with default WINDOW_SIZE 200 bp and FRAGMENT_SIZE 150 bp; its broadPeak output
    provides the domains used to compute aggregate length
  repo: github.com/zanglab/recognicer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:18c61ce70f5e90ccf400d042a7c8fb59e9d3c40ffc87c772bb1fab85812eaf5a
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
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# sequencing-depth-downsampling

## Summary

Robustness assessment in which a mapped ChIP-seq dataset is randomly downsampled (e.g., human CD4+ T cell H3K27me3 from 17.23M to 4M reads) and a broad-domain caller is re-run at each depth to test invariance of total aggregate domain length and FRIP score.

## When to use

Use when validating that a broad-domain caller (e.g., RECOGNICER on H3K27me3/H3K9me3 ChIP-seq) is robust to sequencing depth by downsampling mapped reads and comparing aggregate domain length and FRIP across depths.

## When NOT to use

- Calling peaks on the downsampled data for discovery purposes — downsampling here is strictly a robustness/replication test, not a depth-normalization or peak-calling strategy
- The input is raw sequencing data (FASTQ) — the method operates on already-mapped reads (BED); remapping each downsampled subset is a different question not covered here
- Peak-caller robustness to narrow, punctate marks — the article frames RECOGNICER and this test around broad domains from kb to Mb such as H3K27me3 and H3K9me3

## Inputs

- Mapped ChIP-seq read file (BED format) for the ChIP sample at full depth (e.g., 17.23M reads)
- Mapped control sample read file (BED format)
- Target downsampled depth (e.g., 4 million randomly sampled reads)

## Outputs

- Significant broad-domain calls per depth condition ($SAMPLE-fdr0.01_broadPeak.bed, ENCODE broadPeak format)
- Total aggregate domain length per depth (sum of lengths of significant domains)
- FRIP score per depth condition
- depth_invariance_summary.tsv (sequencing depth vs. aggregate domain length vs. FRIP)

## How to apply

Randomly sample the mapped-read BED file down to a target depth (the paper used 4M reads from 17.23M) while keeping the full-depth dataset as the reference condition. Run RECOGNICER on each depth condition with default parameters — WINDOW_SIZE 200 bp, FRAGMENT_SIZE 150 bp for 5'-end shifting — so coarse-graining, domain retrieval with boundary determination, and significance determination are performed identically per condition. Call significant domains at a fixed FDR threshold (e.g., 0.01 passed to RECOGNICER.sh) and derive two summary metrics per run: total aggregate length of significant domains (summed from the broadPeak output) and FRIP score. Write depth, aggregate length, and FRIP into a summary table such as depth_invariance_summary.tsv. The rationale is that a coarse-graining method that identifies domains across multiple length scales should not systematically lose long-range structure as read count decreases; invariance of both metrics validates depth robustness.

## Related tools

- **RECOGNICER** (Broad-domain caller re-run at each sequencing depth (full and downsampled) with default WINDOW_SIZE 200 bp and FRAGMENT_SIZE 150 bp; its broadPeak output provides the domains used to compute aggregate length) — github.com/zanglab/recognicer

## Examples

```
sh RECOGNICER.sh ChIP_4M.bed control.bed 0.01
```

## Evaluation signals

- Total aggregate length of significant H3K27me3 domains is approximately invariant between the 17.23M-read and 4M-read runs (no systematic degradation)
- FRIP score remains at the same level across depths despite the ~4.3x difference in read count
- Both depth runs use identical RECOGNICER parameters (WINDOW_SIZE 200, FRAGMENT_SIZE 150, same FDR threshold), so metric differences are attributable to depth alone
- Each run produces a well-formed $SAMPLE-fdr0.01_broadPeak.bed with chrom, start, end and the cgsummary Q-value/FDR statistics
- depth_invariance_summary.tsv contains one row per depth with non-decreasing or flat aggregate domain length and FRIP as depth increases

## Limitations

- Downsampling discards reads, so background significance at shallow depth relies on the local Poisson model with Benjamini-Hochberg FDR; very shallow depths below the tested 4M reads were not assessed
- RECOGNICER can falsely merge two domains separated by fine-scale gaps into one coarse-scale domain, which could bias aggregate-length comparisons between depths
- RECOGNICER scripts must not be run in parallel from the same directory (hard-coded temporary files), which constrains running multiple depth conditions concurrently from one install
- RECOGNICER v1.0 is implemented in python2 and requires scipy, an environment constraint when reproducing the downsampling analysis

## Evidence
