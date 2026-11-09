---
name: chromatin-accessibility-deviation-computation
description: Use when analyzing sparse single-cell (or bulk) ATAC-seq fragment counts
  to identify motifs or genomic annotations associated with cell-to-cell variability
  in chromatin accessibility, or to derive per-cell deviation scores for downstream
  clustering/t-SNE.
when_to_use_negative:
- The goal is pure cell clustering or embedding rather than annotating TF motif usage
  — the chromVAR README notes that kmers + PCA is the best chromVAR variant for clustering
  and that SnapATAC outperforms chromVAR for the clustering tasks evaluated
- The input is already a matrix of per-cell deviation/deviation scores (e.g. feeding
  deviationsTsne) rather than raw fragment counts
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0749
tools:
- name: chromVAR
  role: Core R package providing addGCBias, filterSamples, filterPeaks, getBackgroundPeaks,
    computeDeviations, and computeVariability
  repo: github.com/GreenleafLab/chromVAR
- name: motifmatchr
  role: Matches PWMs to peaks via matchMotifs to produce the motif_matches annotation
    assay consumed by computeDeviations
  repo: github.com/GreenleafLab/motifmatchr
- name: SummarizedExperiment
  role: Container class for the filtered fragment-count matrix, bias rowData column,
    and motif_matches assay
- name: BiocParallel
  role: Registers the parallel backend (MulticoreParam/SnowParam/SerialParam) used
    by chromVAR functions
- name: chromVARmotifs
  role: Supplies additional motif collections such as human_pwms_v1, mouse_pwms_v1,
    homer_pwms, and encode_pwms
  repo: github.com/GreenleafLab/chromVARmotifs
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/nmeth.4401
    title: 'chromVAR: inferring transcription-factor-associated accessibility from
      single-cell epigenomic data'
  source_content_hash: sha256:788f563a118dc2565bcae9c1cb505e5153ba1d57097533c8f18986a1b6afcd4e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - BiocParallel
  - R
  - SummarizedExperiment
  - chromVAR
  - chromVARmotifs
  - motifmatchr
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# chromatin-accessibility-deviation-computation

## Summary

Compute chromVAR bias-corrected motif deviation scores and per-motif variability from sparse single-cell ATAC-seq counts: after GC-bias correction, sample/peak filtering, and motif matching with motifmatchr, computeDeviations yields per-cell 'deviations' and Z-score 'deviationScores' assays, and computeVariability ranks motifs by standard deviation of Z-scores.

## When to use

Use when analyzing sparse single-cell (or bulk) ATAC-seq fragment counts to identify motifs or genomic annotations associated with cell-to-cell variability in chromatin accessibility, or to derive per-cell deviation scores for downstream clustering/t-SNE.

## When NOT to use

- The goal is pure cell clustering or embedding rather than annotating TF motif usage — the chromVAR README notes that kmers + PCA is the best chromVAR variant for clustering and that SnapATAC outperforms chromVAR for the clustering tasks evaluated
- The input is already a matrix of per-cell deviation/deviation scores (e.g. feeding deviationsTsne) rather than raw fragment counts

## Inputs

- RangedSummarizedExperiment of single-cell ATAC-seq fragment counts per peak (e.g. example_counts from the chromVAR package)
- Position weight matrices (PWMatrixList/PFMatrixList) from JASPAR via getJasparMotifs or chromVARmotifs data(human_pwms_v1)
- Genome sequence for bias correction and motif matching (BSgenome.Hsapiens.UCSC.hg19 or hg19 FASTA)
- Optional GenomicRanges peak set / narrowPeak file via getPeaks and readNarrowpeaks

## Outputs

- chromVARDeviations object with 'deviations' and 'deviationScores' (Z-score) assays
- motif_variability.csv data.frame with per-motif variability, bootstrap confidence intervals, and p-values
- Filtered SummarizedExperiment with 'bias' rowData column and 'motif_matches' assay

## How to apply

