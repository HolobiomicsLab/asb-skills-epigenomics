---
name: atacseq-peak-gene-annotation
description: Use when you need to derive a TF-target gene prior from open-chromatin
  ATAC-seq peaks (e.g. the TCGA Genomic Data Commons ATACseq-AWG atlas) to constrain
  a neural decoder's TF-to-gene layer, as in CITRUS.
when_to_use_negative:
- You already have a curated TF-target gene matrix (e.g. packaged in dataset_CITRUS.pkl)
  — regenerating the prior from peaks is redundant
- Your analysis requires TF-target directionality (activation vs. repression) or co-binding
  TF complexes (e.g. AP-1–IRF) — this binary prior intentionally omits both
- You need to disambiguate TFs of the same family sharing near-identical motifs —
  the Jaccard > 0.5 deduplication collapses them
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0659
tools:
- name: CITRUS
  role: Downstream model whose decoder's TF-to-gene layer is masked by the binary
    prior produced here
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: FIMO
  role: Scans ATAC-seq peak sequences against the MEME-curated cis-BP motif reference
    at P < 1e-5 to link TF motifs to open-chromatin peaks
- name: ChIPpeakAnno
  role: Assigns each motif-bearing peak to its nearest gene via the nearest-distance
    rule before binarization
- name: Pandas
  role: Data handling for the expression matrix, peak assignments, and packaging the
    prior into a pickle file
- name: Numpy
  role: Array operations for binarizing TF-target assignments and validating matrix
    contents
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:81ae6c26ccbe4df024f4d9e33f1ab5e0c9a088c56cab2fe6b7ecb60f724ef4a5
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# ATAC-seq peak-to-gene annotation for TF-target prior construction

## Summary

Constructs a binary TF-target gene prior matrix by scanning pan-cancer ATAC-seq open-chromatin peaks for TF motifs (FIMO vs. MEME-curated cis-BP at P < 1e-5), then assigning each motif-bearing peak to its nearest gene and binarizing into a {0,1} matrix (320 TF motifs x 5541 genes) that constrains CITRUS's decoder TF-to-gene connections.

## When to use

Use when you need to derive a TF-target gene prior from open-chromatin ATAC-seq peaks (e.g. the TCGA Genomic Data Commons ATACseq-AWG atlas) to constrain a neural decoder's TF-to-gene layer, as in CITRUS.

## When NOT to use

- You already have a curated TF-target gene matrix (e.g. packaged in dataset_CITRUS.pkl) — regenerating the prior from peaks is redundant
- Your analysis requires TF-target directionality (activation vs. repression) or co-binding TF complexes (e.g. AP-1–IRF) — this binary prior intentionally omits both
- You need to disambiguate TFs of the same family sharing near-identical motifs — the Jaccard > 0.5 deduplication collapses them

## Inputs

- Pan-cancer ATAC-seq open-chromatin peak atlas (~200K peaks) from TCGA Genomic Data Commons ATACseq-AWG release
- TCGA gene expression matrix (RNA-seq, used for the >=50% expression filter)
- MEME-curated cis-BP motif reference for FIMO scanning

## Outputs

- tf_target_matrix.pkl: binary 320 x 5541 {0,1} TF-target matrix (320 TF motifs x 5541 filtered genes) constraining CITRUS decoder connections

## How to apply

Download the pan-cancer ATAC-seq open-chromatin peak atlas (~200K peaks) from the TCGA Genomic Data Commons ATACseq-AWG release together with the TCGA gene expression matrix. Scan all peak sequences with FIMO against the MEME-curated cis-BP motif reference at significance threshold P < 1e-5 to link TF motifs to peaks. Retain only TFs whose genes are expressed in at least 50% of samples in at least one of the 17 TCGA tumor types, and deduplicate near-identical motifs by removing TF motif pairs with Jaccard similarity > 0.5, yielding the final 320 TF motifs. Assign each motif-bearing peak to its nearest gene using the ChIPpeakAnno nearest-distance rule and binarize the assignments into a {0,1} TF-target matrix restricted to the 5541 filtered genes, packaging the 320 x 5541 matrix as tf_target_matrix.pkl.

## Related tools

- **CITRUS** (Downstream model whose decoder's TF-to-gene layer is masked by the binary prior produced here) — https://github.com/osmanbeyoglulab/CITRUS
- **FIMO** (Scans ATAC-seq peak sequences against the MEME-curated cis-BP motif reference at P < 1e-5 to link TF motifs to open-chromatin peaks)
- **ChIPpeakAnno** (Assigns each motif-bearing peak to its nearest gene via the nearest-distance rule before binarization)
- **Pandas** (Data handling for the expression matrix, peak assignments, and packaging the prior into a pickle file)
- **Numpy** (Array operations for binarizing TF-target assignments and validating matrix contents)

## Evaluation signals

- Verify tf_target_matrix.pkl contains only {0,1} entries (no weights or directionality values)
- Check matrix dimensions are exactly 320 TF motifs x 5541 genes
- Confirm every retained TF's gene is expressed in >= 50% of samples in at least one of the 17 TCGA tumor types
- Confirm no retained TF motif pairs have Jaccard similarity > 0.5 (deduplication applied)
- Check the 5541-gene axis matches the GEP's filtered gene set (top variant genes per cancer type plus gene availability in the TF-target matrix)

## Limitations

- TFs of the same family often share a similar motif and cannot be disambiguated, so each motif may encompass the activities of multiple TFs
- No directionality (activation vs. repression) is represented in the TF–target gene priors
- Co-binding TF binding patterns (e.g. AP-1–IRF complexes) are not represented in the model
- Tumor regulatory network analysis is confounded by stromal/immune cells and cancer cell heterogeneity

## Evidence

- [other] Assign each motif-bearing peak to its nearest gene with the ChIPpeakAnno nearest-distance rule and binarize the assignments into a {0,1} TF-target matrix restricted to the 5541 filtered genes.: [evidence span withheld — anchor status: not_found]
- [readme] GEP contains continuous values of gene expression with filtered 5541 genes by considering top variant genes in each cancer type as well as gene availability in TF-target gene matrix.: "GEP contains continuous values of gene expression with filtered 5541 genes by considering top variant genes in each cancer type as well as gene availability in TF-target gene matrix."
- [discussion] For a more accurate representation of TF-target gene relationships, we leveraged ATAC-seq tumor data from TCGA patients.: "For a more accurate representation of TF-target gene relationships, we leveraged ATAC-seq tumor data from TCGA patients."
- [discussion] TFs of the same family often share a similar motif and thus are difficult to disambiguate. Therefore, TF motifs may encompass the activities of multiple TFs.: "TFs of the same family often share a similar motif and thus are difficult to disambiguate. Therefore, TF motifs may encompass the activities of multiple TFs."
