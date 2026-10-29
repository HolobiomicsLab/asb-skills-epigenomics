---
name: single-cell-tsne-embedding
description: Use when you have a chromVARDeviations object (per-cell motif deviation
  scores) from single-cell ATAC-seq data and need a 2D embedding/visualization of
  cell-to-cell similarity.
when_to_use_negative:
- 'If the goal is cluster identification or embedding as the primary purpose: the
  chromVAR authors note newer methods such as SnapATAC outperform chromVAR for clustering,
  and suggest chromVAR as a way to annotate TF motif usage in cells and clusters rather
  than cluster identification'
- Input is raw peaks or fragment counts not yet processed into bias-corrected deviation
  scores — the t-SNE step requires a chromVARDeviations object
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: chromVAR
  role: provides deviationsTsne and plotDeviationsTsne to embed and visualize cells
    from deviation scores
  repo: https://github.com/GreenleafLab/chromVAR
- name: motifmatchr
  role: matchMotifs finds which peaks contain which motifs, producing the motif annotation
    matrix needed for computeDeviations before t-SNE
  repo: https://github.com/GreenleafLab/motifmatchr
- name: SummarizedExperiment
  role: stores the single-cell ATAC-seq counts, GC bias, and sample colData (e.g.
    Cell_Type) feeding the embedding
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nmeth.4401
    title: 'chromVAR: inferring transcription-factor-associated accessibility from
      single-cell epigenomic data'
  source_content_hash: sha256:3dc483a2292c9ecbe718f94281c8ad91c484c6e0b9f528d93084da50bd3363d5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GreenleafLab/chromVAR
  tools:
  - R
  - SummarizedExperiment
  - chromVAR
  - motifmatchr
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-tSNE-embedding

## Summary

Embeds single-cell ATAC-seq profiles in two dimensions using chromVAR's per-cell bias-corrected motif deviation scores, via the deviationsTsne function, to visualize cell-to-cell similarity.

## When to use

Use when you have a chromVARDeviations object (per-cell motif deviation scores) from single-cell ATAC-seq data and need a 2D embedding/visualization of cell-to-cell similarity.

## When NOT to use

- If the goal is cluster identification or embedding as the primary purpose: the chromVAR authors note newer methods such as SnapATAC outperform chromVAR for clustering, and suggest chromVAR as a way to annotate TF motif usage in cells and clusters rather than cluster identification
- Input is raw peaks or fragment counts not yet processed into bias-corrected deviation scores — the t-SNE step requires a chromVARDeviations object

## Inputs

- single-cell ATAC-seq fragment counts as a SummarizedExperiment (e.g. example_counts from the chromVAR package)
- chromVARDeviations object from computeDeviations on motif-matched peaks
- BSgenome object (e.g. BSgenome.Hsapiens.UCSC.hg19) for GC bias correction and motif matching

## Outputs

- 2D t-SNE embedding data.frame with columns Dim1 and Dim2, one row per cell passing sample filters
- per-cell t-SNE plots (PNG) colored by annotation deviations and cell type via plotDeviationsTsne

## How to apply

Start from the chromVAR workflow: load example_counts or fragment counts, run addGCBias with BSgenome.Hsapiens.UCSC.hg19, filter samples with filterSamples (min_depth = 1500, min_in_peaks = 0.15), run filterPeaks, match JASPAR motifs with matchMotifs, and computeDeviations to obtain a chromVARDeviations object. Then call deviationsTsne on the deviation object with threshold = 1.5 and perplexity = 10 (shiny = FALSE) to embed cells in two dimensions, and plotDeviationsTsne with an annotation (e.g. 'TEAD3') and sample_column 'Cell_Type' to render per-cell t-SNE plots. Export the resulting t-SNE data.frame (columns Dim1, Dim2) to CSV for validation. Before running chromVAR functions, register a parallelization backend with BiocParallel (e.g. register(MulticoreParam(8))).

## Related tools

- **chromVAR** (provides deviationsTsne and plotDeviationsTsne to embed and visualize cells from deviation scores) — https://github.com/GreenleafLab/chromVAR
- **motifmatchr** (matchMotifs finds which peaks contain which motifs, producing the motif annotation matrix needed for computeDeviations before t-SNE) — https://github.com/GreenleafLab/motifmatchr
- **SummarizedExperiment** (stores the single-cell ATAC-seq counts, GC bias, and sample colData (e.g. Cell_Type) feeding the embedding)

## Examples

```
dev <- computeDeviations(counts_filtered, motif_ix); tsne <- deviationsTsne(dev, threshold = 1.5, perplexity = 10, shiny = FALSE)
```

## Evaluation signals

- tsne_coordinates.csv exists and contains the Dim1 and Dim2 fields
- the t-SNE data.frame has exactly one row per cell passing the filterSamples thresholds (min_depth = 1500, min_in_peaks = 0.15)
- the embedding is computed from a chromVARDeviations object produced with threshold = 1.5 and perplexity = 10
- plotDeviationsTsne renders both the annotation-deviation-colored and cell-type-colored (sample_column) t-SNE panels

## Limitations

- Newer methods such as SnapATAC outperform chromVAR for the clustering tasks evaluated; kmers + PCA appears to be the best variant of chromVAR for clustering
- Samples with insufficient reads or low proportion of reads in peaks must be filtered out before computing deviations, or the embedding will be distorted
- chromVAR may be complementary to other methods as a way of annotating TF motif usage in cells and clusters rather than cluster identification or embedding
- Installation issues: dependencies require the gsl system library, and C++ compilation problems have been reported on Mac; on Windows MulticoreParam does not work and SnowParam/SerialParam must be used

## Evidence

- [readme] Before running chromVAR functions, it is advisable to use the `register` function from BiocParallel to specify your preferred method of parallelization.: [evidence span withheld — anchor status: external_doc]
