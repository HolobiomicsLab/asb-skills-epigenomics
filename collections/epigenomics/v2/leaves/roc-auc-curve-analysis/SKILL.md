---
name: roc-auc-curve-analysis
description: Use when comparing how sensitively and accurately multiple methods (e.g.,
  TSMiner vs clustering tools) recover a known positive set of pathways/TFs from time-series
  expression data, and a negative set can be defined.
when_to_use_negative:
- Input lacks a defined positive reference set (e.g., no list of known LR-associated
  pathways or known TFs) — ROC/AUC against an arbitrary negative set is meaningless.
- The goal is to construct time-specific regulatory networks itself, not to evaluate
  detection sensitivity/accuracy across competing methods.
- Only one method's predictions exist with no comparable ranked predictions from other
  methods — there is no ranking to sweep thresholds over for a fair AUC comparison.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: TSMiner
  role: Primary method under evaluation; its predicted TF-pathway interactions (activator
    and repressor groups) are scored for sensitivity and AUC against known LR-associated
    pathways
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: DREM
  role: Comparison method (v. 2.0) run on the same LR RNA-seq data, with pathway enrichment,
    to benchmark TSMiner's ROC/AUC performance
- name: Java
  role: Runtime for launching TSMiner v1.2.0 with a 1024 MB heap during the benchmark
    runs
  repo: http://www.java.com
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:fe73d498f0d2293088328be6b2f26d305be786e874792ae592026c386c254f4d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/free1234hm/tsminer-tool
  repo_url: https://github.com/free1234hm/tsminer-tool.git
  tools:
  - DREM
  - Java
  - TSMiner
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# roc-auc-curve-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark sensitivity and accuracy of pathway-detection methods (TSMiner, DREM v2.0, hclust, k-means, STEM, WGCNA) by sweeping P-value thresholds 0–0.05 against known LR-associated pathways and building ROC curves with AUC against a defined negative pathway set.

## When to use

Use when comparing how sensitively and accurately multiple methods (e.g., TSMiner vs clustering tools) recover a known positive set of pathways/TFs from time-series expression data, and a negative set can be defined.

## When NOT to use

- Input lacks a defined positive reference set (e.g., no list of known LR-associated pathways or known TFs) — ROC/AUC against an arbitrary negative set is meaningless.
- The goal is to construct time-specific regulatory networks itself, not to evaluate detection sensitivity/accuracy across competing methods.
- Only one method's predictions exist with no comparable ranked predictions from other methods — there is no ranking to sweep thresholds over for a fair AUC comparison.

## Inputs

- Time-series expression matrix from mouse liver regeneration RNA-seq data (GEO accession GSE95135)
- TF–gene interaction files (positively and negatively correlated TF–TG interactions processed separately)
- KEGG/Reactome pathway annotation files (from the 'Gene annotation file' directory of tsminer-tool)
- Pathway predictions from each compared method: TSMiner network output for activator and repressor groups, DREM v2.0 results, and hclust/k-means/STEM/WGCNA cluster enrichment results
- Positive reference set: 58 known LR-associated KEGG pathways; negative set: 78 pathways drawn from the 499-pathway background

## Outputs

- Sensitivity-vs-P-value-threshold curves (thresholds 0 to 0.05) per method against the 58 known LR-associated pathways
- ROC curves and AUC values per method against the 78-pathway negative set
- Count of uniquely identified pathways per method
- Consolidated benchmark table (e.g., tsminer_pathway_benchmark_results.csv)

## How to apply

Run each method (TSMiner v1.2.0 via java -mx1024M -jar TSMiner.jar with Case_analysis_of_mouseLR.txt defaults; DREM v2.0; hclust, k-means, STEM, WGCNA each followed by KEGG/Reactome pathway enrichment) on the same mouse liver regeneration RNA-seq time series (GSE95135). Extract predicted TF-pathway interactions from the TSMiner activator and repressor network outputs and from each comparison method's enrichment results. Compute sensitivity at P-value thresholds sweeping from 0 to 0.05 against the 58 known LR-associated KEGG pathways, and tally uniquely identified pathways per method. Build ROC curves and compute AUC for each method using a 78-pathway negative set drawn from the 499-pathway background. Consolidate per-method sensitivity, unique counts and AUC into a results table (e.g., tsminer_pathway_benchmark_results.csv); validate that TSMiner ranks highest on the sensitivity curve and AUC.

## Related tools

- **TSMiner** (Primary method under evaluation; its predicted TF-pathway interactions (activator and repressor groups) are scored for sensitivity and AUC against known LR-associated pathways) — https://github.com/free1234hm/tsminer-tool.git
- **DREM** (Comparison method (v. 2.0) run on the same LR RNA-seq data, with pathway enrichment, to benchmark TSMiner's ROC/AUC performance)
- **Java** (Runtime for launching TSMiner v1.2.0 with a 1024 MB heap during the benchmark runs) — http://www.java.com

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- The sensitivity curve at P-value thresholds 0–0.05 reproduces TSMiner ranking highest among TSMiner, DREM v2.0, hclust, k-means, STEM and WGCNA.
- AUC values computed against the 78-pathway negative set (from the 499-pathway background) place TSMiner first in accuracy, as reported in the paper.
- The benchmark table contains per-method sensitivity, unique-pathway counts and AUC, and is accepted only if the TSMiner-highest ranking holds.
- All methods are evaluated on the same LR RNA-seq input (GSE95135) and the same 58 known LR-associated pathway reference set, ensuring a like-for-like comparison.
- Unique-count tallies per method match the paper's expectation that TSMiner identifies pathways the four clustering methods miss.

## Limitations

- Fewer results were obtained from negative TF–TG interactions than positive interactions because known repressors of many genes are limited, which can skew repressor-group benchmarking.
- Sensitivity/accuracy conclusions depend on the choice of the 58-pathway positive set and the 78-pathway negative set; results are specific to the LR KEGG benchmark.
- Clustering methods compared here cannot evaluate the temporal dynamics of regulators or pathways, so the comparison covers detection sensitivity but not temporal resolution.

## Evidence

- [other] Build ROC curves and compute AUC for each method against the 78-pathway negative set drawn from the 499-pathway background.: [evidence span withheld — anchor status: not_found]
- [other] TSMiner displayed consistently and significantly higher sensitivity in detecting known LR-associated pathways than the four clustering methods at P-value thresholds ranging from 0 to 0.05.: [evidence span withheld — anchor status: not_found]
