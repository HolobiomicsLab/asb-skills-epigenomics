---
name: peak-score-distribution-modelling
description: Use when you have multiple peak files (e.g. CUT&Tag/CUT&Run H3K27ac BED
  files as GRanges) that should be collapsed into a single consensus peak set, and
  you must choose between fast overlap-based and distribution-model-based consensus
  calling.
when_to_use_negative:
- The peak files are already a single merged consensus set and only QC/comparison
  against a reference (e.g. ENCODE_H3K27ac) is needed rather than consensus calling
- Only a fast descriptive overlap summary is needed and the runtime cost of distribution
  modelling (consensusseeker) cannot be tolerated
- Input is raw alignment data (e.g. BAM fragments) rather than called peak files;
  EpiCompare operates on peak files such as BED imports
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: EpiCompare
  role: Provides compute_consensus_peaks with the 'granges' (fast interval-overlap)
    and 'consensusseeker' (peak score distribution modelling) consensus-calling strategies,
    plus built-in CnT_H3K27ac/CnR_H3K27ac example peak files
  repo: https://github.com/neurogenomics/EpiCompare
- name: R
  role: Language in which the consensus peak calling workflow is executed
- name: Bioconductor
  role: Distribution channel for EpiCompare (≥ v3.15) and the GRanges/Bioconductor
    infrastructure used for peak objects
  repo: https://bioconductor.org/packages/release/bioc/html/EpiCompare.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:a4510fd3f5172406d31e30a2c2d758048c8c824b3d629f5ad39d8fe67a4d213d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neurogenomics/EpiCompare
  tools:
  - Bioconductor
  - EpiCompare
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# peak-score-distribution-modelling

## Summary

Calling consensus peaks from a group of epigenomic peak files using EpiCompare's compute_consensus_peaks, which offers a fast interval-overlap strategy (method='granges') and a slower, more accurate strategy that models peak score distributions (method='consensusseeker').

## When to use

Use when you have multiple peak files (e.g. CUT&Tag/CUT&Run H3K27ac BED files as GRanges) that should be collapsed into a single consensus peak set, and you must choose between fast overlap-based and distribution-model-based consensus calling.

## When NOT to use

- The peak files are already a single merged consensus set and only QC/comparison against a reference (e.g. ENCODE_H3K27ac) is needed rather than consensus calling
- Only a fast descriptive overlap summary is needed and the runtime cost of distribution modelling (consensusseeker) cannot be tolerated
- Input is raw alignment data (e.g. BAM fragments) rather than called peak files; EpiCompare operates on peak files such as BED imports

## Inputs

- Named list of peak files as GRanges objects (e.g. EpiCompare built-in CnT_H3K27ac and CnR_H3K27ac H3K27ac peak files)
- Group assignment assigning the peak files to a single consensus group
- Method choice: 'granges' (fast overlap) or 'consensusseeker' (peak score distribution modelling)

## Outputs

- Consensus peak set as a GRanges object (one per method)
- Exported consensus peak files in BED format (granges-method and consensusseeker-method outputs)

## How to apply

Load the peak files of interest as GRanges (e.g. the built-in CnT_H3K27ac and CnR_H3K27ac H3K27ac files shipped with EpiCompare) and place them into a single consensus group. Run EpiCompare::compute_consensus_peaks twice on that group: once with method='granges' for a fast but simplistic interval-overlap consensus, and once with method='consensusseeker' for a slower but more accurate consensus that incorporates modelling of the peak score distributions. Export each resulting consensus GRanges to BED (e.g. granges-method and consensusseeker-method outputs) for downstream comparison. Per EpiCompare's preprocessing contract, peaks in ENCODE blacklisted regions (hg19/hg38) and on non-standard chromosomes are removed prior to analysis. Because the paper benchmarks the two strategies only descriptively, validation is that both consensus outputs exist, parse as valid GRanges/BED, and contain a non-empty peak set covering the CnT and CnR H3K27ac intervals.

## Related tools

- **EpiCompare** (Provides compute_consensus_peaks with the 'granges' (fast interval-overlap) and 'consensusseeker' (peak score distribution modelling) consensus-calling strategies, plus built-in CnT_H3K27ac/CnR_H3K27ac example peak files) — https://github.com/neurogenomics/EpiCompare
- **R** (Language in which the consensus peak calling workflow is executed)
- **Bioconductor** (Distribution channel for EpiCompare (≥ v3.15) and the GRanges/Bioconductor infrastructure used for peak objects) — https://bioconductor.org/packages/release/bioc/html/EpiCompare.html

## Examples

```
library(EpiCompare); data("CnT_H3K27ac"); data("CnR_H3K27ac"); cons_cs <- compute_consensus_peaks(peakfiles=list(CnT=CnT_H3K27ac,CnR=CnR_H3K27ac), groups=c("consensus","consensus"), method="consensusseeker")
```

## Evaluation signals

- Both consensus outputs exist and parse as valid GRanges/BED files
- Each consensus output contains a non-empty set of peaks covering the CnT and CnR H3K27ac intervals
- Consensus peaks do not fall in ENCODE blacklist (hg19/hg38) regions or on non-standard chromosomes, consistent with EpiCompare's preprocessing
- The two methods produce two distinct consensus sets (granges vs consensusseeker) that can be exported and compared descriptively, as the paper reports no numeric benchmark criterion

## Limitations

- The granges method is fast but simplistic; the consensusseeker method is slower but more accurate because it models peak score distributions
- The paper benchmarks the two consensus methods only descriptively — no numeric performance criterion is reported
- Peaks in blacklisted regions and non-standard chromosomes are removed before analysis, so consensus calling reflects the filtered peak sets

## Evidence

- [other] Two consensus methods in compute_consensus_peaks: [evidence span withheld — anchor status: not_found]
- [abstract] EpiCompare scope: "EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets."
- [readme] Blacklist/non-standard chromosome filtering: [evidence span withheld — anchor status: external_doc]
- [readme] Built-in example peak files: "data("CnT_H3K27ac") # example peakfile
data("CnR_H3K27ac") # example peakfile"
- [abstract] Availability on Bioconductor: "EpiCompare is available on Bioconductor (≥ v3.15): https://bioconductor.org/packages/release/bioc/html/EpiCompare.html"
