---
name: random-sample-subsetting-of-expression-cohorts
description: Use when assessing how stable an unsupervised clustering solution (e.g.,
  SRIQ K3/K5/K6 on 434 TCGA LUAD samples) is to cohort-size reduction, alongside BagSize
  reduction tests.
when_to_use_negative:
- When the goal is to test parameter sensitivity rather than cohort-size sensitivity
  — use the BagSize stepwise reduction (1100 down to 300) test instead
- When the biological question is differential expression or pathway analysis of established
  clusters (SAM one-versus-rest, EnrichR), not reproducibility testing
- When the dataset is too small to begin with — the article notes SRIQ cannot circumvent
  small datasets limiting detection of small or underrepresented subgroups
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: SRIQ
  role: Clustering engine re-run on each randomly reduced cohort to produce perturbed
    cluster solutions for co-occurrence similarity comparison
  repo: https://github.com/StaafLab/SRIQ
- name: Java
  role: Runtime environment for executing SRIQS.jar (e.g., 'java -jar -Xmx8g SRIQS.jar
    VRLA path-to/test.properties') on each perturbed cohort
- name: Python
  role: Part of the SRIQ Java/Python pipeline; used for computing pairwise co-occurrence
    similarity of sample assignments across runs
  repo: https://github.com/StaafLab/SRIQ
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:3d8ca1a7e632a06a32f6f04d96fafa088f426078b9aa1f8aeba623b019d53338
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# Random sample subsetting of expression cohorts

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A technical reproducibility test for SRIQ clustering in which the cohort size is perturbed by randomly removing an increasing fraction of the original samples while keeping all other parameters intact, then comparing perturbed cluster solutions to the original run.

## When to use

Use when assessing how stable an unsupervised clustering solution (e.g., SRIQ K3/K5/K6 on 434 TCGA LUAD samples) is to cohort-size reduction, alongside BagSize reduction tests.

## When NOT to use

- When the goal is to test parameter sensitivity rather than cohort-size sensitivity — use the BagSize stepwise reduction (1100 down to 300) test instead
- When the biological question is differential expression or pathway analysis of established clusters (SAM one-versus-rest, EnrichR), not reproducibility testing
- When the dataset is too small to begin with — the article notes SRIQ cannot circumvent small datasets limiting detection of small or underrepresented subgroups

## Inputs

- Preprocessed log2-transformed, median-centered expression matrix (e.g., GeneData_AC_fpkm_mc_log2_nz, tab-separated .txt from SRIQPreprocess.jar)
- SRIQ test.properties configuration file (permutations=10000, iterations=10, method=PEARSON, minBagSize=1200)
- Reference SRIQ run output with cluster solutions (K3, K5, K6) and sample assignments

## Outputs

- Cluster solutions (K3, K5, K6) for each randomly reduced-cohort SRIQ run
- Pairwise co-occurrence similarity table per perturbation level and cluster solution (e.g., sriq_stability_similarity.csv)
- Stability verdict: >80% similarity for K3/K5/K6 with K3 near 100%

## How to apply

Starting from the reference SRIQ run (434 LUAD samples, log2-transformed median-centered FPKM matrix GeneData_AC_fpkm_mc_log2_nz, test.properties with permutations=10000, iterations=10, minBagSize=1200, method=PEARSON), repeatedly create perturbed cohorts by randomly removing an increasing fraction of the original samples while keeping BagSize, permutations, and the distance method intact. Run SRIQ on each reduced cohort via 'java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties'. For each perturbed run, extract the resulting cluster solutions (K3, K5, K6) and compute the pairwise co-occurrence similarity of sample assignments against the corresponding solution from the original full-cohort run. Tabulate similarity per perturbation type and level (e.g., sriq_stability_similarity.csv); accept stability if K3, K5, and K6 retain >80% pairwise co-occurrence similarity, with K3 near 100%. The rationale is that a robust solution should reproduce its core/spiral sample assignments even when a substantial fraction of the cohort is missing.

## Related tools

- **SRIQ** (Clustering engine re-run on each randomly reduced cohort to produce perturbed cluster solutions for co-occurrence similarity comparison) — https://github.com/StaafLab/SRIQ
- **Java** (Runtime environment for executing SRIQS.jar (e.g., 'java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties') on each perturbed cohort)
- **Python** (Part of the SRIQ Java/Python pipeline; used for computing pairwise co-occurrence similarity of sample assignments across runs) — https://github.com/StaafLab/SRIQ

## Examples

```
java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties
```

## Evaluation signals

- K3, K5, and K6 solutions each retain >80% pairwise co-occurrence similarity to the original BagSize-1200, full-cohort run
- K3 solution shows near-100% co-occurrence similarity for each perturbation level
- Similarity values are tabulated for every sample-fraction reduction level and match the reported stability outcome when compiled into sriq_stability_similarity.csv
- All other SRIQ parameters (BagSize=1200, permutations=10000, method=PEARSON, iterations=10) are held constant across perturbed runs
- Each perturbed run uses a documented random fraction of removed samples so the perturbation series is reproducible

## Limitations

- Stability testing is computationally and memory intensive: large BagSize, high permutation counts (10,000), and large sample/feature sets (434 samples, 25,785 transcripts) require substantial resources (e.g., -Xmx8g)
- Solutions are sensitive to the cluster diameter parameter, which must be considered in the context of stability and occurrence across iterations
- Stable overall solutions do not guarantee detection of small or underrepresented biological subgroups, especially in smaller reduced cohorts
- Unstable samples (52 of 434 in the reference run) are excluded from core/spiral membership, so similarity metrics reflect core/spiral sample assignments (382 samples)

## Evidence
