---
name: epigenetic-feature-matrix-encoding
description: Use when preparing sgRNA training/prediction inputs for a CNN that must
  combine target-site sequence with cell-type epigenetic signals (nucleosome, accessibility,
  methylation, expression) around CRISPRoff/CRISPRi/CRISPRa target sites.
when_to_use_negative:
- Prediction runs requiring sequence-only input should use the 4-channel sequence
  encoding only (e.g. CRISPRoff_seq model), not the full epigenetic matrix
- Cell lines or epigenetic marks not covered by the available epi_reference tracks
  — incomplete epigenetic feature information limits model performance
- Analyses targeting nucleases other than dCas-based epigenome editing systems (CRISPRoff/CRISPRi/CRISPRa),
  where the epigenetic channel semantics do not apply
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: EpiCas-DL
  role: Deep learning framework whose CNN consumes the 11x40 encoded feature matrices
    as model input
  repo: https://github.com/yangqianq/EpiCas-DL
- name: pybedtools
  role: Custom Python scripts using pybedtools to extract epigenetic signal at sgRNA
    genomic coordinates
  repo: https://github.com/yangqianq/EpiCas-DL
- name: ATACdb
  role: Source of genome-wide ATAC-seq chromatin accessibility tracks used as an encoding
    channel
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:b4dbf19fae7c3f64b658081e4fc31ee9c3c91cba7914e5ba6f8f2af1958d491c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - ATACdb
  - EpiCas-DL
  - GPyOpt
  - pybedtools
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# epigenetic-feature-matrix-encoding

## Summary

Encodes sgRNA target sequences and epigenetic context into an 11x40 image-like feature matrix (4 one-hot sequence channels plus nucleosome positioning, chromatin accessibility, DNA methylation, and gene expression channels) for input to the EpiCas-DL CNN predicting sgRNA activity in CRISPR-mediated epigenome editing.

## When to use

Use when preparing sgRNA training/prediction inputs for a CNN that must combine target-site sequence with cell-type epigenetic signals (nucleosome, accessibility, methylation, expression) around CRISPRoff/CRISPRi/CRISPRa target sites.

## When NOT to use

- Prediction runs requiring sequence-only input should use the 4-channel sequence encoding only (e.g. CRISPRoff_seq model), not the full epigenetic matrix
- Cell lines or epigenetic marks not covered by the available epi_reference tracks — incomplete epigenetic feature information limits model performance
- Analyses targeting nucleases other than dCas-based epigenome editing systems (CRISPRoff/CRISPRi/CRISPRa), where the epigenetic channel semantics do not apply

## Inputs

- sgRNA target sequences (40-nt target window) with chromosome, start/end coordinates, PAM position
- Epigenetic reference tracks: nucleosome positioning, ATAC-seq chromatin accessibility (ATACdb), DNA methylation, gene expression, per cell line (e.g. Hek293t, K562)
- Gene/transcript annotations (TSS, transcript e.g. P1P2, gene name)
- Screening datasets with phenotype scores (e.g. CRISPRoff_tiling: 111,682 sgRNAs, 520 genes)

## Outputs

- 11x40 feature matrices per sgRNA (4 one-hot sequence channels + 7 min-max normalized epigenetic channels)
- Gene-level normalized sgRNA activity scores as regression targets

## How to apply

Formulate each sgRNA sequence with one-hot encoding into an image-like coding scheme of 4 nucleotide channels over the 40-nt target window. Add 7 epigenetic channels per position — nucleosome positioning, chromatin accessibility (ATAC-seq from ATACdb), DNA methylation, and gene expression — min-max normalizing every epigenetic feature before model input, yielding an 11x40 matrix. For CRISPRoff_tiling this covers 111,682 sgRNAs within ±2.5 kb of the TSS of 520 genes with gene-level normalized activity scores (normalized by the absolute average score of the top 3 sgRNAs per gene). Locate each sgRNA's genomic coordinates, chromosome, transcript, gene, and cell line (e.g. Hek293t, K562) to pull matching epigenetic tracks via the repository's epi_reference files and pybedtools scripts. The rationale is that epigenetic state (e.g. methylated target sites show higher repression efficiency) improves sgRNA activity prediction beyond sequence alone.

## Related tools

- **EpiCas-DL** (Deep learning framework whose CNN consumes the 11x40 encoded feature matrices as model input) — https://github.com/yangqianq/EpiCas-DL
- **pybedtools** (Custom Python scripts using pybedtools to extract epigenetic signal at sgRNA genomic coordinates) — https://github.com/yangqianq/EpiCas-DL
- **ATACdb** (Source of genome-wide ATAC-seq chromatin accessibility tracks used as an encoding channel)

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- Every encoded matrix has shape 11x40: 4 one-hot sequence channels with exactly one hot nucleotide per position, plus 7 epigenetic channels
- All epigenetic channel values fall in [0,1] after min-max normalization
- Encoded sgRNA coordinates match their stated chromosome/start/end/PAM positions and gene/transcript annotations
- Activity-score targets reproduce gene-level normalization: each sgRNA's phenotype score divided by the absolute average score of the top 3 sgRNAs for that gene
- Downstream EpiCas-DL training on the encoded matrices reproduces published Spearman correlation, MSE, and cosine similarity performance on CRISPRoff_tiling consistent with Fig. 2b-2c

## Limitations

- Only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression are encoded; incomplete epigenetic feature information limited model performance
- Available CRISPRoff screening datasets are relatively small and mainly in HEK293T and K562 cell lines, increasing overfitting risk and limiting generalization
- Epigenetic reference tracks must be fetched externally (epi_reference folder hosted on pan.baidu.com), which may hinder reproducibility

## Evidence

- [methods] One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme: "One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme"
- [methods] All the epigenetic features were min–max normalized before model input: "All the epigenetic features were min–max normalized before model input"
- [methods] CRISPRoff_tiling dataset contains 111,682 sgRNAs within ± 2.5 kb from the TSS of 520 genes: "CRISPRoff_tiling dataset contains 111,682 sgRNAs within ± 2.5 kb from the TSS of 520 genes"
- [methods] normalizing the phenotype score of each sgRNA with the absolute average score of the top 3 sgRNAs for each gene: "normalizing the phenotype score of each sgRNA with the absolute average score of the top 3 sgRNAs for each gene"
- [readme] The tool combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site: "The tool combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site"
