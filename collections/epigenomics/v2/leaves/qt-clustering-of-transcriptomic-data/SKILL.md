---
name: qt-clustering-of-transcriptomic-data
description: Use when clustering a gene-by-sample transcriptomic matrix (e.g., FPKM
  RNA-seq) to discover stable tumor subgroups without predefining the number of clusters
  K, and when consensus/bootstrap clustering stability is in question.
when_to_use_negative:
- The number of clusters K is fixed in advance and the task is only to assign samples
  — consensus clustering with predefined K is the comparison method here, not SRIQ's
  purpose
- The dataset is small and the goal is detecting small or underrepresented biological
  subgroups — the authors state SRIQ cannot circumvent this limitation
- A distance metric other than Pearson correlation or Euclidean distance is required
  — additional metrics are not yet implemented
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SRIQ
  role: Java/Python clustering pipeline that fuses Random Forest, QT clustering, and
    KNN to find core clusters of highly correlated observations and spiral outward
    into larger clusters with self-evaluated stability
  repo: https://github.com/StaafLab/SRIQ
- name: SRIQPreprocess.jar
  role: Preprocesses the raw FPKM file into the offset, log2-transformed, median-centered,
    non-expressed-filtered input required before clustering
  repo: https://github.com/StaafLab/SRIQ
- name: Java
  role: Runtime for SRIQS.jar (VRLA clustering, EXTRACT cluster data, SAMDEG differential
    expression) launched with -Xmx8g for memory-intensive runs
- name: UMAP
  role: Used to visually confirm that core cluster samples of the selected solution
    are clearly separated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:e3ff06045d0d19d5e9ffd951eb82d209bd40c4ee371d0133ea124cafb23d9751
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
  - SRIQ
  - SRIQPreprocess.jar
  - UMAP
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# qt-clustering-of-transcriptomic-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply SRIQ, a systematic random forest integration to quality threshold (QT) clustering that fuses Random Forest, QT clustering, and KNN concepts, to discover stable unsupervised cluster solutions in tumor gene expression data without predefining K, as demonstrated on 434 TCGA-LUAD cases with 25,785 transcripts.

## When to use

Use when clustering a gene-by-sample transcriptomic matrix (e.g., FPKM RNA-seq) to discover stable tumor subgroups without predefining the number of clusters K, and when consensus/bootstrap clustering stability is in question.

## When NOT to use

- The number of clusters K is fixed in advance and the task is only to assign samples — consensus clustering with predefined K is the comparison method here, not SRIQ's purpose
- The dataset is small and the goal is detecting small or underrepresented biological subgroups — the authors state SRIQ cannot circumvent this limitation
- A distance metric other than Pearson correlation or Euclidean distance is required — additional metrics are not yet implemented

## Inputs

- Tab-separated gene-by-sample FPKM expression matrix (.txt) from 434 TCGA-LUAD cases
- Preprocessed offset, log2-transformed, median-centered, non-expressed-filtered matrix (e.g., GeneData_AC_fpkm_mc_log2_nz)
- Configured test.properties file (studyName, studyPath, inFileName, outPath, distCutOff scan, permutations=10000, iterations=10, minBagSize=1200, method=PEARSON, spiral=TRUE)

## Outputs

- ..._Clusters_Frequencies.pdf listing candidate cluster solutions by occurrence across 10 iterations
- Results_log_0.63_6 output folder with cluster assignments (382 core/spiral and 52 unstable samples)
- Recurrence, QSS stability, and silhouette values for candidate solutions (e.g., K6 at diameter 0.63: 100% recurrence, QSS 60%)

## How to apply

Preprocess the FPKM matrix with SRIQPreprocess.jar (off-set, log2-transform, median-center, filter non-expressed and low-variation transcripts) to produce a file such as GeneData_AC_fpkm_mc_log2_nz. Configure test.properties with minBagSize=1200, permutations=10000, iterations=10, method=PEARSON, spiral=TRUE, minClusterSize=0, and scan distCutOff from 0.8 down to 0.52. Launch clustering with 'java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties', then inspect the generated ..._Clusters_Frequencies.pdf to identify candidate solutions by their recurrence across the 10 iterations. Extract the chosen solution (e.g., K6 at diameter 0.63) with the EXTRACT command into a Results_log_0.63_6 folder, and optionally run SAMDEG for differential expression. Judge success by recurrence/stability: e.g., the K6 solution at diameter cut-off 0.63 showed 100% recurrence across all 10 iterations with QSS of 60%, and 382 of 434 samples classified as core/spiral with 52 unstable.

## Related tools

- **SRIQ** (Java/Python clustering pipeline that fuses Random Forest, QT clustering, and KNN to find core clusters of highly correlated observations and spiral outward into larger clusters with self-evaluated stability) — https://github.com/StaafLab/SRIQ
- **SRIQPreprocess.jar** (Preprocesses the raw FPKM file into the offset, log2-transformed, median-centered, non-expressed-filtered input required before clustering) — https://github.com/StaafLab/SRIQ
- **Java** (Runtime for SRIQS.jar (VRLA clustering, EXTRACT cluster data, SAMDEG differential expression) launched with -Xmx8g for memory-intensive runs)
- **UMAP** (Used to visually confirm that core cluster samples of the selected solution are clearly separated)

## Examples

```
java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties
```

## Evaluation signals

- The preprocessed input file exists and ends with '_mc_log2_nz', with >25000 transcripts retained after removing non-expressed and low-variation transcripts
- The ..._Clusters_Frequencies.pdf is generated and candidate solutions can be ranked by occurrence across the 10 iterations
- The target solution appears at the expected diameter with matching recurrence: K6 at diameter cut-off 0.63 with 100% recurrence and QSS of 60%
- Sample classification totals match: 382 of 434 samples classified as core or spiral, 52 considered unstable
- The extracted solution lands in a folder named Results_log_0.63_6 under the QC output path

## Limitations

- The combination of large bag size, high permutation count, and large datasets (samples and features) makes SRIQ computation and memory intensive
- Cluster solutions are sensitive to the diameter parameter, which must be evaluated against stability and occurrence across iterations
- Only Pearson and Euclidean distance metrics are implemented; additional metrics are not yet available
- SRIQ cannot circumvent the general problem of small datasets limiting detection of small or underrepresented biological subgroups
- Not yet applied to single cell sequencing or epigenomic data; more optimized computing strategies would be needed

## Evidence
