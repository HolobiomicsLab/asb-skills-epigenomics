---
name: known-transcription-factor-recovery-benchmarking
description: Use when a time-series regulatory-network tool must be benchmarked on
  recovery of a curated list of known TFs, requiring matched runs, P-value cut-offs
  (0.01/0.05), unique-hit tallies, and ROC/AUC analysis.
when_to_use_negative:
- The goal is de novo discovery of novel regulatory events rather than benchmarking
  recovery of an already-curated known-TF reference set
- The expression data are not time-series (time-specific regulatory networks and subpath
  parameters are undefined for static data)
- No baseline method or matched parameter settings can be run, so method comparison
  would be confounded
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: TSMiner
  role: Method under test; run on the LR RNA-seq data four times varying only the
    'Maximum number of subpaths' parameter (3, 4, 5, unlimited) to produce time-specific
    regulatory networks for known-TF recovery counting
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: DREM
  role: Baseline method (v. 2.0); run on the same LR RNA-seq input under matched settings
    so recovered known-TF counts can be compared against TSMiner
- name: Java
  role: Runtime environment for TSMiner (Java 1.5 or later); used to launch TSMiner.jar
    from the command line with the -mx1024M heap option
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:451afa95a45b1720133cf5851c93cb0eb8382a093e7de0e62721113cf6522e5b
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# known-transcription-factor-recovery-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a time-specific gene regulatory network method (e.g. TSMiner) against a baseline (DREM v. 2.0) by counting how many known LR-associated transcription factors each recovers from mouse liver regeneration RNA-seq data across parameter settings and P-value cut-offs, yielding sensitivity curves and ROC/AUC scores.

## When to use

Use when a time-series regulatory-network tool must be benchmarked on recovery of a curated list of known TFs, requiring matched runs, P-value cut-offs (0.01/0.05), unique-hit tallies, and ROC/AUC analysis.

## When NOT to use

- The goal is de novo discovery of novel regulatory events rather than benchmarking recovery of an already-curated known-TF reference set
- The expression data are not time-series (time-specific regulatory networks and subpath parameters are undefined for static data)
- No baseline method or matched parameter settings can be run, so method comparison would be confounded

## Inputs

- Time-series RNA-seq expression dataset (mouse liver regeneration, GEO accession GSE95135)
- TF-gene interaction file (from the 'TF-gene file' directory)
- Case-analysis configuration file (Case_analysis_of_mouseLR.txt)
- Defaults file (Defaults_for_case_analysis.txt)
- Curated list of known LR-associated TFs (including 27 cell-proliferation TFs and 10 growth-suppression TFs)

## Outputs

- Per-method, per-subpath-setting counts of recovered known LR-associated TFs at P-value cut-offs 0.01 and 0.05
- Counts of uniquely recovered known TFs per method across settings
- Sensitivity curve over P-value cut-offs spanning 0 to 1
- ROC/AUC scores for the 27 cell-proliferation TFs and 10 growth-suppression TFs at their designated LR periods
- Compiled comparison table (tsminer_drem_comparison_results.csv)

## How to apply

Install Java 1.5+ and launch TSMiner with a 1024 MB heap ('java -mx1024M -jar TSMiner.jar'), then load the mouse LR time-series expression dataset and the TF-gene interaction file using the Case_analysis_of_mouseLR.txt configuration and Defaults_for_case_analysis.txt parameters. Run TSMiner four times on the LR RNA-seq data, varying only the 'Maximum number of subpaths' parameter across 3, 4, 5, and unlimited, saving the time-specific regulatory network model after each run; run DREM (v. 2.0) on the same input under matched settings as the baseline. For each method and each subpath setting, count recovered known LR-associated TFs at P-value cut-offs of 0.01 and 0.05, and tally uniquely recovered TFs across settings. Compute a sensitivity curve over P-value cut-offs spanning 0 to 1, and perform ROC/AUC analysis separately for the 27 cell-proliferation TFs and the 10 growth-suppression TFs at their designated LR periods. Compile all recovery counts, unique-TF counts, sensitivity values, and AUC scores into a results table (e.g. tsminer_drem_comparison_results.csv).

## Related tools

- **TSMiner** (Method under test; run on the LR RNA-seq data four times varying only the 'Maximum number of subpaths' parameter (3, 4, 5, unlimited) to produce time-specific regulatory networks for known-TF recovery counting) — https://github.com/free1234hm/tsminer-tool.git
- **DREM** (Baseline method (v. 2.0); run on the same LR RNA-seq input under matched settings so recovered known-TF counts can be compared against TSMiner)
- **Java** (Runtime environment for TSMiner (Java 1.5 or later); used to launch TSMiner.jar from the command line with the -mx1024M heap option)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- TSMiner should recover more known LR-associated TFs than DREM: at P-value cut-offs of 0.01 and 0.05, TSMiner uniquely identified 10 and 9 known TFs respectively, versus 1 and 4 for DREM
- Exactly four TSMiner runs are performed, one per 'Maximum number of subpaths' setting (3, 4, 5, unlimited), with a saved time-specific regulatory network model after each run
- Sensitivity is evaluated across the full P-value cut-off range 0 to 1, producing a curve rather than single-point values
- AUC analysis is reported separately for the 27 cell-proliferation TFs and the 10 growth-suppression TFs at their designated LR periods
- All recovery counts, unique-TF counts, sensitivity values, and AUC scores are compiled into tsminer_drem_comparison_results.csv with no missing cells

## Limitations

- Fewer results were obtained from negative TF-TG interactions than from positive interactions because the known repressors of many genes were limited
- DREM and its extended methods have limitations (dependence on initial parameters, inability to determine when TFs exert influence, inability to distinguish period-specific subnetworks), which affects the fairness of matched-settings comparison
- The precise regulatory mechanisms require more rigorous evaluation even when experimental results support the predictions

## Evidence

- [results] TSMiner identified more known LR-associated TFs than DREM: at P-value cut-offs of 0.01 and 0.05, TSMiner uniquely identified 10 and 9 known TFs, respectively, whereas DREM provided only 1 and 4 unique results, respectively.: [evidence span withheld — anchor status: not_found]
- [other] Workflow: install Java, launch TSMiner with minimum heap, run four times varying subpaths.: [evidence span withheld — anchor status: not_found]
