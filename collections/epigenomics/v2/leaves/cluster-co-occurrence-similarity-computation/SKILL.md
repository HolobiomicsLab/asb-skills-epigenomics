---
name: cluster-co-occurrence-similarity-computation
description: Use when you have two or more sample-to-cluster assignments over the
  same cohort (e.g., a reference SRIQ run at BagSize 1200 and perturbed runs) and
  need to quantify cluster stability per K solution.
when_to_use_negative:
- Comparing cluster assignments over different sample sets with no shared cohort (co-occurrence
  requires the same samples to compare pair membership)
- Evaluating biological relevance or subtype validation (e.g., differential expression,
  Cibersort immune fractions) rather than technical reproducibility of the clustering
- The clustering method already outputs its own stability metric and no cross-run
  perturbation comparison is planned
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: SRIQ
  role: Unsupervised clustering engine producing the cluster solutions whose stability
    is compared across perturbed runs
  repo: https://github.com/StaafLab/SRIQ
- name: Java
  role: Runtime for SRIQS.jar (VRLA clustering and EXTRACT commands) used to generate
    reference and perturbed runs
  repo: https://github.com/StaafLab/SRIQ
- name: Python
  role: Used in the SRIQ pipeline for downstream analysis and tabulation of stability
    similarity values
  repo: https://github.com/StaafLab/SRIQ
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:6d9bba547d4742de65d4050da59affde49047b2734204f32aa4b01a162faaaf4
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
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cluster-co-occurrence-similarity-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies technical reproducibility of unsupervised cluster solutions by computing pairwise co-occurrence similarity of sample assignments from perturbed clustering runs (e.g., SRIQ with reduced BagSize or reduced sample fraction) against a reference run, for each cluster solution (K3, K5, K6).

## When to use

Use when you have two or more sample-to-cluster assignments over the same cohort (e.g., a reference SRIQ run at BagSize 1200 and perturbed runs) and need to quantify cluster stability per K solution.

## When NOT to use

- Comparing cluster assignments over different sample sets with no shared cohort (co-occurrence requires the same samples to compare pair membership)
- Evaluating biological relevance or subtype validation (e.g., differential expression, Cibersort immune fractions) rather than technical reproducibility of the clustering
- The clustering method already outputs its own stability metric and no cross-run perturbation comparison is planned

## Inputs

- Preprocessed log2-transformed, median-centered expression matrix (e.g., GeneData_AC_fpkm_mc_log2_nz, tab-separated .txt)
- SRIQ test.properties configuration (permutations=10000, iterations=10, method=PEARSON, minBagSize=1200)
- Reference SRIQ cluster solution (sample-to-cluster assignments for K3, K5, K6)
- Perturbed SRIQ run outputs (BagSize 1100–300 series; sample-fraction reduction series)

## Outputs

- Pairwise co-occurrence similarity values per cluster solution (K3, K5, K6) per perturbation type and level
- sriq_stability_similarity.csv table of similarity values

## How to apply

First obtain a reference clustering run, e.g., SRIQ on the preprocessed log2-transformed, median-centered LUAD expression matrix (GeneData_AC_fpkm_mc_log2_nz) with test.properties set to permutations=10000, iterations=10, method=PEARSON, minBagSize=1200. Then rerun the clustering under perturbations: stepwise reduce the BagSize from 1100 down to 300 keeping all other parameters intact, and/or randomly remove an increasing fraction of the original samples while keeping BagSize, permutations, and distance method intact. For each perturbed run, extract the cluster solution (e.g., via the SRIQS.jar EXTRACT command with chosen diameter and number of clusters) and compute the pairwise co-occurrence similarity of sample assignments against the original run separately for each cluster solution (K3, K5, K6). Tabulate similarity per perturbation type and level and save as sriq_stability_similarity.csv; accept the stability result if K3, K5, and K6 retain >80% pairwise co-occurrence similarity, with K3 near 100%.

## Related tools

- **SRIQ** (Unsupervised clustering engine producing the cluster solutions whose stability is compared across perturbed runs) — https://github.com/StaafLab/SRIQ
- **Java** (Runtime for SRIQS.jar (VRLA clustering and EXTRACT commands) used to generate reference and perturbed runs) — https://github.com/StaafLab/SRIQ
- **Python** (Used in the SRIQ pipeline for downstream analysis and tabulation of stability similarity values) — https://github.com/StaafLab/SRIQ

## Examples

```
java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties
```

## Evaluation signals

- K3, K5, and K6 solutions each retain >80% pairwise co-occurrence similarity to the original BagSize-1200 run across BagSize reductions from 1100 to 300
- K3 solution shows nearly 100% similarity for each BagSize-reduced run
- K5 similarity remains high until it starts to drop around a BagSize of 500, matching the reported stability outcome
- sriq_stability_similarity.csv contains one similarity value per perturbation type and level for each of the three cluster solutions
- Perturbed series differ from the reference only in BagSize or sample fraction, with permutations=10000, iterations=10, and method=PEARSON unchanged

## Limitations

- Cluster solutions are sensitive to the cluster diameter parameter, which must be evaluated in the context of solution stability and occurrence across iterations
- The combination of large bag size, high permutation count, and large datasets makes the perturbation series computationally and memory intensive
- Even high technical reproducibility does not guarantee detection of small or underrepresented biological subgroups in smaller datasets

## Evidence

- [results] BagSize reduction series design: "performed new runs in which the BagSize value was stepwise reduced from 1100 down to 300 (Fig. 2B)"
- [results] Sample-fraction reduction design: "we changed the sample size for SRIQ by randomly removing an increasing fraction of the original samples, while keeping other parameters intact"
- [results] Reference run parameters: "using a BagSize parameter value of 1200 and 10,000 permutations (Fig. 2A, Supplementary Table 1)"
- [other] Co-occurrence acceptance threshold: [evidence span withheld — anchor status: not_found]
