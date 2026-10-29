---
name: consensus-peak-computation-across-peaksets
description: Use when multiple peak files from related samples or technologies (e.g.
  CnT and CnR H3K27ac peak sets) need to be collapsed into a single consensus GRanges/BED
  peak set for QC or benchmarking.
when_to_use_negative:
- The goal is per-sample QC or pairwise overlap statistics rather than a merged consensus
  set — EpiCompare's main report (peak overlap, blacklist filtering, functional annotation)
  covers that instead
- Only one peak file is available, since consensus computation requires a group of
  peak files to intersect/aggregate
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: EpiCompare
  role: Provides compute_consensus_peaks with the 'granges' (fast overlap) and 'consensusseeker'
    (distribution modelling) methods, plus built-in CnT_H3K27ac and CnR_H3K27ac example
    peak files
  repo: https://github.com/neurogenomics/EpiCompare
- name: Bioconductor
  role: Distribution platform through which EpiCompare (≥ v3.15) is installed
- name: R
  role: Language in which the consensus peak calling is executed (GRanges/BED handling)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:4b726c1c6f417c594c1df36b4f390d93780a59e69cbf83b13a86e33a65c728ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neurogenomics/EpiCompare
  tools:
  - Bioconductor
  - EpiCompare
  - R
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# consensus-peak-computation-across-peaksets

## Summary

Calls a consensus peak set from a group of epigenomic peak files (e.g. CUT&Tag and CUT&Run H3K27ac BED/GRanges inputs) using EpiCompare's compute_consensus_peaks, choosing between a fast interval-overlap strategy (method='granges') and a slower distribution-modelling strategy (method='consensusseeker').

## When to use

Use when multiple peak files from related samples or technologies (e.g. CnT and CnR H3K27ac peak sets) need to be collapsed into a single consensus GRanges/BED peak set for QC or benchmarking.

## When NOT to use

- The goal is per-sample QC or pairwise overlap statistics rather than a merged consensus set — EpiCompare's main report (peak overlap, blacklist filtering, functional annotation) covers that instead
- Only one peak file is available, since consensus computation requires a group of peak files to intersect/aggregate

## Inputs

- Named list/group of peak files as GRanges or BED file paths (e.g. EpiCompare built-in CnT_H3K27ac and CnR_H3K27ac)
- Peak files imported via EpiCompare::gather_files (e.g. type='peaks.stringent' BED files)
- Choice of consensus method: 'granges' or 'consensusseeker'

## Outputs

- Consensus peak set as a GRanges object (one per method)
- Consensus peak sets exported as BED files (granges-method and consensusseeker-method outputs)

## How to apply

Assemble the peak files into a single named list/group (e.g. EpiCompare's built-in CnT_H3K27ac and CnR_H3K27ac H3K27ac peak files, or BED files imported via gather_files). Run compute_consensus_peaks on the group twice: once with method='granges' for a fast but simplistic interval-overlap consensus, and once with method='consensusseeker' for a slower but more accurate consensus that incorporates modelling of peak distributions (peak scores). Export each resulting consensus GRanges object to a BED file labelled by method (granges-method and consensusseeker-method outputs). Note that the paper benchmarks the two methods only descriptively, so acceptance is that both consensus outputs exist, parse as valid GRanges/BED, and contain a non-empty set of peaks covering the CnT and CnR H3K27ac intervals.

## Related tools

- **EpiCompare** (Provides compute_consensus_peaks with the 'granges' (fast overlap) and 'consensusseeker' (distribution modelling) methods, plus built-in CnT_H3K27ac and CnR_H3K27ac example peak files) — https://github.com/neurogenomics/EpiCompare
- **Bioconductor** (Distribution platform through which EpiCompare (≥ v3.15) is installed)
- **R** (Language in which the consensus peak calling is executed (GRanges/BED handling))

## Examples

```
consensus <- EpiCompare::compute_consensus_peaks(peakfiles = list("CnT"=CnT_H3K27ac, "CnR"=CnR_H3K27ac), method = "consensusseeker")
```

## Evaluation signals

- Both consensus outputs (granges-method and consensusseeker-method) are produced and exist as files
- Each consensus output parses as a valid GRanges object and exports to a well-formed BED file
- The consensus peak set is non-empty and covers intervals from both the CnT and CnR H3K27ac inputs
- Peaks falling in ENCODE blacklist regions (hg19/hg38) and non-standard chromosomes are excluded prior to analysis, per EpiCompare's standard pre-filtering

## Limitations

- The two methods are benchmarked only descriptively in the paper — no numeric acceptance criterion is reported, so validity is judged by output existence and non-emptiness
- method='granges' is fast but simplistic (pure interval overlap); method='consensusseeker' is more accurate via distribution modelling of peak scores but slower
- EpiCompare removes blacklisted-region and non-standard-chromosome peaks before analysis, which can shrink the consensus set

## Evidence

- [readme] granges vs consensusseeker consensus strategies: [evidence span withheld — anchor status: external_doc]
- [abstract] comparison and QC scope of EpiCompare: "EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets."
- [readme] importing peak files for the consensus group: [evidence span withheld — anchor status: external_doc]
- [readme] blacklist pre-filtering applied before analysis: [evidence span withheld — anchor status: external_doc]
