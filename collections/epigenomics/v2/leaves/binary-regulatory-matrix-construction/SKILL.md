---
name: binary-regulatory-matrix-construction
description: Use when building a chromatin-informed TF-target prior to constrain decoder
  weights of a transcriptional-regulator model, from ATAC-seq open-chromatin peaks
  and a motif reference.
when_to_use_negative:
- The input is already a packaged TF-target prior (e.g. dataset_CITRUS.pkl with its
  TF-target gene matrix included) — re-deriving motifs would duplicate work
- Directionality of regulation (activation vs repression) is required — the prior
  is strictly binary and the paper does not represent activation/repression
- Working with a cancer type outside the 17 TCGA tumor types used for the expression-retention
  filter
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: CITRUS
  role: Downstream model whose decoder hidden-node (TF) to gene output connections
    are constrained by this binary prior matrix; exposes the trained constrained weights
    as tf_gene
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: FIMO
  role: Scans pan-cancer ATAC-seq peak sequences against the MEME-curated cis-BP motifs
    at P < 1e-5 to link TF motifs to peaks
- name: ChIPpeakAnno
  role: Assigns each motif-bearing peak to its nearest gene via the nearest-distance
    rule
- name: Pandas
  role: Used with Python/NumPy/SciPy to assemble, binarize, and package the 320 x
    5541 matrix as tf_target_matrix.pkl
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:bc6edefc292a148f9e50cce3cecc0e0f8e9bb7a554cf1daa5f25506867f32bba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/osmanbeyoglulab/CITRUS
  tools:
  - CITRUS
  - ChIPpeakAnno
  - FIMO
  - Numpy
  - Pandas
  - Python
  - SciPy
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7727272727272727
n_claims: 22
---

# binary-regulatory-matrix-construction

## Summary

Constructs the binary TF-target gene prior matrix (320 TF motifs x 5541 genes) that constrains the decoder's TF-to-gene connections in CITRUS, by linking TF binding motifs to open-chromatin peaks from the pan-cancer TCGA ATAC-seq atlas and assigning peaks to genes.

## When to use

Use when building a chromatin-informed TF-target prior to constrain decoder weights of a transcriptional-regulator model, from ATAC-seq open-chromatin peaks and a motif reference.

## When NOT to use

- The input is already a packaged TF-target prior (e.g. dataset_CITRUS.pkl with its TF-target gene matrix included) — re-deriving motifs would duplicate work
- Directionality of regulation (activation vs repression) is required — the prior is strictly binary and the paper does not represent activation/repression
- Working with a cancer type outside the 17 TCGA tumor types used for the expression-retention filter

## Inputs

- Pan-cancer ATAC-seq open-chromatin peak atlas (~200K peaks) from TCGA Genomic Data Commons ATACseq-AWG release
- TCGA gene expression matrix (RNA-seq, Firehose run) for TF expression filtering across 17 tumor types
- MEME-curated cis-BP motif reference (PWM file for FIMO)
- Filtered gene list (5541 top-variant genes per cancer type)

## Outputs

- tf_target_matrix.pkl: 320 x 5541 binary {0,1} TF-target prior matrix consumed by the CITRUS decoder
- Set of 320 deduplicated TF motifs retained after Jaccard-similarity filtering

## How to apply

Download the pan-cancer ATAC-seq open-chromatin peak atlas (~200K peaks) from the TCGA Genomic Data Commons ATACseq-AWG release and scan all peak sequences with FIMO against the MEME-curated cis-BP motif reference at significance threshold P < 1e-5 to link TF motifs to open-chromatin peaks. Retain only TFs whose genes are expressed in >= 50% of samples in at least one of the 17 TCGA tumor types, then deduplicate near-identical motifs by removing TF motif pairs with Jaccard similarity > 0.5, yielding 320 TF motifs. Assign each motif-bearing peak to its nearest gene with the ChIPpeakAnno nearest-distance rule and binarize the assignments into a {0,1} matrix restricted to the 5541 filtered genes (the same gene set selected as top variant genes per cancer type with gene availability in the TF-target matrix). Package the 320 x 5541 binary matrix as tf_target_matrix.pkl for use as the decoder constraint; in CITRUS the trained decoder weight is exposed as 'tf_gene' via model.layer_w_2.weight.

## Related tools

- **CITRUS** (Downstream model whose decoder hidden-node (TF) to gene output connections are constrained by this binary prior matrix; exposes the trained constrained weights as tf_gene) — https://github.com/osmanbeyoglulab/CITRUS
- **FIMO** (Scans pan-cancer ATAC-seq peak sequences against the MEME-curated cis-BP motifs at P < 1e-5 to link TF motifs to peaks)
- **ChIPpeakAnno** (Assigns each motif-bearing peak to its nearest gene via the nearest-distance rule)
- **Pandas** (Used with Python/NumPy/SciPy to assemble, binarize, and package the 320 x 5541 matrix as tf_target_matrix.pkl)

## Examples

```
import pandas as pd; m = pd.read_pickle('tf_target_matrix.pkl'); assert set(pd.unique(m.values.ravel())) <= {0, 1} and m.shape == (320, 5541)
```

## Evaluation signals

- Shape check: the saved matrix must have exactly 320 TF motif rows x 5541 gene columns
- Value-domain check: all entries of the matrix must be in {0,1}
- TF retention check: every retained TF's gene is expressed in >= 50% of samples in at least one of the 17 TCGA tumor types
- Deduplication check: no remaining TF motif pair has Jaccard similarity > 0.5
- Consistency check: matrix gene axis matches the 5541 filtered genes used in the CITRUS GEP

## Limitations

- TFs of the same family often share a similar motif and thus are difficult to disambiguate, so a TF motif may encompass the activities of multiple TFs
- No directionality is represented — whether a gene is activated or repressed by a TF is not encoded in the prior
- Co-binding TF binding patterns (e.g. AP-1-IRF complexes) are biologically meaningful but not represented in the model
- Tumor regulatory network analysis is complicated by stromal/immune cells within the tumor and cancer cell heterogeneity

## Evidence

- [abstract] CITRUS uses a layer of hidden nodes to explicitly represent the state of transcription factors (TFs) to learn the relationships between TFs and their target genes based on TF binding motifs: "CITRUS uses a layer of hidden nodes to explicitly represent the state of transcription factors (TFs) to learn the relationships between TFs and their target genes based on TF binding motifs"
- [discussion] For a more accurate representation of TF-target gene relationships, we leveraged ATAC-seq tumor data from TCGA patients.: "For a more accurate representation of TF-target gene relationships, we leveraged ATAC-seq tumor data from TCGA patients."
- [supplementary] ATAC-seq data are available in the public repository Genomic Data Commons (https://gdc.cancer.gov/about-data/publications/ATACseq-AWG).: "ATAC-seq data are available in the public repository Genomic Data Commons (https://gdc.cancer.gov/about-data/publications/ATACseq-AWG)."
- [readme] GEP contains continuous values of gene expression with filtered 5541 genes by considering top variant genes in each cancer type as well as gene availability in TF-target gene matrix.: "GEP contains continuous values of gene expression with filtered 5541 genes by considering top variant genes in each cancer type as well as gene availability in TF-target gene matrix."
- [readme] "tf_gene": model.layer_w_2.weight.data.cpu().numpy(),  # trained weight of tf_gene constrains: ""tf_gene": model.layer_w_2.weight.data.cpu().numpy(),  # trained weight of tf_gene constrains"
