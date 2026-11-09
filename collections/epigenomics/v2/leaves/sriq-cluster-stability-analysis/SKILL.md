---
name: sriq-cluster-stability-analysis
description: Use when SRIQ cluster solutions have been obtained (e.g. K3/K5/K6 at
  BagSize 1200, 10000 permutations) and you must verify that they remain reproducible
  under parameter or cohort-size perturbation.
when_to_use_negative:
- You have not yet produced a reference SRIQ clustering solution (e.g. at BagSize
  1200 with 10000 permutations) to compare perturbed runs against
- The goal is biological validation of clusters (differential expression, Cibersort
  immune deconvolution, UMAP separation) rather than technical reproducibility testing
- You are benchmarking SRIQ against another clustering method such as consensus clustering
  (ward's linkage, Pearson correlation, 2000 iterations, 70% resampling) rather than
  testing SRIQ's own stability
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: SRIQ
  role: Clustering engine run with perturbed BagSize values and reduced sample sets
    to generate stability-test cluster solutions
  repo: https://github.com/StaafLab/SRIQ
- name: Java
  role: Runtime for SRIQS.jar (e.g. java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties)
    used to execute each perturbed SRIQ run
  repo: https://github.com/StaafLab/SRIQ
- name: Python
  role: Pipeline component used for pre-processing and downstream computation/tabulation
    of pairwise co-occurrence similarity values
  repo: https://github.com/StaafLab/SRIQ
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:5db908b5ca5853efe9843284683e311e0f5d8f31b3ea4dab84b05debbe77efc3
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
yes_rate: 0.875
n_claims: 8
---

# sriq-cluster-stability-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A stability-testing procedure for SRIQ clustering solutions (e.g. K3, K5, K6) in which the BagSize parameter is stepwise reduced from 1100 down to 300 and sample fractions are randomly removed, with pairwise co-occurrence similarity computed against the original BagSize-1200 run.

## When to use

Use when SRIQ cluster solutions have been obtained (e.g. K3/K5/K6 at BagSize 1200, 10000 permutations) and you must verify that they remain reproducible under parameter or cohort-size perturbation.

## When NOT to use

- You have not yet produced a reference SRIQ clustering solution (e.g. at BagSize 1200 with 10000 permutations) to compare perturbed runs against
- The goal is biological validation of clusters (differential expression, Cibersort immune deconvolution, UMAP separation) rather than technical reproducibility testing
- You are benchmarking SRIQ against another clustering method such as consensus clustering (ward's linkage, Pearson correlation, 2000 iterations, 70% resampling) rather than testing SRIQ's own stability

## Inputs

- Preprocessed log2-transformed, median-centered expression matrix (e.g. GeneData_AC_fpkm_mc_log2_nz, tab-separated .txt)
- test.properties configuration file (permutations=10000, iterations=10, distCutOff series, spiral=TRUE, minClusterSize=0, minBagSize=1200, method=PEARSON)
- Reference SRIQ run output at BagSize 1200 (cluster assignments for K3, K5, K6)

## Outputs

- Perturbed SRIQ cluster solutions for each BagSize level (1100–300) and each sample-fraction reduction
- Pairwise co-occurrence similarity table of sample assignments per perturbation and cluster solution (e.g. sriq_stability_similarity.csv)

## How to apply

Start from the preprocessed log2-transformed, median-centered expression matrix (e.g. GeneData_AC_fpkm_mc_log2_nz) and the reference test.properties configuration (permutations=10000, iterations=10, method=PEARSON, minBagSize=1200). Run a first stability series with 'java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties' in which minBagSize is stepwise reduced from 1100 down to 300, keeping all other parameters intact. Run a second stability series in which an increasing fraction of the original samples is randomly removed while keeping BagSize, permutations, and distance method intact. For each perturbed run, extract the cluster solution and compute the pairwise co-occurrence similarity of sample assignments against the original BagSize-1200 run, separately per cluster solution (K3, K5, K6). Tabulate similarity values per perturbation type and level (e.g. as sriq_stability_similarity.csv). Accept the stability result if all K3, K5 and K6 solutions retain >80% pairwise co-occurrence similarity to the original run, with the K3 solution near 100%; note that K5 similarity begins dropping around BagSize 500.

## Related tools

- **SRIQ** (Clustering engine run with perturbed BagSize values and reduced sample sets to generate stability-test cluster solutions) — https://github.com/StaafLab/SRIQ
- **Java** (Runtime for SRIQS.jar (e.g. java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties) used to execute each perturbed SRIQ run) — https://github.com/StaafLab/SRIQ
- **Python** (Pipeline component used for pre-processing and downstream computation/tabulation of pairwise co-occurrence similarity values) — https://github.com/StaafLab/SRIQ

## Examples

```
java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties  # with minBagSize stepwise reduced 1100→300 in test.properties
```

## Evaluation signals

- K3, K5 and K6 solutions each retain >80% pairwise co-occurrence similarity to the original BagSize-1200 run
- The K3 solution shows near-100% similarity for each BagSize reduction step
- K5 similarity begins to drop around BagSize 500, providing a sanity anchor for the perturbation series
- Similarity values are tabulated per perturbation type (BagSize vs. sample-fraction reduction) and level in a file such as sriq_stability_similarity.csv
- All perturbed runs use unchanged permutations=10000, iterations=10, and method=PEARSON relative to the reference run

## Limitations

- SRIQ is computationally and memory intensive when combining large bag size, high number of permutations, and large datasets (samples and features)
- Cluster solutions are sensitive to the cluster diameter parameter, which must be evaluated in the context of stability and occurrence across iterations
- SRIQ cannot circumvent the general problem of smaller datasets limiting detection of small or underrepresented biological subgroups, which is relevant when randomly removing samples in the stability series

## Evidence
