---
name: bagsize-parameter-perturbation
description: Use when validating the stability of SRIQ cluster solutions on TCGA LUAD
  RNA-seq data by perturbing the BagSize gene-sampling parameter and quantifying how
  cluster assignments co-occur across runs.
when_to_use_negative:
- When the goal is selecting K or cluster diameter (distCutOff) rather than testing
  parameter sensitivity — SRIQ evaluates solution stability on its own and the diameter
  must instead be judged against solution stability and occurrence across iterations
- 'When the dataset is small: SRIQ cannot circumvent the limitation that smaller datasets
  fail to detect small or underrepresented biological subgroups, so stability results
  may not generalize'
- When applying a different clustering method (e.g. consensus clustering with ward's
  linkage and 70% resampling) — this perturbation protocol is specific to SRIQ's random-forest
  BagSize parameter
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SRIQ
  role: Clustering engine whose BagSize parameter is perturbed; run via SRIQS.jar
    in VRLA mode for each reduced BagSize value and EXTRACT mode to pull cluster assignments
  repo: https://github.com/StaafLab/SRIQ
- name: Java
  role: Runtime environment for SRIQS.jar (e.g. java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties)
    with large heap to handle memory-intensive runs
  repo: https://github.com/StaafLab/SRIQ
- name: Python
  role: Part of the SRIQ pipeline (analysis_pipeline notebook) used to tabulate and
    compare co-occurrence similarity across perturbed runs
  repo: https://github.com/StaafLab/SRIQ
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:8915feea143a40124aa3d7ad9d6bf12948ab69df180e33b67c53270ed2743b0f
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# bagsize-parameter-perturbation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A technical reproducibility test for SRIQ clustering in which the BagSize parameter (number of randomly sampled genes per permutation) is stepwise reduced from 1100 down to 300, and the resulting K3, K5, and K6 cluster solutions are compared for pairwise co-occurrence similarity against the original BagSize-1200 run.

## When to use

Use when validating the stability of SRIQ cluster solutions on TCGA LUAD RNA-seq data by perturbing the BagSize gene-sampling parameter and quantifying how cluster assignments co-occur across runs.

## When NOT to use

- When the goal is selecting K or cluster diameter (distCutOff) rather than testing parameter sensitivity — SRIQ evaluates solution stability on its own and the diameter must instead be judged against solution stability and occurrence across iterations
- When the dataset is small: SRIQ cannot circumvent the limitation that smaller datasets fail to detect small or underrepresented biological subgroups, so stability results may not generalize
- When applying a different clustering method (e.g. consensus clustering with ward's linkage and 70% resampling) — this perturbation protocol is specific to SRIQ's random-forest BagSize parameter

## Inputs

- Preprocessed log2-transformed, median-centered gene expression matrix (tab-separated .txt, e.g. GeneData_AC_fpkm_mc_log2_nz from SRIQPreprocess.jar)
- test.properties configuration file (studyName, studyPath, inFileName, outPath, distCutOff, permutations=10000, iterations=10, spiral=TRUE, minClusterSize=0, minBagSize, method=PEARSON)

## Outputs

- SRIQ cluster solutions (K3, K5, K6) for each perturbed BagSize level (1100–300)
- Co-occurrence similarity table per perturbation type and level (e.g. sriq_stability_similarity.csv)
- Stability verdict: pass if K3, K5, K6 retain >80% pairwise similarity to the BagSize-1200 reference run

## How to apply

Load the preprocessed log2-transformed, median-centered LUAD expression matrix (e.g. GeneData_AC_fpkm_mc_log2_nz) with a test.properties configuration holding permutations=10000, iterations=10, method=PEARSON, and minBagSize=1200 as the reference. Run SRIQ via 'java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties' in new runs where minBagSize is stepwise reduced from 1100 down to 300 while keeping permutations, iterations, and distance method intact. Optionally run a parallel stability series in which an increasing fraction of samples is randomly removed. For each perturbed run, extract the K3, K5, and K6 cluster solutions and compute pairwise co-occurrence similarity of sample assignments against the original BagSize-1200 run. Tabulate similarity per perturbation level (e.g. sriq_stability_similarity.csv) and accept the stability result if all solutions retain >80% similarity, with K3 near 100%.

## Related tools

- **SRIQ** (Clustering engine whose BagSize parameter is perturbed; run via SRIQS.jar in VRLA mode for each reduced BagSize value and EXTRACT mode to pull cluster assignments) — https://github.com/StaafLab/SRIQ
- **Java** (Runtime environment for SRIQS.jar (e.g. java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties) with large heap to handle memory-intensive runs) — https://github.com/StaafLab/SRIQ
- **Python** (Part of the SRIQ pipeline (analysis_pipeline notebook) used to tabulate and compare co-occurrence similarity across perturbed runs) — https://github.com/StaafLab/SRIQ

## Examples

```
java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties
```

## Evaluation signals

- K3, K5, and K6 solutions each retain >80% pairwise co-occurrence similarity to the original BagSize-1200 run across all reduced BagSize levels (1100 down to 300)
- The K3 solution shows nearly 100% similarity at every BagSize level
- K5 similarity begins to drop only around a BagSize of 500, not at higher values
- All other parameters (permutations=10000, iterations=10, method=PEARSON) are held constant across the perturbation series
- Output similarity values are tabulated per perturbation type and level in a single CSV (sriq_stability_similarity.csv)

## Limitations

- The combination of large BagSize, high number of permutations (10000), and large datasets (434 samples, 25785 transcripts) makes running SRIQ computation and memory intensive
- Cluster solutions are sensitive to the cluster diameter (distCutOff) parameter, which must be considered in the context of solution stability and occurrence across iterations
- Stability of cluster assignments does not guarantee detection of small or underrepresented biological subgroups, particularly in smaller datasets
- SRIQ software version is not specified in the source documentation

## Evidence

- [results] BagSize parameter stability evaluation with stepwise reduction from 1100 down to 300: "performed new runs in which the BagSize value was stepwise reduced from 1100 down to 300 (Fig. 2B)"
- [results] Reference clustering run uses BagSize 1200 and 10,000 permutations on 434 LUAD cases: "Application of SRIQ to 434 LUAD cases with expression data for 25,785 transcripts resulted in multiple potential clustering solutions using a BagSize parameter value of 1200 and 10,000 permutations"
- [readme] SRIQ self-evaluates solution stability, motivating perturbation-based reproducibility checks: "SRIQ evaluates clustering solution stability on its own and won't need user input for what number of cluster solutions to be evaluated."
- [readme] test.properties holds the key run parameters including minBagSize and method: "permutations=10000
iterations=10
spiral=TRUE
minClusterSize=0
minBagSize=1200
method=PEARSON"
- [discussion] Computational and memory limits are a stated constraint when running perturbation series: "The combination of a large bag size, high number of permutations, and large datasets (samples and features) can make running SRIQ computation and memory intensive"
