---
name: granges-interval-overlap-operations
description: Use when you have a named group of replicate peak files (e.g. GRanges/BED
  H3K27ac datasets such as CUT&Tag CnT_H3K27ac and CUT&Run CnR_H3K27ac) and need a
  single consensus GRanges peak set.
when_to_use_negative:
- You only need pairwise overlap statistics or precision-recall/plots between peak
  files rather than a merged consensus set — use EpiCompare's Peak Overlap report
  features instead
- The peaks have not been filtered for blacklisted regions and non-standard chromosomes;
  EpiCompare removes these prior to analysis and results assume that cleaning step
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: EpiCompare
  role: Provides compute_consensus_peaks with method='granges' (fast interval-overlap)
    and method='consensusseeker' (distribution-modelled) consensus strategies
  repo: https://github.com/neurogenomics/EpiCompare
- name: R
  role: Language in which the consensus peak calling functions are run
- name: Bioconductor
  role: Distribution channel for EpiCompare (>= v3.15) and underlying GRanges/BED
    infrastructure
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:3e56e2ee4ef04fc6f6d50f908e194593be1b2e76d1e32b88f76a67e94adac53f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neurogenomics/EpiCompare
  tools:
  - Bioconductor
  - EpiCompare
  - R
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# granges-interval-overlap-operations

## Summary

Derives consensus peak sets from groups of replicate peak files using GenomicRanges-style interval overlap operations, as implemented in EpiCompare's compute_consensus_peaks with two interchangeable strategies (fast interval-overlap vs. distribution modelling).

## When to use

Use when you have a named group of replicate peak files (e.g. GRanges/BED H3K27ac datasets such as CUT&Tag CnT_H3K27ac and CUT&Run CnR_H3K27ac) and need a single consensus GRanges peak set.

## When NOT to use

- You only need pairwise overlap statistics or precision-recall/plots between peak files rather than a merged consensus set — use EpiCompare's Peak Overlap report features instead
- The peaks have not been filtered for blacklisted regions and non-standard chromosomes; EpiCompare removes these prior to analysis and results assume that cleaning step

## Inputs

- Named list/group of replicate peak files as GRanges objects or BED file paths (e.g. EpiCompare built-ins CnT_H3K27ac, CnR_H3K27ac)
- Method choice string: 'granges' or 'consensusseeker'

## Outputs

- Consensus peak set as a GRanges object
- Consensus peak set exported as a BED file (one per method used)

## How to apply

Load the replicate peak files into a single named group (e.g. CnT_H3K27ac and CnR_H3K27ac H3K27ac peak files from EpiCompare). Call compute_consensus_peaks on the group with method='granges' to derive a fast but simplistic interval-overlap consensus GRanges object, or method='consensusseeker' for a slower but more accurate strategy that incorporates modelling of peak distributions. Decide between methods by trading speed against accuracy: 'granges' is the fast overlap strategy, 'consensusseeker' models the distributions of peak scores. Export the resulting consensus GRanges objects to BED files so they can be validated as parseable, non-empty peak sets covering the input replicate intervals. Note that EpiCompare removes peaks in blacklisted regions (ENCODE hg19/hg38 blacklists) and non-standard chromosomes from files prior to analysis.

## Related tools

- **EpiCompare** (Provides compute_consensus_peaks with method='granges' (fast interval-overlap) and method='consensusseeker' (distribution-modelled) consensus strategies) — https://github.com/neurogenomics/EpiCompare
- **R** (Language in which the consensus peak calling functions are run)
- **Bioconductor** (Distribution channel for EpiCompare (>= v3.15) and underlying GRanges/BED infrastructure)

## Examples

```
peakfiles <- list("CnT"=CnT_H3K27ac, "CnR"=CnR_H3K27ac); consensus <- EpiCompare::compute_consensus_peaks(peakfiles = peakfiles, method = "granges")
```

## Evaluation signals

- Both consensus outputs exist and parse as valid GRanges/BED objects
- The consensus peak set is non-empty and covers the CnT and CnR H3K27ac input intervals
- Running the same group with method='granges' and method='consensusseeker' yields two comparable consensus sets (descriptive comparison; the paper reports no numeric criterion)
- Consensus peaks do not fall in ENCODE blacklist (hg19/hg38) regions or non-standard chromosomes

## Limitations

- The 'granges' method is fast but simplistic (interval overlap only); the more accurate 'consensusseeker' method is slower because it models peak distributions
- The two consensus methods are benchmarked only descriptively in the paper — no numeric acceptance criterion is reported, so validation is existence/non-emptiness of valid outputs
- Results depend on EpiCompare's preprocessing: peaks in blacklisted regions and on non-standard chromosomes are removed before analysis

## Evidence

- [abstract] EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets.: "EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets."
- [readme] Peaks located in blacklisted regions and non-standard chromosomes are removed from the files prior to analysis.: "Peaks located in blacklisted regions and non-standard
chromosomes are removed from the files prior to analysis."
- [intro] It also contains human genome hg19 and hg38 blacklisted regions obtained from ENCODE.: "It also contains human genome hg19 and hg38 blacklisted regions obtained from ENCODE."
