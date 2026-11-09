---
name: genomic-blacklist-region-filtering
description: Use when preparing narrowPeak/BED peak files (ChIP-seq, CUT&Tag, CUT&Run,
  ATAC-seq, DNase-seq) for cross-dataset comparison or QC, so that artifactual peaks
  in ENCODE blacklisted regions (hg19/hg38) do not bias overlap statistics, correlation
  plots, or ChromHMM/ChIPseeker annotations.
when_to_use_negative:
- Your peaks are on a non-human genome (mouse, fly, etc.) — EpiCompare's bundled blacklists
  are human hg19/hg38 only
- You are performing raw signal/fragment-level QC (e.g. Picard duplication-rate inspection)
  rather than peak-level comparisons, where blacklist filtering is not the primary
  step
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0769
tools:
- name: EpiCompare
  role: Performs the blacklist and non-standard-chromosome filtering internally when
    its EpiCompare() function is called with the 'blacklist' argument; also reports
    the fraction of blacklisted peaks per sample
  repo: https://github.com/neurogenomics/EpiCompare
- name: Bioconductor
  role: Distribution platform for EpiCompare (>= v3.15) including the bundled hg19/hg38
    ENCODE blacklist data objects
- name: R
  role: Runtime environment in which GRanges peak objects are filtered against blacklist
    regions before analysis
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:d0fcf88cc5274630324533e90b24f30e09d8a4acec93c2381cd51ca23a74a549
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neurogenomics/EpiCompare
  tools:
  - BiocStyle
  - Bioconductor
  - ChIPseeker
  - EpiCompare
  - R
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# genomic-blacklist-region-filtering

## Summary

A QC preprocessing step in EpiCompare that removes peak regions overlapping ENCODE blacklisted genomic regions and peaks on non-standard chromosomes (e.g. chrUn, alt contigs) from each peak file before any overlap, correlation, or functional-annotation analysis.

## When to use

Use when preparing narrowPeak/BED peak files (ChIP-seq, CUT&Tag, CUT&Run, ATAC-seq, DNase-seq) for cross-dataset comparison or QC, so that artifactual peaks in ENCODE blacklisted regions (hg19/hg38) do not bias overlap statistics, correlation plots, or ChromHMM/ChIPseeker annotations.

## When NOT to use

- Your peaks are on a non-human genome (mouse, fly, etc.) — EpiCompare's bundled blacklists are human hg19/hg38 only
- You are performing raw signal/fragment-level QC (e.g. Picard duplication-rate inspection) rather than peak-level comparisons, where blacklist filtering is not the primary step

## Inputs

- Peak files (GRanges objects or paths to narrowPeak/BED files) from ATAC-seq, DNase-seq, CUT&Tag, CUT&Run, or ChIP-seq experiments
- ENCODE blacklist region file for the matching genome build (hg19 or hg38), e.g. hg19_blacklist
- Genome build annotations per peak file (via the genome_build list argument)

## Outputs

- Filtered GRanges peak files with blacklisted-region overlaps and non-standard chromosomes removed
- Percentage of blacklisted/non-standard peaks per sample, reported in the General Metrics section of the HTML report

## How to apply

Load the ENCODE blacklist matching the genome build of your peaks — EpiCompare ships human hg19 and hg38 blacklisted regions obtained from ENCODE (e.g. data('hg19_blacklist') for hg19-built peak files like the CnT_H3K27ac/CnR_H3K27ac examples). Pass the blacklist to EpiCompare() via the 'blacklist' argument; the package automatically removes blacklisted peaks and non-standard-chromosome peaks from all query and reference peakfiles prior to computing metrics. Match genome builds explicitly using the genome_build argument (e.g. list(peakfiles='hg19', reference='hg38')) with genome_build_output specifying the target build so liftover and filtering are consistent. The step is a prerequisite for all three report sections (General Metrics, Peak Overlap, Functional Annotation); in the K562 ENCODE workflow, peaks are imported as GRanges and blacklisted regions are removed before calling EpiCompare() with genome_build='hg38'.

## Related tools

- **EpiCompare** (Performs the blacklist and non-standard-chromosome filtering internally when its EpiCompare() function is called with the 'blacklist' argument; also reports the fraction of blacklisted peaks per sample) — https://github.com/neurogenomics/EpiCompare
- **Bioconductor** (Distribution platform for EpiCompare (>= v3.15) including the bundled hg19/hg38 ENCODE blacklist data objects)
- **R** (Runtime environment in which GRanges peak objects are filtered against blacklist regions before analysis)

## Examples

```
EpiCompare::EpiCompare(peakfiles = peakfiles, genome_build = list(peakfiles="hg19", reference="hg38"), genome_build_output = "hg19", blacklist = hg19_blacklist, reference = reference, output_dir = tempdir())
```

## Evaluation signals

- Reported 'percentage of blacklisted and non-standard peaks' per sample is present in the General Metrics section of the generated HTML report
- No retained peak coordinates fall within the ENCODE blacklist intervals for the declared genome build (check via overlapsAny against the blacklist GRanges)
- All retained peaks are on standard chromosomes (no chrUn, chr*alt, or contig names outside the canonical chromosome set)
- Downstream Peak Overlap and Functional Annotation statistics are computed only on post-filter peaks, so re-running with/without the blacklist argument should measurably change overlap percentages in blacklisted regions
- Genome build consistency: filtering used the blacklist matching each file's declared build (e.g. hg19_blacklist for hg19 peakfiles, hg38 blacklist for hg38 ATAC/DNase peaks)

## Limitations

- Filtering is automatic and applied to all peakfiles, so users who want to retain blacklisted-region peaks (e.g. for artifact auditing) must account for this behavior
- Only human hg19 and hg38 ENCODE blacklists ship with the package; other genome builds require supplying an externally sourced blacklist
- Input peak and blacklist genome builds must be declared correctly via genome_build; mismatched builds will produce wrong filtering and require liftover via genome_build_output

## Evidence

- [intro] ENCODE blacklist coverage in package data: "It also contains human genome hg19 and hg38 blacklisted regions obtained from ENCODE."
- [readme] Filtering step description in README: [evidence span withheld — anchor status: external_doc]
- [readme] Metrics reported after filtering: "Metrics on peaks (percentage of blacklisted and non-standard peaks, and peak widths) and fragments (duplication rate) of samples."
- [full_text] hg38 workflow with blacklist argument: [evidence span withheld — anchor status: not_found]
- [readme] Single-call invocation showing blacklist/genome_build parameters: "genome_build = list(peakfiles="hg19",
                                           reference="hg38"),
                       genome_build_output = "hg19""
