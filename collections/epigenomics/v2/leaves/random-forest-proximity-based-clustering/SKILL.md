---
name: random-forest-proximity-based-clustering
description: Use when clustering high-dimensional tumor gene expression data (e.g.,
  RNA-seq FPKM matrices) into stable, reproducible subgroups without predefining K,
  and when consensus clustering overfits due to the curse of dimensionality.
when_to_use_negative:
- Input is single-cell sequencing or epigenomic data — the pipeline is not yet applied
  to these data types
- The dataset is small and the goal is detecting small or underrepresented biological
  subgroups — SRIQ cannot circumvent this limitation
- The analysis requires distance metrics beyond Pearson correlation or Euclidean distance,
  which are the only implemented options
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SRIQ (SRIQS.jar)
  role: Java tool that performs the Random Forest/QT/KNN clustering, stability evaluation,
    cluster extraction (EXTRACT), and differential expression (SAMDEG)
  repo: https://github.com/StaafLab/SRIQ
- name: SRIQPreprocess.jar
  role: Pre-processes the raw FPKM matrix (offset, log2-transformation, median-centering,
    non-expressed filtering) into the *_mc_log2_nz input format required by SRIQ
  repo: https://github.com/StaafLab/SRIQ
- name: Java
  role: Runtime environment for SRIQS.jar and SRIQPreprocess.jar; memory allocation
    (e.g., -Xmx8g) is set at launch because large bags/permutations are memory intensive
  repo: https://github.com/StaafLab/SRIQ
- name: Python/Jupyter pipeline
  role: Alternative pipeline entry point (analysis_pipeline notebook) wrapping SRIQ-clustering,
    silhouette-plot analysis, UMAP, SAM differential expression, and EnrichR enrichment
  repo: https://github.com/StaafLab/SRIQ
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:ffa0e059fd0f676b75461db3980eaabc10a8956c5f148e72c5856ade110cdbb1
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
  - Python/Jupyter pipeline
  - SRIQ (SRIQS.jar)
  - SRIQPreprocess.jar
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# random-forest-proximity-based-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

SRIQ clustering fuses Random Forest, quality threshold (QT) clustering, and KNN concepts to find core clusters of highly correlated observations and spirals out to build larger clusters, self-evaluating solution stability without a predefined K. Applied to 434 TCGA-LUAD cases with 25,785 transcripts, it identifies stable transcriptional subgroups with recurrence, QSS, and silhouette metrics.

## When to use

Use when clustering high-dimensional tumor gene expression data (e.g., RNA-seq FPKM matrices) into stable, reproducible subgroups without predefining K, and when consensus clustering overfits due to the curse of dimensionality.

## When NOT to use

- Input is single-cell sequencing or epigenomic data — the pipeline is not yet applied to these data types
- The dataset is small and the goal is detecting small or underrepresented biological subgroups — SRIQ cannot circumvent this limitation
- The analysis requires distance metrics beyond Pearson correlation or Euclidean distance, which are the only implemented options

## Inputs

- Tab-separated .txt gene-by-sample expression matrix (e.g., TCGA-LUAD FPKM, 434 cases x 25,785 transcripts with Ensembl gene IDs)
- test.properties configuration file (studyName, studyPath, inFileName, outPath, distCutOff list, permutations, iterations, spiral, minClusterSize, minBagSize, method)

## Outputs

- Preprocessed *_mc_log2_nz median-centered log2 expression file from SRIQPreprocess.jar
- Clusters_Frequencies.pdf showing candidate cluster solutions and their occurrence across iterations
- Cluster solution folders (e.g., Results_log_0.63_6) with cluster assignments, recurrence, QSS, and silhouette values
- Classification of samples as core, spiral, or unstable (e.g., 382 core/spiral, 52 unstable of 434)

## How to apply

Preprocess the FPKM expression matrix with SRIQPreprocess.jar (offset by 0.1 or set values <1 to 1, log2-transform, median-center, remove non-expressed/low-variation transcripts), producing a file named *_mc_log2_nz. Configure test.properties with studyName, studyPath, inFileName, outPath, and scan distCutOff from 0.8 down to 0.52 with minBagSize=1200, permutations=10000, iterations=10, method=PEARSON, spiral=TRUE. Run SRIQ via 'java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties' and inspect ..._Clusters_Frequencies.pdf to identify candidate solutions by their recurrence across the 10 iterations. Select solutions balancing diameter, stability (QSS), and occurrence — e.g., the K6 solution at diameter cut-off 0.63 showed 100% recurrence with QSS 60% and classified 382/434 samples as core or spiral and 52 as unstable. Rationale: requiring a predefined K (as in consensus/bootstrap clustering) limits subtype discovery, and SRIQ evaluates solution stability on its own.

## Related tools

- **SRIQ (SRIQS.jar)** (Java tool that performs the Random Forest/QT/KNN clustering, stability evaluation, cluster extraction (EXTRACT), and differential expression (SAMDEG)) — https://github.com/StaafLab/SRIQ
- **SRIQPreprocess.jar** (Pre-processes the raw FPKM matrix (offset, log2-transformation, median-centering, non-expressed filtering) into the *_mc_log2_nz input format required by SRIQ) — https://github.com/StaafLab/SRIQ
- **Java** (Runtime environment for SRIQS.jar and SRIQPreprocess.jar; memory allocation (e.g., -Xmx8g) is set at launch because large bags/permutations are memory intensive) — https://github.com/StaafLab/SRIQ
- **Python/Jupyter pipeline** (Alternative pipeline entry point (analysis_pipeline notebook) wrapping SRIQ-clustering, silhouette-plot analysis, UMAP, SAM differential expression, and EnrichR enrichment) — https://github.com/StaafLab/SRIQ

## Examples

```
java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties
```

## Evaluation signals

- SRIQPreprocess.jar output file ends with '_mc_log2_nz' and expression data are offset by 0.1 (or values <1 set to 1), log2-transformed, and median-centered
- Candidate solutions are identified from ..._Clusters_Frequencies.pdf by their recurrence across the 10 iterations (e.g., K6 at diameter 0.63 showed 100% recurrence)
- Selected solution reports QSS and silhouette values (e.g., QSS of 60% for the K6/diameter-0.63 solution)
- Sample classification counts match expectations: 382 core/spiral and 52 unstable samples out of 434
- Extraction of a chosen solution creates the expected output folder (e.g., .../QC_Spiral(false)/Results_log_0.63_6/)

## Limitations

- The combination of a large bag size, high number of permutations, and large datasets (samples and features) makes SRIQ computationally and memory intensive
- Cluster solutions are sensitive to the cluster diameter parameter, which must be evaluated against stability and occurrence across iterations
- SRIQ cannot circumvent the general problem of smaller datasets limiting the possibility to detect small or underrepresented biological subgroups (clusters)

## Evidence
