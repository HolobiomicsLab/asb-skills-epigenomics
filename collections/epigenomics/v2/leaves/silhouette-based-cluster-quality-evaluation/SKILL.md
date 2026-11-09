---
name: silhouette-based-cluster-quality-evaluation
description: Use when SRIQ clustering output (e.g. ..._Clusters_Frequencies.pdf) lists
  multiple candidate solutions across 10 iterations and you must rank them by recurrence,
  QSS, and silhouette to pick a stable solution.
when_to_use_negative:
- The clustering method requires a predefined K chosen by the analyst (e.g. consensus/bootstrap
  clustering) — silhouette selection of emergent SRIQ solutions across diameter cut-offs
  is not applicable
- Only a single clustering run (no repeated iterations) is available, so recurrence
  and QSS cannot be computed and silhouette alone would overstate stability
- The dataset is too small to detect underrepresented subgroups, making stability/silhouette
  metrics unreliable for solution selection
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: SRIQ
  role: Produces the iterative cluster solutions whose recurrence, QSS, and silhouette
    values are evaluated; includes Silhoutte-plot analysis
  repo: https://github.com/StaafLab/SRIQ
- name: Java
  role: Runtime for SRIQS.jar, which executes the clustering iterations and reports
    solution stability used for QSS/silhouette evaluation
  repo: https://github.com/StaafLab/SRIQ
- name: SRIQPreprocess.jar
  role: Generates the offset, log2-transformed, median-centered, non-expressed-filtered
    (_mc_log2_nz) input on which stable solutions are evaluated
  repo: https://github.com/StaafLab/SRIQ
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:c5f3dfe495f43da044267ea582bfa03614052c36248bdcf99c0f1f3cfbdba633
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# silhouette-based-cluster-quality-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compile recurrence, QSS (quality/stability score), and silhouette values for candidate SRIQ cluster solutions to select highly stable solutions, e.g. the K6 solution at diameter cut-off 0.63 on 434 TCGA-LUAD cases.

## When to use

Use when SRIQ clustering output (e.g. ..._Clusters_Frequencies.pdf) lists multiple candidate solutions across 10 iterations and you must rank them by recurrence, QSS, and silhouette to pick a stable solution.

## When NOT to use

- The clustering method requires a predefined K chosen by the analyst (e.g. consensus/bootstrap clustering) — silhouette selection of emergent SRIQ solutions across diameter cut-offs is not applicable
- Only a single clustering run (no repeated iterations) is available, so recurrence and QSS cannot be computed and silhouette alone would overstate stability
- The dataset is too small to detect underrepresented subgroups, making stability/silhouette metrics unreliable for solution selection

## Inputs

- SRIQ output folder (e.g. .../10000/QC_Spiral(false)/) with per-iteration cluster assignments
- SRIQ cluster-solution frequency plot (..._Clusters_Frequencies.pdf)
- SRIQ test.properties (distCutOff scan, permutations=10000, iterations=10, minBagSize=1200, method=PEARSON)
- Preprocessed expression file (e.g. GeneData_AC_fpkm_mc_log2_nz) for silhouette-plot analysis

## Outputs

- Table of candidate solutions with recurrence count, QSS stability score, and silhouette value
- Selected stable solution identifier (e.g. Results_log_0.63_6, K6 at diameter 0.63, QSS 60%, 100% recurrence)
- Silhouette plots for candidate cluster solutions

## How to apply

After running SRIQ (e.g. BagSize 1200, 10,000 permutations, 10 iterations, PEARSON, spiral=TRUE, distCutOff scanned 0.8 down to 0.52) on the median-centered log2 _mc_log2_nz expression file, inspect the generated ..._Clusters_Frequencies.pdf to identify candidate solutions by their occurrence across the 10 iterations. For each candidate, compile three metrics: recurrence (how often the solution reappears across iterations), the QSS stability score, and the silhouette value produced by the pipeline's Silhoutte-plot analysis feature. Accept a solution only if its recurrence, QSS, and silhouette values are jointly reported and consistent — e.g. the K6 solution at diameter 0.63 (Results_log_0.63_6) showed 100% recurrence across all 10 iterations with QSS of 60% and was selected as highly stable. The cluster diameter must always be considered in the context of the stability and occurrence of a solution across iterations, not as a standalone quality measure.

## Related tools

- **SRIQ** (Produces the iterative cluster solutions whose recurrence, QSS, and silhouette values are evaluated; includes Silhoutte-plot analysis) — https://github.com/StaafLab/SRIQ
- **Java** (Runtime for SRIQS.jar, which executes the clustering iterations and reports solution stability used for QSS/silhouette evaluation) — https://github.com/StaafLab/SRIQ
- **SRIQPreprocess.jar** (Generates the offset, log2-transformed, median-centered, non-expressed-filtered (_mc_log2_nz) input on which stable solutions are evaluated) — https://github.com/StaafLab/SRIQ

## Examples

```
java -jar -Xmx8g SRIQS.jar VRLA path-to/test.properties
```

## Evaluation signals

- The chosen solution is present in ..._Clusters_Frequencies.pdf across all 10 iterations (e.g. 100% recurrence for K6 at diameter 0.63)
- Recurrence, QSS, and silhouette values are all reported for the accepted solution (e.g. QSS = 60% for the K6/diameter-0.63 solution)
- Solution classification matches sample-level assignment: 382 core/spiral samples and 52 unstable samples out of 434 TCGA-LUAD cases
- Output directory Results_log_0.63_6 exists under .../10000/QC_Spiral(false)/ confirming extraction of the evaluated solution
- Silhouette plots are generated by the pipeline's Silhoutte-plot analysis feature for each candidate solution

## Limitations

- Cluster solutions are sensitive to the cluster diameter parameter, which must be evaluated against stability and occurrence across iterations
- The combination of a large bag size, high number of permutations, and large datasets makes running SRIQ computation and memory intensive, limiting how many candidate solutions can be evaluated
- Additional metrics for cluster stability to facilitate cluster solution selection are not yet implemented in the pipeline; SRIQ cannot detect small or underrepresented biological subgroups in smaller datasets

## Evidence

- [results] Compile recurrence, QSS, and silhouette values for each candidate solution: [evidence span withheld — anchor status: not_found]
- [results] K6 solution showed 100% recurrence with QSS of 60%: [evidence span withheld — anchor status: not_found]
