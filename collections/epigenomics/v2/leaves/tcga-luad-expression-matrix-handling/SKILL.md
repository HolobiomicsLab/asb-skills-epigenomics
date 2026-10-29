---
name: tcga-luad-expression-matrix-handling
description: Use when raw TCGA-LUAD FPKM expression data (samples as columns, genes
  as rows) must be converted into SRIQ's required log2/median-centered format, or
  when per-sample TRU/PI/PP subtype labels must be derived from Wilkerson centroid
  genes.
when_to_use_negative:
- Input is single-cell sequencing or epigenomic data — the paper states the pipeline
  is not yet applied to such data types
- The expression matrix is already off-set, log2-transformed, and median-centered
  '_mc_log2_nz' output — re-running SRIQPreprocess.jar would double-transform the
  data
- The dataset is small, since SRIQ cannot circumvent limited detection of small or
  underrepresented biological subgroups in smaller datasets
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SRIQPreprocess.jar
  role: Java tool that off-sets (0.1 or values <1 set to 1), log2-transforms, and
    median-centers the FPKM matrix and removes non-expressed/low-variation transcripts,
    producing the '_mc_log2_nz' input for SRIQ
  repo: https://github.com/StaafLab/SRIQ
- name: pandas
  role: Python library used to load the '_mc_log2_nz' expression matrix and restrict
    it to Wilkerson centroid genes for Pearson-correlation subtype assignment
- name: SRIQ
  role: Java/Python clustering pipeline whose input format requirements (tab-separated
    .txt, off-set, log2, median-centered) dictate the matrix handling; cluster solutions
    feed the contingency table
  repo: https://github.com/StaafLab/SRIQ
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:55ef528230add214cd0c691d038826cc2d35a11ec6c161117e5939749f9060f0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/StaafLab/SRIQ
  repo_url: https://github.com/StaafLab/SRIQ
  tools:
  - Java
  - Python
  - SRIQ
  - SRIQPreprocess.jar
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# tcga-luad-expression-matrix-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Handles TCGA-LUAD gene expression matrices for SRIQ clustering and downstream Wilkerson TRU/PI/PP centroid subtyping: preprocessing 434 RNA-seq samples into a log2-transformed, median-centered '_mc_log2_nz' tab-separated matrix and restricting it to subtype centroid genes for Pearson-correlation classification.

## When to use

Use when raw TCGA-LUAD FPKM expression data (samples as columns, genes as rows) must be converted into SRIQ's required log2/median-centered format, or when per-sample TRU/PI/PP subtype labels must be derived from Wilkerson centroid genes.

## When NOT to use

- Input is single-cell sequencing or epigenomic data — the paper states the pipeline is not yet applied to such data types
- The expression matrix is already off-set, log2-transformed, and median-centered '_mc_log2_nz' output — re-running SRIQPreprocess.jar would double-transform the data
- The dataset is small, since SRIQ cannot circumvent limited detection of small or underrepresented biological subgroups in smaller datasets

## Inputs

- Tab-separated .txt FPKM expression matrix with a Gene column and TCGA-LUAD sample barcode columns (e.g. GeneData_AC_fpkm.txt)
- Log2-transformed, median-centered '_mc_log2_nz' output file from SRIQPreprocess.jar
- Wilkerson et al. TRU/PI/PP subtype centroid gene lists
- SRIQ cluster membership assignments (e.g. 6-cluster solution at diameter 0.63, spiral=false)

## Outputs

- '_mc_log2_nz' log2-transformed, median-centered expression matrix (>25,000 transcripts x 434 samples)
- Per-sample TRU/PI/PP subtype labels for all 434 LUAD samples
- Cluster-by-subtype contingency table (e.g. subtype_sriq_crosstab.csv)

## How to apply

Run SRIQPreprocess.jar on the tab-separated FPKM file (Gene column plus TCGA sample barcode columns, e.g. TCGA-05-4384-01A), which off-sets values, log2-transforms, median-centers, and removes non-expressed/low-variation transcripts while retaining >25,000 transcripts, producing a '_mc_log2_nz' output file. Load this matrix with pandas in Python for downstream steps. For centroid classification, restrict the matrix to the Wilkerson et al. terminal respiratory unit (TRU), proximal-inflammatory (PI), and proximal-proliferative (PP) centroid genes, compute Pearson correlations between each of the 434 samples and each subtype centroid, and assign each sample to the highest-correlating subtype. Cross-tabulate these labels against an SRIQ cluster solution (e.g. the 6-cluster solution at diameter 0.63 from a 10,000-permutation, BagSize-1200 run) flagging unstable (non-core/non-spiral) samples separately.

## Related tools

- **SRIQPreprocess.jar** (Java tool that off-sets (0.1 or values <1 set to 1), log2-transforms, and median-centers the FPKM matrix and removes non-expressed/low-variation transcripts, producing the '_mc_log2_nz' input for SRIQ) — https://github.com/StaafLab/SRIQ
- **pandas** (Python library used to load the '_mc_log2_nz' expression matrix and restrict it to Wilkerson centroid genes for Pearson-correlation subtype assignment)
- **SRIQ** (Java/Python clustering pipeline whose input format requirements (tab-separated .txt, off-set, log2, median-centered) dictate the matrix handling; cluster solutions feed the contingency table) — https://github.com/StaafLab/SRIQ

## Examples

```
java -jar SRIQPreprocess.jar path-to/datafile.txt
```

## Evaluation signals

- SRIQPreprocess.jar output file name ends with '_mc_log2_nz' and resides in the same folder as the input file
- Preprocessed matrix values are log2-transformed and median-centered (centered around 0), with filtered genes set to 0
- More than 25,000 transcripts are retained after removal of non-expressed and very-low-variation transcripts
- The matrix is a tab-separated .txt file with a Gene column and 434 TCGA-LUAD sample columns
- The subtype-vs-SRIQ crosstab shows SRIQ subclusters partitioning preferentially within TRU and PP samples (e.g. cluster 5 ~100% TRU core; PP split across core clusters 2, 4, and 6) while PI remains comparatively homogeneous

## Limitations

- Data must be in SRIQ's exact format: tab-separated .txt, off-set by 0.1 (or values <1 set to 1), log2-transformed, and median-centered — other formats will not be accepted
- Only Pearson and Euclidean distance metrics are implemented; additional distance metrics are not yet available
- Handling large datasets with large bag size and high permutation counts is computation- and memory-intensive

## Evidence

- [readme] For clustering, expression data should be:
1. Off-set by 0.1 OR set all values < 1 set to 1
2. Log2transformed
3. Median-centered: "1. Off-set by 0.1 OR set all values < 1 set to 1
2. Log2transformed
3. Median-centered"
