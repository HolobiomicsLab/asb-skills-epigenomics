---
name: motif-variability-quantification
description: Use when you have a sparse scATAC-seq fragment-count matrix over peaks
  and want to rank motifs/annotations by cell-to-cell accessibility variability with
  bias correction, or to test whether motif deviations or variability differ between
  cell-type groups.
when_to_use_negative:
- Your goal is clustering or embedding of single cells — SnapATAC outperforms chromVAR
  for the clustering tasks evaluated in the paper; chromVAR is better for annotating
  TF motif usage within cells and clusters.
- Your data are bulk with high depth where sparse-count bias correction is less critical,
  or input is already a feature/deviation table rather than raw fragment counts over
  peaks.
- You lack a suitable BSgenome for the genome build of your peaks, so addGCBias and
  matchMotifs cannot be performed.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0749
tools:
- name: chromVAR
  role: 'Core R package: addGCBias, filterSamples, filterPeaks, getBackgroundPeaks,
    computeDeviations, computeVariability, differentialDeviations, differentialVariability'
  repo: https://github.com/GreenleafLab/chromVAR
- name: motifmatchr
  role: Fast motif matching (matchMotifs) to find which peaks contain which motifs,
    producing the motif_matches assay used by computeDeviations
  repo: https://github.com/GreenleafLab/motifmatchr
- name: SummarizedExperiment
  role: Container class for the peak x cell counts and colData/rowData metadata passed
    through the workflow
- name: BiocParallel
  role: Registers the parallel backend (MulticoreParam, SnowParam, or SerialParam)
    used by chromVAR functions
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Reference genome sequence for GC-bias computation and motif-to-peak matching
- name: chromVARmotifs
  role: Supplies PWM motif collections (human_pwms_v1, mouse_pwms_v1, homer_pwms,
    encode_pwms) as alternative inputs to matchMotifs
  repo: https://github.com/GreenleafLab/chromVARmotifs
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/nmeth.4401
    title: 'chromVAR: inferring transcription-factor-associated accessibility from
      single-cell epigenomic data'
  source_content_hash: sha256:5f26bbc9a98404795942e0d3f10a4a5bd9e1aec6a6fe3adb38d567a8f5aea614
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GreenleafLab/chromVAR
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - BiocParallel
  - R
  - SummarizedExperiment
  - chromVAR
  - chromVARmotifs
  - motifmatchr
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# motif-variability-quantification

## Summary

Quantify per-motif bias-corrected deviation scores and variability (standard deviation of deviation Z-scores) from sparse single-cell ATAC-seq counts using the chromVAR R package, to identify motifs or genomic annotations associated with variability in chromatin accessibility between individual cells.

## When to use

Use when you have a sparse scATAC-seq fragment-count matrix over peaks and want to rank motifs/annotations by cell-to-cell accessibility variability with bias correction, or to test whether motif deviations or variability differ between cell-type groups.

## When NOT to use

- Your goal is clustering or embedding of single cells — SnapATAC outperforms chromVAR for the clustering tasks evaluated in the paper; chromVAR is better for annotating TF motif usage within cells and clusters.
- Your data are bulk with high depth where sparse-count bias correction is less critical, or input is already a feature/deviation table rather than raw fragment counts over peaks.
- You lack a suitable BSgenome for the genome build of your peaks, so addGCBias and matchMotifs cannot be performed.

## Inputs

- Sparse peak x cell fragment-count matrix as a RangedSummarizedExperiment (e.g. chromVAR example_counts, or counts built from BAM/bed files via getCounts and getPeaks)
- Peak set as GenomicRanges (optionally from narrowPeak via readNarrowpeaks)
- Reference genome sequence (BSgenome.Hsapiens.UCSC.hg19) for GC bias and motif matching
- PWM motif collection (JASPAR via getJasparMotifs, or chromVARmotifs human_pwms_v1 / mouse_pwms_v1 / homer_pwms / encode_pwms)
- Per-cell metadata in colData, e.g. a Cell_Type column for differential tests

## Outputs