Start from a RangedSummarizedExperiment of fragment counts (e.g. chromVAR's example_counts) and add a GC bias term with addGCBias against BSgenome.Hsapiens.UCSC.hg19, producing the 'bias' rowData column. Filter samples with filterSamples (min_depth = 1500, min_in_peaks = 0.15) and filterPeaks (non_overlapping = TRUE) so peaks retain at least one fragment and overlapping peaks are removed. Match a PWM collection (getJasparMotifs from JASPAR core, or human_pwms_v1 from chromVARmotifs) to peaks with matchMotifs from motifmatchr at the default p.cutoff of 0.00005 to get the motif_matches assay. Compute background peaks matched on GC content and average accessibility with getBackgroundPeaks, then call computeDeviations to obtain the chromVARDeviations object containing 'deviations' and 'deviationScores' (Z-score) assays. Finally quantify per-motif variability with computeVariability, which computes the standard deviation of Z-scores across cells, bootstrap confidence intervals, and a p-value against the null hypothesis of variability equal to 1; register a BiocParallel backend before running.

## Related tools

- **chromVAR** (Core R package providing addGCBias, filterSamples, filterPeaks, getBackgroundPeaks, computeDeviations, and computeVariability) — github.com/GreenleafLab/chromVAR
- **motifmatchr** (Matches PWMs to peaks via matchMotifs to produce the motif_matches annotation assay consumed by computeDeviations) — github.com/GreenleafLab/motifmatchr
- **SummarizedExperiment** (Container class for the filtered fragment-count matrix, bias rowData column, and motif_matches assay)
- **BiocParallel** (Registers the parallel backend (MulticoreParam/SnowParam/SerialParam) used by chromVAR functions)
- **chromVARmotifs** (Supplies additional motif collections such as human_pwms_v1, mouse_pwms_v1, homer_pwms, and encode_pwms) — github.com/GreenleafLab/chromVARmotifs

## Examples

```
dev <- computeDeviations(object = counts_filtered, annotations = motif_ix)
```

## Evaluation signals

- motif_variability.csv exists and is a data.frame with per-motif variability, standard deviation of Z-scores, bootstrap CI, and a p-value testing variability = 1
- The chromVARDeviations object contains both 'deviations' and 'deviationScores' assays with one column per cell passing filterSamples thresholds (min_depth 1500, min_in_peaks 0.15)
- Peaks retain at least one fragment and the peak set is non-overlapping after filterPeaks (non_overlapping = TRUE)
- A 'bias' rowData column is present after addGCBias and a 'motif_matches' assay after matchMotifs (p.cutoff 0.00005)
- Background peaks returned by getBackgroundPeaks are matched on GC content and average accessibility

## Limitations

- One dependency requires the gsl system library; installation may fail without it, and GO.db sometimes needs separate installation (BiocManager::install("GO.db"))
- For clustering tasks, SnapATAC outperforms chromVAR; kmers + PCA is the best chromVAR variant, so chromVAR is best used to annotate TF motif usage rather than identify clusters
- MulticoreParam does not work on Windows; a SnowParam or explicitly registered SerialParam backend is needed
- Sample filtering is advisable because samples with insufficient reads or a low proportion of reads in peaks degrade deviation estimates

## Evidence

- [abstract] chromVAR estimates gain or loss of accessibility within peaks sharing the same motif or annotation while controlling for technical biases: "estimating gain or loss of accessibility within peaks sharing the same motif or annotation while controlling for technical biases"
- [methods] addGCBias adds a bias rowData column to the SummarizedExperiment: "The function `addGCBias` returns an updated SummarizedExperiment with a new rowData column named "bias""
- [methods] Two filtering parameters min_in_peaks and min_depth control sample filtering: "Two parameters are used for filtering -- min_in_peaks and min_depth."
- [methods] Background peaks are matched on GC content and average accessibility: "Background peaks are peaks that are similar to a peak in GC content and average accessibility."
- [methods] matchMotifs from motifmatchr assigns motifs to peaks for computeDeviations: "The function `matchMotifs` from the motifmatchr package finds which peaks contain which motifs."
- [readme] README quickstart shows the full computeDeviations invocation on example_counts: "data(example_counts, package = "chromVAR")"
- [readme] README advises registering a BiocParallel backend before running chromVAR: [evidence span withheld — anchor status: external_doc]
