---
name: perplexity-parameter-tuning-for-tsne
description: Use when visualizing cell-to-cell similarity of single-cell ATAC-seq
  profiles via chromVAR deviationsTsne and a perplexity must be set for the number
  of filtered cells (e.g., perplexity = 10 on small filtered sets).
when_to_use_negative:
- 'The goal is cluster identification or a global embedding of a large scATAC-seq
  dataset: the paper notes newer methods such as SnapATAC outperform chromVAR for
  the clustering tasks evaluated, and chromVAR is better used to annotate TF motif
  usage in cells and clusters.'
- The input is raw fragment counts rather than a chromVARDeviations object of bias-corrected
  deviation scores — computeDeviations must run first, since deviationsTsne operates
  on the deviation object.
- The number of cells after filtering is large enough that the small-set perplexity
  value (10) is inappropriate; the perplexity=10 setting is tailored to the filtered
  example_counts set and should be revisited for bigger datasets.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: chromVAR
  role: Provides deviationsTsne (embedding of deviation scores) and plotDeviationsTsne
    (rendering per-cell t-SNE colored by annotation and cell type), plus computeDeviations
    to generate the input deviation object
  repo: https://github.com/GreenleafLab/chromVAR
- name: motifmatchr
  role: matchMotifs finds which filtered peaks contain which JASPAR motifs, producing
    the annotations matrix required before deviations can be computed and embedded
  repo: https://github.com/GreenleafLab/motifmatchr
- name: SummarizedExperiment
  role: Container for the filtered single-cell ATAC-seq counts and GC-bias rowData
    that feeds computeDeviations upstream of the t-SNE step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nmeth.4401
    title: 'chromVAR: inferring transcription-factor-associated accessibility from
      single-cell epigenomic data'
  source_content_hash: sha256:cefd1073b8f39c0d50b22f376623740acaa6c1e779ab1d7086148fe2b93cd380
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GreenleafLab/chromVAR
  tools:
  - R
  - SummarizedExperiment
  - chromVAR
  - motifmatchr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Perplexity parameter tuning for t-SNE

## Summary

Embeds chromVAR's per-cell, bias-corrected motif deviation scores in 2D with the deviationsTsne function, choosing a perplexity appropriate to the number of cells surviving sample filtering (perplexity = 10, threshold = 1.5 in the chromVAR example_counts workflow).

## When to use

Use when visualizing cell-to-cell similarity of single-cell ATAC-seq profiles via chromVAR deviationsTsne and a perplexity must be set for the number of filtered cells (e.g., perplexity = 10 on small filtered sets).

## When NOT to use

- The goal is cluster identification or a global embedding of a large scATAC-seq dataset: the paper notes newer methods such as SnapATAC outperform chromVAR for the clustering tasks evaluated, and chromVAR is better used to annotate TF motif usage in cells and clusters.
- The input is raw fragment counts rather than a chromVARDeviations object of bias-corrected deviation scores — computeDeviations must run first, since deviationsTsne operates on the deviation object.
- The number of cells after filtering is large enough that the small-set perplexity value (10) is inappropriate; the perplexity=10 setting is tailored to the filtered example_counts set and should be revisited for bigger datasets.

## Inputs

- chromVARDeviations object produced by computeDeviations on motif annotations from matchMotifs (JASPAR motifs)
- RangedSummarizedExperiment of single-cell ATAC-seq fragment counts filtered with filterSamples(min_depth = 1500, min_in_peaks = 0.15) and filterPeaks
- GC-bias annotations added via addGCBias against BSgenome.Hsapiens.UCSC.hg19

## Outputs

- Per-cell 2D t-SNE embedding as a data.frame with columns Dim1 and Dim2 (exported as tsne_coordinates.csv, one row per cell passing sample filters)
- Static t-SNE plots from plotDeviationsTsne colored by annotation deviations (e.g., TEAD3) and by Cell_Type, exported as PNG files

## How to apply

First compute the chromVARDeviations object: load example_counts from the chromVAR package, add GC bias with addGCBias against BSgenome.Hsapiens.UCSC.hg19, drop weak samples with filterSamples(min_depth = 1500, min_in_peaks = 0.15), run filterPeaks, match JASPAR motifs with matchMotifs, and call computeDeviations. Then run deviationsTsne on the deviation object with threshold = 1.5 (restricting the embedding to motifs with sufficient deviation variability) and perplexity = 10, which is sized for the small post-filter cell set rather than the default large-set perplexity, setting shiny = FALSE for a static run. The rationale is that chromVAR's abstract-level finding — that its deviation scores enable accurate clustering of scATAC-seq profiles — makes the deviation matrix a valid similarity input, while perplexity must remain compatible with the number of cells being embedded. Finally, render per-cell views with plotDeviationsTsne using a motif annotation (e.g., 'TEAD3') and sample_column 'Cell_Type' to check that the embedding separates known cell types, and write the Dim1/Dim2 coordinates to tsne_coordinates.csv.

## Related tools

- **chromVAR** (Provides deviationsTsne (embedding of deviation scores) and plotDeviationsTsne (rendering per-cell t-SNE colored by annotation and cell type), plus computeDeviations to generate the input deviation object) — https://github.com/GreenleafLab/chromVAR
- **motifmatchr** (matchMotifs finds which filtered peaks contain which JASPAR motifs, producing the annotations matrix required before deviations can be computed and embedded) — https://github.com/GreenleafLab/motifmatchr
- **SummarizedExperiment** (Container for the filtered single-cell ATAC-seq counts and GC-bias rowData that feeds computeDeviations upstream of the t-SNE step)

## Examples

```
dev <- computeDeviations(counts_filtered, motif_ix); tsne_coords <- deviationsTsne(dev, threshold = 1.5, perplexity = 10, shiny = FALSE)
```

## Evaluation signals

- tsne_coordinates.csv exists, contains Dim1 and Dim2 fields, and has exactly one row per cell that passed filterSamples (min_depth = 1500, min_in_peaks = 0.15).
- Perplexity is compatible with the embedding: it must not exceed the number of cells retained after filtering and thresholding (perplexity = 10 for the small chromVAR example set with threshold = 1.5).
- plotDeviationsTsne colored by sample_column 'Cell_Type' shows cells grouping by known cell type, consistent with the finding that chromVAR deviation scores enable accurate clustering of scATAC-seq profiles.
- The embedding uses only motifs passing threshold = 1.5; motifs with insufficient deviation variability are excluded from the t-SNE input.
- shiny = FALSE yields a static embedding reproducible as a Dim1/Dim2 data.frame rather than an interactive session.

## Limitations

- chromVAR's t-SNE/clustering use is bounded: the paper reports newer methods such as SnapATAC outperform chromVAR for the clustering tasks evaluated, and kmers + PCA is the best chromVAR variant for clustering — so the embedding is best treated as motif-usage annotation rather than definitive cluster identification.
- Samples with insufficient reads or a low proportion of reads in peaks must be filtered out first; embedding unfiltered or sparse cells degrades the similarity structure.
- Perplexity is cell-count dependent: the value 10 from the example workflow is suited to the small filtered example_counts set and is not a universal default.

## Evidence