- chromVARDeviations object with 'deviations' (bias-corrected) and 'deviationScores' (Z-score) assays, cells x motifs
- motif_variability.csv: data.frame of per-motif variability (SD of Z-scores), bootstrap confidence intervals, and p-values
- differential_deviations.csv / differentialVariability result tables with per-motif differential statistics and p-values between cell-type groups
- motif_matches assay (boolean motif x peak matrix) from matchMotifs

## How to apply

Load the peak x cell counts as a RangedSummarizedExperiment (e.g. chromVAR's example_counts) and add a 'bias' rowData column via addGCBias against BSgenome.Hsapiens.UCSC.hg19. Filter low-quality samples with filterSamples (min_depth = 1500, min_in_peaks = 0.15, shiny = FALSE) and reduce to non-overlapping peaks observed in at least one fragment with filterPeaks(non_overlapping = TRUE). Match JASPAR motifs (getJasparMotifs or chromVARmotifs human_pwms_v1) to peaks with matchMotifs from motifmatchr at the default p.cutoff of 0.00005, then compute background peaks matched on GC content and average accessibility with getBackgroundPeaks and pass both to computeDeviations to obtain the chromVARDeviations object with 'deviations' and 'deviationScores' (Z-score) assays. Finally run computeVariability to get per-motif variability (SD of Z-scores across cells), bootstrap confidence intervals, and a p-value against the null hypothesis of variability equal to 1; for grouped designs, use differentialVariability and differentialDeviations with the cell-type column (e.g. 'Cell_Type'). Register a BiocParallel backend (e.g. register(MulticoreParam(8)) on unix) before running chromVAR functions.

## Related tools

- **chromVAR** (Core R package: addGCBias, filterSamples, filterPeaks, getBackgroundPeaks, computeDeviations, computeVariability, differentialDeviations, differentialVariability) — https://github.com/GreenleafLab/chromVAR
- **motifmatchr** (Fast motif matching (matchMotifs) to find which peaks contain which motifs, producing the motif_matches assay used by computeDeviations) — https://github.com/GreenleafLab/motifmatchr
- **SummarizedExperiment** (Container class for the peak x cell counts and colData/rowData metadata passed through the workflow)
- **BiocParallel** (Registers the parallel backend (MulticoreParam, SnowParam, or SerialParam) used by chromVAR functions)
- **BSgenome.Hsapiens.UCSC.hg19** (Reference genome sequence for GC-bias computation and motif-to-peak matching)
- **chromVARmotifs** (Supplies PWM motif collections (human_pwms_v1, mouse_pwms_v1, homer_pwms, encode_pwms) as alternative inputs to matchMotifs) — https://github.com/GreenleafLab/chromVARmotifs

## Examples

```
dev <- computeDeviations(object = counts_filtered, annotations = motif_ix); variability <- computeVariability(dev); write.csv(variability, "motif_variability.csv")
```

## Evaluation signals

- motif_variability.csv exists and contains one row per matched motif with variability, bootstrap CI, and p-value columns from computeVariability
- The chromVARDeviations object contains both 'deviations' and 'deviationScores' assays with motifs as rows/features and cells as columns after filtering
- Differential output tables contain rows for each tested motif and include the per-motif differential test statistics and p-value columns from differentialDeviations/differentialVariations
- filterSamples removed cells below min_depth = 1500 or min_in_peaks = 0.15, and filterPeaks returned a non-overlapping peak set with each peak having at least one fragment
- The 'bias' rowData column is populated after addGCBias and the motif_matches assay is a boolean motif x peak matrix

## Limitations

- chromVAR handles sparse scATAC-seq data but for clustering tasks newer methods such as SnapATAC outperform chromVAR; it may be complementary for annotating TF motif usage rather than cluster identification or embedding.
- Installation dependencies have system requirements (gsl library) and some dependencies (e.g. GO.db, S4Vector on Windows R 3.3.3, C++ compilation on Mac) have known installation issues reported in the chromVAR README.
- Parallelization must be registered explicitly with BiocParallel; MulticoreParam does not work on Windows (use SnowParam or SerialParam).

## Evidence

- [readme] README advises registering BiocParallel backend before running chromVAR functions: [evidence span withheld — anchor status: external_doc]
