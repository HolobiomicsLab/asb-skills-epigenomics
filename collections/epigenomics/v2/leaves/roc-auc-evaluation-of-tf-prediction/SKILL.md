---
name: roc-auc-evaluation-of-tf-prediction
description: Use when comparing tools that predict transcription factors from time-series
  expression data (e.g. liver-regeneration RNA-seq) and you must quantify sensitivity/accuracy
  against a curated list of known TFs.
when_to_use_negative:
- The expression data are not time-series (TSMiner/DREM require time-course profiles
  to build tree-structure or time-specific models).
- No curated set of known TFs exists for the studied process, making recovery counting
  and ROC/AUC analysis impossible.
- The goal is clustering of expression modules rather than TF-level benchmarking —
  clustering tools cannot evaluate temporal dynamics of regulators.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: TSMiner
  role: Primary method whose TF predictions are benchmarked; run four times on the
    LR data with 'Maximum number of subpaths' set to 3, 4, 5, and unlimited
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: DREM
  role: Baseline method (v. 2.0) run on the same LR RNA-seq input under matched settings
    for comparison of known-TF recovery
- name: Java
  role: Runtime used to launch TSMiner from the command line with a minimum 1024 MB
    heap (java -mx1024M -jar TSMiner.jar)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:df00e8db372a098b2636ff6fdbb7515a752198acdbfba31608cd704c9b5188f2
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# roc-auc-evaluation-of-tf-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a time-specific regulatory network method (e.g. TSMiner vs DREM v2.0) on its ability to recover known LR-associated TFs, counting recovered and unique TFs at P-value cut-offs of 0.01 and 0.05 and computing sensitivity curves and ROC/AUC scores across P-value cut-offs spanning 0 to 1.

## When to use

Use when comparing tools that predict transcription factors from time-series expression data (e.g. liver-regeneration RNA-seq) and you must quantify sensitivity/accuracy against a curated list of known TFs.

## When NOT to use

- The expression data are not time-series (TSMiner/DREM require time-course profiles to build tree-structure or time-specific models).
- No curated set of known TFs exists for the studied process, making recovery counting and ROC/AUC analysis impossible.
- The goal is clustering of expression modules rather than TF-level benchmarking — clustering tools cannot evaluate temporal dynamics of regulators.

## Inputs

- LR time-series RNA-seq expression dataset (GSE95135) loaded via the 'Time-series data' directory
- TF-gene interaction file from the 'TF-gene file' directory
- Case-analysis configuration file Case_analysis_of_mouseLR.txt with defaults from Defaults_for_case_analysis.txt
- Curated lists of known LR-associated TFs (27 cell-proliferation TFs and 10 growth-suppression TFs)
- DREM (v. 2.0) output as the baseline prediction set

## Outputs

- Recovery counts of known LR-associated TFs per method and subpath setting at P-value cut-offs 0.01 and 0.05
- Counts of uniquely detected known TFs per method (e.g. TSMiner 10 and 9 vs DREM 1 and 4)
- Sensitivity curves over P-value cut-offs spanning 0 to 1
- AUC scores for the 27 cell-proliferation TFs and 10 growth-suppression TFs at their designated LR periods
- tsminer_drem_comparison_results.csv

## How to apply

Run each method on the same mouse liver regeneration (LR) RNA-seq input — TSMiner four times varying only the 'Maximum number of subpaths' parameter across 3, 4, 5, and unlimited — and save the time-specific regulatory network models after each run. For each method and each subpath setting, count recovered known LR-associated TFs at P-value cut-offs of 0.01 and 0.05, and tally the TFs uniquely recovered across settings. Compute the sensitivity curve over P-value cut-offs spanning 0 to 1, then perform ROC/AUC analysis separately for the 27 cell-proliferation TFs and the 10 growth-suppression TFs at their designated LR periods. Compile recovery counts, unique-TF counts, sensitivity values, and AUC scores into a results table (e.g. tsminer_drem_comparison_results.csv) and compare against the baseline tool (DREM v2.0).

## Related tools

- **TSMiner** (Primary method whose TF predictions are benchmarked; run four times on the LR data with 'Maximum number of subpaths' set to 3, 4, 5, and unlimited) — https://github.com/free1234hm/tsminer-tool.git
- **DREM** (Baseline method (v. 2.0) run on the same LR RNA-seq input under matched settings for comparison of known-TF recovery)
- **Java** (Runtime used to launch TSMiner from the command line with a minimum 1024 MB heap (java -mx1024M -jar TSMiner.jar))

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- At P-value cut-offs 0.01 and 0.05, TSMiner should uniquely identify 10 and 9 known LR-associated TFs, respectively, versus 1 and 4 for DREM.
- Sensitivity curves are computed over the full P-value cut-off range 0 to 1, not just the two standard cut-offs.
- AUC is reported separately for the 27 cell-proliferation TFs and the 10 growth-suppression TFs at their designated LR periods.
- All recovery counts, unique-TF counts, sensitivity values, and AUC scores are compiled into a single results file (tsminer_drem_comparison_results.csv).
- Each TSMiner run varies only the 'Maximum number of subpaths' parameter (3, 4, 5, unlimited); all other settings match the Defaults_for_case_analysis.txt configuration.

## Limitations

- DREM relies on initial parameters without effective initialization and cannot determine when TFs exert influence on targets or distinguish subnetworks occurring in specific periods, which may depress its recovery counts.
- Fewer results are obtained from negative TF–TG interactions than positive ones because known repressors of many genes are limited.
- Precise regulatory mechanisms supported by these benchmarks require more rigorous experimental evaluation beyond the computational comparison.

## Evidence

- [results] at P-value cut-offs of 0.01 and 0.05, TSMiner uniquely identified 10 and 9 known TFs, respectively, whereas DREM provided only 1 and 4 unique results: [evidence span withheld — anchor status: not_found]
