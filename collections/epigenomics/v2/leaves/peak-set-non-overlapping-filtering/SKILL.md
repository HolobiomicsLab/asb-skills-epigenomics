---
name: peak-set-non-overlapping-filtering
description: Use when a RangedSummarizedExperiment of ATAC-seq/DNase-seq fragment
  counts has already been sample-filtered and the peak set still contains peaks with
  no fragments across samples or overlapping peak intervals that would bias motif
  accessibility estimates.
when_to_use_negative:
- Input is raw, unfiltered counts with low-quality samples not yet removed via filterSamples
  — peak filtering assumes the sample set is already curated
- The peak set is already non-overlapping and every peak has sufficient fragment support
  (filterPeaks would be a no-op)
- Downstream analysis uses an external fixed peak set (e.g. a reference cCRE catalog)
  that must not be collapsed or subset
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0769
tools:
- name: chromVAR
  role: Provides filterPeaks, which reduces the peak set to non-overlapping peaks
    with at least a minimum number of fragments
  repo: https://github.com/GreenleafLab/chromVAR
- name: SummarizedExperiment
  role: Container class for the fragment-count matrix and peak rowData that filterPeaks
    takes as input and returns
- name: R
  role: Language in which the chromVAR filtering workflow is run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nmeth.4401
    title: 'chromVAR: inferring transcription-factor-associated accessibility from
      single-cell epigenomic data'
  source_content_hash: sha256:17e62950721f8b36c019148880804b7d91db90fcd4d897d9b69f35ae96a51c98
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GreenleafLab/chromVAR
  tools:
  - BiocParallel
  - R
  - SummarizedExperiment
  - chromVAR
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# Peak-set non-overlapping filtering

## Summary

Reduces a chromVAR fragment-count peak set to high-quality, non-overlapping peaks after sample filtering, so that downstream motif deviation computations are not biased by redundant or poorly-supported peaks.

## When to use

Use when a RangedSummarizedExperiment of ATAC-seq/DNase-seq fragment counts has already been sample-filtered and the peak set still contains peaks with no fragments across samples or overlapping peak intervals that would bias motif accessibility estimates.

## When NOT to use

- Input is raw, unfiltered counts with low-quality samples not yet removed via filterSamples — peak filtering assumes the sample set is already curated
- The peak set is already non-overlapping and every peak has sufficient fragment support (filterPeaks would be a no-op)
- Downstream analysis uses an external fixed peak set (e.g. a reference cCRE catalog) that must not be collapsed or subset

## Inputs

- RangedSummarizedExperiment of fragment counts (output of chromVAR getCounts and filterSamples)
- Sample colData including 'depth' column and 'in.peaks' fractions

## Outputs

- Filtered RangedSummarizedExperiment with non-overlapping peaks, each supported by at least one fragment across retained samples (e.g. counts_filtered.rds)

## How to apply

After filtering samples with filterSamples (e.g. min_depth = 1500, min_in_peaks = 0.15), call chromVAR's filterPeaks on the RangedSummarizedExperiment with non_overlapping = TRUE. This removes peaks that fail to reach the required fragment count across samples (at least one fragment per peak across samples) and collapses overlapping peaks to the single peak with higher counts. Rationale: chromVAR's computeDeviations estimates gain or loss of accessibility within peaks sharing a motif while controlling for technical biases, so redundant or empty peaks distort both the deviation estimates and background-peak selection (getBackgroundPeaks matches on GC content and average accessibility). Save the result (e.g. as counts_filtered.rds) for downstream motif matching with matchMotifs.

## Related tools

- **chromVAR** (Provides filterPeaks, which reduces the peak set to non-overlapping peaks with at least a minimum number of fragments) — https://github.com/GreenleafLab/chromVAR
- **SummarizedExperiment** (Container class for the fragment-count matrix and peak rowData that filterPeaks takes as input and returns)
- **R** (Language in which the chromVAR filtering workflow is run)

## Examples

```
counts_filtered <- filterPeaks(counts_filtered, non_overlapping = TRUE); saveRDS(counts_filtered, "counts_filtered.rds")
```

## Evaluation signals

- Output loads as a RangedSummarizedExperiment retaining the 'counts' assay
- No two retained peaks overlap; overlapping intervals have been collapsed to the peak with higher counts
- Every retained peak has at least one fragment across samples
- Retained samples still satisfy the prior filterSamples cutoffs (min_depth = 1500, min_in_peaks = 0.15)
- colData 'depth' column remains consistent with the retained sample set

## Limitations

- filterPeaks only reduces to non-overlapping peaks; it does not resize or merge peaks — use readNarrowpeaks beforehand if peaks need resizing to a given size
- Filtering on fragment counts interacts with sample filtering: peaks are judged on support across retained samples, so overly aggressive sample filtering can drop peaks needed for motif analysis
- For clustering tasks, chromVAR variants (kmers + PCA) are outperformed by newer methods such as SnapATAC, so peak filtering within chromVAR may not be the optimal preprocessing for clustering

## Evidence
