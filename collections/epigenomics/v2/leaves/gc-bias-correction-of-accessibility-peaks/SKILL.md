---
name: gc-bias-correction-of-accessibility-peaks
description: 'Use when computing chromVAR motif deviation scores from scATAC-seq peak
  counts: getBackgroundPeaks and computeDeviations require a ''bias'' rowData column
  derived from peak GC content to control for technical biases.'
when_to_use_negative:
- The input is already a bias-corrected SummarizedExperiment (rowData already contains
  a 'bias' column) — re-running would be redundant
- The downstream analysis does not use chromVAR's background-peak-based deviation
  framework, e.g. clustering with kmers + PCA or SnapATAC, which do not require the
  'bias' annotation
- Peak coordinates lack a matching BSgenome/fasta (genome build unknown), making GC
  computation unreliable
tools:
- name: chromVAR
  role: Provides the addGCBias function that computes per-peak GC bias and writes
    the 'bias' rowData column; its getBackgroundPeaks and computeDeviations functions
    consume that annotation
  repo: github.com/GreenleafLab/chromVAR
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Genome sequence source from which peak GC content (the 'bias' value) is computed
    for hg19 peak coordinates
- name: SummarizedExperiment
  role: Container class whose rowData carries the 'bias' column produced by addGCBias
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nmeth.4401
    title: 'chromVAR: inferring transcription-factor-associated accessibility from
      single-cell epigenomic data'
  source_content_hash: sha256:b23000e1413d938c1d9facfe5be22649f4ad235fe3b0f0afbc3a96891fb6b345
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - BiocParallel
  - R
  - SummarizedExperiment
  - chromVAR
  - motifmatchr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# GC bias correction of accessibility peaks

## Summary

Annotates ATAC-seq peaks with per-peak GC-content bias so that downstream chromVAR deviation scores control for sequence-dependent technical bias in sparse single-cell chromatin accessibility data.

## When to use

Use when computing chromVAR motif deviation scores from scATAC-seq peak counts: getBackgroundPeaks and computeDeviations require a 'bias' rowData column derived from peak GC content to control for technical biases.

## When NOT to use

- The input is already a bias-corrected SummarizedExperiment (rowData already contains a 'bias' column) — re-running would be redundant
- The downstream analysis does not use chromVAR's background-peak-based deviation framework, e.g. clustering with kmers + PCA or SnapATAC, which do not require the 'bias' annotation
- Peak coordinates lack a matching BSgenome/fasta (genome build unknown), making GC computation unreliable

## Inputs

- SummarizedExperiment / RangedSummarizedExperiment of ATAC-seq fragment counts per peak per cell (e.g. chromVAR example_counts)
- BSgenome object matching the peak genome build (e.g. BSgenome.Hsapiens.UCSC.hg19)
- Peak ranges in rowData/granges of the SummarizedExperiment

## Outputs

- SummarizedExperiment with a new rowData column named 'bias' holding per-peak GC-content bias values
- Bias-annotated counts object suitable for getBackgroundPeaks and computeDeviations

## How to apply

Load the peak-count SummarizedExperiment (e.g. the chromVAR example_counts dataset) and call addGCBias with a BSgenome object matching the peak coordinates, e.g. BSgenome.Hsapiens.UCSC.hg19. The function computes the GC content of each peak range and stores it as a new rowData column named 'bias'. This bias annotation is subsequently consumed by getBackgroundPeaks, which selects background peaks matched on GC content and average accessibility, and by computeDeviations, which uses the matched background to remove expected accessibility variation before computing motif deviation Z-scores. Apply the correction before filterSamples/filterPeaks ordering used in the vignette (addGCBias is called on example_counts immediately after loading, prior to filtering with min_depth = 1500 and min_in_peaks = 0.15). The genome build of the BSgenome must match the seqinfo of the peaks or the computed GC values will be wrong.

## Related tools

- **chromVAR** (Provides the addGCBias function that computes per-peak GC bias and writes the 'bias' rowData column; its getBackgroundPeaks and computeDeviations functions consume that annotation) — github.com/GreenleafLab/chromVAR
- **BSgenome.Hsapiens.UCSC.hg19** (Genome sequence source from which peak GC content (the 'bias' value) is computed for hg19 peak coordinates)
- **SummarizedExperiment** (Container class whose rowData carries the 'bias' column produced by addGCBias)

## Examples

```
example_counts <- addGCBias(example_counts, genome = BSgenome.Hsapiens.UCSC.hg19)
```

## Evaluation signals

- rowData of the returned SummarizedExperiment contains a column literally named 'bias'
- The 'bias' values are numeric GC-content-derived quantities bounded within [0, 1] per peak
- The genome passed to addGCBias matches the peak seqinfo (hg19 for the chromVAR example_counts dataset); a mismatch would raise errors or yield inconsistent GC values
- getBackgroundPeaks and computeDeviations run without complaining about a missing bias annotation
- Downstream motif variability output (computeVariability data.frame, written as motif_variability.csv) reproduces expected variability ranking when the pipeline is rerun with the same background-peak seed

## Limitations

- GC bias correction is only valid when peak coordinates and the supplied genome build agree; no genome sequence, no bias values
- Bias correction controls technical bias but chromVAR's clustering performance is still limited relative to newer methods — SnapATAC outperforms chromVAR for the clustering tasks evaluated in the paper, so this skill targets motif variability analysis, not cell embedding
- chromVAR install can fail via dependencies (gsl system library; GO.db occasionally not auto-installed), blocking the addGCBias step until they are installed separately

## Evidence

- [methods] The function `addGCBias` returns an updated SummarizedExperiment with a new rowData column named "bias": "The function `addGCBias` returns an updated SummarizedExperiment with a new rowData column named "bias""
- [methods] Background peaks are peaks that are similar to a peak in GC content and average accessibility.: "Background peaks are peaks that are similar to a peak in GC content and average accessibility."
- [abstract] chromVAR estimates gain or loss of accessibility within peaks sharing the same motif or annotation while controlling for technical biases: "estimating gain or loss of accessibility within peaks sharing the same motif or annotation while controlling for technical biases"
- [readme] example_counts <- addGCBias(example_counts, genome = BSgenome.Hsapiens.UCSC.hg19): "example_counts <- addGCBias(example_counts,
                              genome = BSgenome.Hsapiens.UCSC.hg19)"
