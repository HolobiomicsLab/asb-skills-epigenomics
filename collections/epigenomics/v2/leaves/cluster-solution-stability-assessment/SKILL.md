---
name: cluster-solution-stability-assessment
description: Use when running SRIQ (or similar permutation-based clustering) and you
  need to determine which clustering solution (K and diameter) is stable enough to
  trust, based on recurrence across iterations, QSS, and silhouette scores.
when_to_use_negative:
- When the number of clusters K must be user-predefined, as in consensus/bootstrap
  clustering — SRIQ evaluates solution stability on its own without user input for
  K
- When the dataset is too small to detect small or underrepresented biological subgroups
  — SRIQ cannot circumvent this general limitation
- When the goal is detecting subtle subtypes regardless of stability, since solutions
  are sensitive to the diameter parameter and must be validated against recurrence
  across iterations
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: SRIQ
  role: Runs the QT/random-forest clustering with permutation-based stability testing;
    the Clusters_Frequencies.pdf output is used to assess recurrence and select stable
    solutions
  repo: https://github.com/StaafLab/SRIQ
- name: Java
  role: Execution environment for SRIQS.jar (e.g. 'java -jar -Xmx8g SRIQS.jar VRLA
    path-to/test.properties') to run clustering and stability evaluation
  repo: https://github.com/StaafLab/SRIQ
- name: SRIQPreprocess.jar
  role: Generates the median-centered log2 non-zero-filtered input matrix (e.g. *_mc_log2_nz)
    required before stability assessment can be performed
  repo: https://github.com/StaafLab/SRIQ
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:bc73c1111a57c611af93c67c1469472459d6205272a4fa221c00a3c7f2e869b5
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cluster-solution-stability-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess the stability and recurrence of unsupervised clustering solutions produced by SRIQ across repeated iterations, using recurrence frequency, QSS (quality/stability score), silhouette values, and diameter cut-off scanning to select robust cluster solutions such as the K6 solution at diameter 0.63 on 434 TCGA-LUAD cases.

## When to use

Use when running SRIQ (or similar permutation-based clustering) and you need to determine which clustering solution (K and diameter) is stable enough to trust, based on recurrence across iterations, QSS, and silhouette scores.

## When NOT to use

- When the number of clusters K must be user-predefined, as in consensus/bootstrap clustering — SRIQ evaluates solution stability on its own without user input for K
- When the dataset is too small to detect small or underrepresented biological subgroups — SRIQ cannot circumvent this general limitation
- When the goal is detecting subtle subtypes regardless of stability, since solutions are sensitive to the diameter parameter and must be validated against recurrence across iterations

## Inputs

- SRIQ-preprocessed expression matrix (tab-separated .txt, offset, log2-transformed, median-centered, non-expressed filtered; e.g. GeneData_AC_fpkm_mc_log2_nz) for 434 TCGA-LUAD cases with 25,785 transcripts
- test.properties file configured with minBagSize=1200, permutations=10000, iterations=10, method=PEARSON, spiral=TRUE, minClusterSize=0, and distCutOff values 0.8 down to 0.52
- ..._Clusters_Frequencies.pdf listing cluster solution occurrence across iterations

## Outputs

- Candidate clustering solutions with recurrence counts across the 10 iterations (e.g. K6 solution at diameter cut-off 0.63 with 100% recurrence and QSS of 60%)
- Silhouette values for each candidate solution
- Sample classification into core, spiral, and unstable groups (e.g. 382 core/spiral samples and 52 unstable samples out of 434)
- Results_log_0.63_6 output directory with the selected stable solution

## How to apply

Run SRIQ on the preprocessed (offset, log2-transformed, median-centered, non-expressed-filtered, e.g. GeneData_AC_fpkm_mc_log2_nz) expression matrix with minBagSize=1200, permutations=10000, iterations=10, method=PEARSON, spiral=TRUE, minClusterSize=0, and distCutOff scanned from 0.8 down to 0.52 via 'java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties'. Inspect the generated ..._Clusters_Frequencies.pdf to identify candidate solutions by their occurrence across the 10 iterations. Compile recurrence, QSS, and silhouette values for each candidate solution; a solution like K6 at diameter 0.63 showing 100% recurrence across all 10 iterations with a QSS of 60% is classified as highly stable. Confirm the final solution is consistent with the sample classification (e.g. 382 core/spiral and 52 unstable samples out of 434), and note that the diameter must be considered in the context of a solution's stability and occurrence across iterations.

## Related tools

- **SRIQ** (Runs the QT/random-forest clustering with permutation-based stability testing; the Clusters_Frequencies.pdf output is used to assess recurrence and select stable solutions) — https://github.com/StaafLab/SRIQ
- **Java** (Execution environment for SRIQS.jar (e.g. 'java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties') to run clustering and stability evaluation) — https://github.com/StaafLab/SRIQ
- **SRIQPreprocess.jar** (Generates the median-centered log2 non-zero-filtered input matrix (e.g. *_mc_log2_nz) required before stability assessment can be performed) — https://github.com/StaafLab/SRIQ

## Examples

```
java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties
```

## Evaluation signals

- The K6 solution at diameter cut-off 0.63 (Results_log_0.63_6) shows 100% recurrence across all 10 iterations with a QSS of 60%
- Silhouette values are reported for each candidate solution and the selected solution is among those in ..._Clusters_Frequencies.pdf
- Final classification yields 382 core or spiral samples and 52 unstable samples out of 434 TCGA-LUAD cases
- Output directory follows the pattern ...\10000\QC_Spiral(false)\Results_log_0.63_6\, consistent with diameter 0.63 and K=6 parameters
- Core cluster samples are clearly separated from each other when assessed by UMAP analysis

## Limitations

- The diameter needs to be considered in the context of the stability and occurrence of a solution across iterations; solutions are sensitive to this parameter
- The combination of large bag size (1200), high permutation count (10,000), and large datasets (samples and features) makes SRIQ computation and memory intensive
- SRIQ cannot circumvent the general problem of smaller datasets limiting the possibility to detect small or underrepresented biological subgroups (clusters)
- Additional metrics for cluster stability to facilitate cluster solution selection are not yet implemented in the pipeline

## Evidence

- [results] K6 solution recurrence and QSS at diameter 0.63: [evidence span withheld — anchor status: not_found]
