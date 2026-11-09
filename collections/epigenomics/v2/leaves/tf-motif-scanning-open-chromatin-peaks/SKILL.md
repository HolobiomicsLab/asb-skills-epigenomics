---
name: tf-motif-scanning-open-chromatin-peaks
description: 'Use when building a TF-target gene prior from open-chromatin data: e.g.,
  you have an ATAC-seq peak atlas plus a gene expression matrix and need a binary
  {0,1} TF-by-gene matrix to constrain a model''s TF-to-gene layer.'
when_to_use_negative:
- You need directional TF-target relationships (activation vs. repression) — CITRUS
  explicitly does not represent directionality in the TF-target gene priors.
- You must resolve activities of individual TFs from the same family — TFs of the
  same family often share a similar motif and cannot be disambiguated, so motifs may
  encompass multiple TFs.
- Your input has no open-chromatin (ATAC-seq) peaks, since the prior is derived from
  TF binding motifs in open chromatin regions.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3173
tools:
- name: CITRUS
  role: Downstream model whose decoder's TF-to-gene layer is constrained by the binary
    prior matrix produced by this skill
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: FIMO
  role: Scans ATAC-seq peak sequences against the MEME-curated cis-BP motif reference
    at P < 1e-5
- name: ChIPpeakAnno
  role: Assigns each motif-bearing peak to its nearest gene via the nearest-distance
    rule
- name: Pandas
  role: Packages and serializes the 320 x 5541 binary matrix as tf_target_matrix.pkl
- name: Numpy
  role: Array manipulation and binarization of TF-target assignments
- name: SciPy
  role: Supporting computations for the prior-matrix construction pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:6b327a0b1ffd2d2213853ec62ecf47ea9748c2e2a30039bce8ec4e5a012cea3a
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
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# tf-motif-scanning-open-chromatin-peaks

## Summary

Constructs a binary TF-target gene prior matrix by scanning pan-cancer ATAC-seq open-chromatin peaks for TF motif occurrences (FIMO vs. MEME-curated cis-BP motifs, P < 1e-5) and assigning motif-bearing peaks to nearest genes. This prior constrains the decoder's TF-to-gene connections in CITRUS.

## When to use

Use when building a TF-target gene prior from open-chromatin data: e.g., you have an ATAC-seq peak atlas plus a gene expression matrix and need a binary {0,1} TF-by-gene matrix to constrain a model's TF-to-gene layer.

## When NOT to use

- You need directional TF-target relationships (activation vs. repression) — CITRUS explicitly does not represent directionality in the TF-target gene priors.
- You must resolve activities of individual TFs from the same family — TFs of the same family often share a similar motif and cannot be disambiguated, so motifs may encompass multiple TFs.
- Your input has no open-chromatin (ATAC-seq) peaks, since the prior is derived from TF binding motifs in open chromatin regions.

## Inputs

- Pan-cancer ATAC-seq open-chromatin peak atlas (~200K peaks) from TCGA Genomic Data Commons ATACseq-AWG release
- TCGA gene expression matrix (RNA-seq, Firehose run)
- MEME-curated cis-BP motif reference for FIMO

## Outputs

- Binary 320 x 5541 TF-target gene prior matrix (tf_target_matrix.pkl) with {0,1} entries
- Filtered set of 320 non-redundant TF motifs

## How to apply

Download the pan-cancer ATAC-seq open-chromatin peak atlas (~200K peaks) from the TCGA Genomic Data Commons ATACseq-AWG release together with the TCGA gene expression matrix. Scan all peak sequences with FIMO against the MEME-curated cis-BP motif reference at significance threshold P < 1e-5 to link TF motifs to open-chromatin peaks. Retain only TFs whose genes are expressed in >= 50% of samples in at least one of the 17 TCGA tumor types, and deduplicate near-identical motifs by removing TF motif pairs with Jaccard similarity > 0.5, yielding 320 TF motifs. Assign each motif-bearing peak to its nearest gene with the ChIPpeakAnno nearest-distance rule, binarize assignments into a {0,1} matrix restricted to the 5541 filtered genes (top variant genes per cancer type with gene availability in the TF-target matrix), and save as tf_target_matrix.pkl (320 x 5541).

## Related tools

- **CITRUS** (Downstream model whose decoder's TF-to-gene layer is constrained by the binary prior matrix produced by this skill) — https://github.com/osmanbeyoglulab/CITRUS
- **FIMO** (Scans ATAC-seq peak sequences against the MEME-curated cis-BP motif reference at P < 1e-5)
- **ChIPpeakAnno** (Assigns each motif-bearing peak to its nearest gene via the nearest-distance rule)
- **Pandas** (Packages and serializes the 320 x 5541 binary matrix as tf_target_matrix.pkl)
- **Numpy** (Array manipulation and binarization of TF-target assignments)
- **SciPy** (Supporting computations for the prior-matrix construction pipeline)

## Examples

```
import pandas as pd; C = pd.read_pickle("tf_target_matrix.pkl"); assert C.shape == (320, 5541) and set(pd.Series(C.values.ravel()).unique()) <= {0, 1}
```

## Evaluation signals

- Saved matrix contains only {0,1} entries (no fractional or NA values)
- Matrix dimensions are exactly 320 TF motifs x 5541 genes, matching CITRUS training requirements
- Retained TF set respects the >= 50% expression filter across at least one of the 17 TCGA tumor types
- No TF motif pairs with Jaccard similarity > 0.5 remain after deduplication
- Every gene in the matrix column set appears in the 5541-gene filtered GEP used by CITRUS

## Limitations

- TFs of the same family often share a similar motif and thus are difficult to disambiguate; TF motifs may encompass the activities of multiple TFs.
- No directionality (activation vs. repression) is represented in the TF-target gene priors.
- Co-binding TF binding patterns (e.g. AP-1-IRF complexes) are not represented in the model.

## Evidence

- [other] The TF-target prior is built by scanning the pan-cancer ATAC-seq peak atlas for TF motif occurrences using the MEME-curated cis-BP motif reference with FIMO at a significance threshold of P < 10^-5.: [evidence span withheld — anchor status: not_found]
- [discussion] For a more accurate representation of TF-target gene relationships, we leveraged ATAC-seq tumor data from TCGA patients.: "For a more accurate representation of TF-target gene relationships, we leveraged ATAC-seq tumor data from TCGA patients."
- [readme] GEP contains continuous values of gene expression with filtered 5541 genes by considering top variant genes in each cancer type as well as gene availability in TF-target gene matrix.: "GEP contains continuous values of gene expression with filtered 5541 genes by considering top variant genes in each cancer type as well as gene availability in TF-target gene matrix."
- [abstract] CITRUS uses a layer of hidden nodes to explicitly represent the state of transcription factors (TFs) to learn the relationships between TFs and their target genes based on TF binding motifs: "CITRUS uses a layer of hidden nodes to explicitly represent the state of transcription factors (TFs) to learn the relationships between TFs and their target genes based on TF binding motifs"
