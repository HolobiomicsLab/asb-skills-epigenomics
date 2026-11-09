---
name: sensitivity-curve-analysis-across-cutoffs
description: Use when comparing a method (e.g., TSMiner vs DREM) on recovering a known
  ground-truth TF set, and you need sensitivity as a function of P-value cut-off plus
  unique-TF counts across parameter settings.
when_to_use_negative:
- There is no curated ground-truth set of known TFs to score recovery against — sensitivity
  and AUC are undefined without it.
- The analysis goal is constructing or visualizing the time-specific regulatory networks
  themselves rather than benchmarking recovery performance.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: TSMiner
  role: Method under evaluation; run with 'Maximum number of subpaths' varied across
    3, 4, 5, and unlimited to generate time-specific regulatory network models for
    TF recovery counting
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: DREM
  role: Baseline method (v. 2.0) run on the same LR RNA-seq input under matched settings
    for sensitivity comparison
- name: Java
  role: Runtime used to launch TSMiner with a minimum 1024 MB heap (java -mx1024M
    -jar TSMiner.jar)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:b3142c6a9594d60851e3b381bb62f3daf8df52e9101805088950f1154df930ca
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
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sensitivity-curve-analysis-across-cutoffs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a network-inference method by sweeping P-value significance cut-offs (0 to 1), computing sensitivity curves and ROC/AUC scores for recovering known LR-associated TFs (e.g., 27 cell-proliferation TFs and 10 growth-suppression TFs), and tallying uniquely detected TFs across parameter settings.

## When to use

Use when comparing a method (e.g., TSMiner vs DREM) on recovering a known ground-truth TF set, and you need sensitivity as a function of P-value cut-off plus unique-TF counts across parameter settings.

## When NOT to use

- There is no curated ground-truth set of known TFs to score recovery against — sensitivity and AUC are undefined without it.
- The analysis goal is constructing or visualizing the time-specific regulatory networks themselves rather than benchmarking recovery performance.

## Inputs

- Time-specific regulatory network models output by TSMiner across 'Maximum number of subpaths' settings (3, 4, 5, unlimited)
- DREM (v. 2.0) output on the same LR RNA-seq input under matched settings
- Curated list of known LR-associated TFs (ground truth), including 27 cell-proliferation TFs and 10 growth-suppression TFs with designated LR periods
- P-value significance cut-off range (0 to 1), with 0.01 and 0.05 as standard thresholds

## Outputs

- Per-method, per-cut-off counts of recovered known LR-associated TFs (e.g., at P-value cut-offs 0.01 and 0.05)
- Counts of uniquely detected known TFs per method across parameter settings
- Sensitivity curve over P-value cut-offs spanning 0 to 1
- ROC/AUC scores for the 27 cell-proliferation TFs and 10 growth-suppression TFs
- Consolidated results table, e.g., tsminer_drem_comparison_results.csv

## How to apply

Run the tool under comparison (e.g., TSMiner with 'Maximum number of subpaths' set to 3, 4, 5, and unlimited) on the mouse LR time-series expression data, saving the resulting time-specific regulatory network models after each run. Run the baseline method (DREM v. 2.0) under matched settings on the same LR RNA-seq input. At each P-value cut-off of interest (0.01 and 0.05 as standard significance thresholds), count recovered known LR-associated TFs per method per setting and tally TFs uniquely recovered across settings. Then sweep P-value cut-offs spanning 0 to 1 to compute the sensitivity curve, and perform ROC/AUC analysis for designated TF groups: the 27 cell-proliferation TFs and 10 growth-suppression TFs at their designated LR periods. Compile all recovery counts, unique-TF counts, sensitivity values, and AUC scores into a results table (e.g., tsminer_drem_comparison_results.csv).

## Related tools

- **TSMiner** (Method under evaluation; run with 'Maximum number of subpaths' varied across 3, 4, 5, and unlimited to generate time-specific regulatory network models for TF recovery counting) — https://github.com/free1234hm/tsminer-tool.git
- **DREM** (Baseline method (v. 2.0) run on the same LR RNA-seq input under matched settings for sensitivity comparison)
- **Java** (Runtime used to launch TSMiner with a minimum 1024 MB heap (java -mx1024M -jar TSMiner.jar))

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- Expected direction of result: TSMiner uniquely identified more known LR-associated TFs than DREM (10 vs 1 at P-value cut-off 0.01; 9 vs 4 at 0.05) — a replication should reproduce this ordering.
- Sensitivity values must increase monotonically (non-decreasing) as the P-value cut-off is relaxed from 0 toward 1.
- Every recovered TF counted must belong to the curated known LR-associated TF list; no TF outside the ground truth contributes to sensitivity.
- Unique-TF counts per method should sum across parameter settings (3, 4, 5, unlimited) without double-counting the same TF recovered in multiple settings.
- AUC scores must fall in [0, 1] and be reported separately for the 27 cell-proliferation TFs and 10 growth-suppression TFs at their designated LR periods.

## Limitations

- Fewer results are obtained from negative TF–TG interactions than from positive interactions because the known repressors of many genes were limited; future collection of negatively correlated TF–TG interactions may help address this issue.
- The comparison depends on the quality of the curated ground truth (known LR-associated TFs); DREM and its extended methods have documented limitations such as dependence on initial parameters without effective initialization, which may bias the baseline's recovery counts.
- Findings are specific to the mouse liver regeneration (LR) RNA-seq dataset (GSE95135); sensitivity rankings may not transfer to other time-series datasets.

## Evidence
