---
name: parameter-sweep-comparison-across-tools
description: Use when a published method exposes a discrete structural parameter (e.g.,
  maximum subpaths) whose effect on known-target recovery must be quantified and compared
  head-to-head with a baseline tool on the same dataset.
when_to_use_negative:
- Input is single-condition or non-time-series expression data — TSMiner constructs
  time-specific regulatory networks specifically from time-series expression profiles
- No curated set of known targets (e.g., known LR-associated TFs) exists to serve
  as ground truth for recovery counting
- The goal is pathway- or gene-set enrichment alone rather than comparing regulator
  detection across tool/parameter settings
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: TSMiner
  role: Primary method whose 'Maximum number of subpaths' parameter is swept (3, 4,
    5, unlimited) to produce time-specific TF–pathway regulatory network models for
    the LR RNA-seq data
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: DREM
  role: Baseline tree-structure method (v. 2.0) run on the same LR RNA-seq input under
    matched settings for head-to-head known-TF recovery comparison
- name: Java
  role: Runtime required to launch TSMiner (v1.2.0) from the command line with a minimum
    1024 MB Java heap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:affd239cdcfc2418bfb2e886c832788e49e56dc1a91bdc4e228d56b7b4bcceba
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# parameter-sweep-comparison-across-tools

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for sweeping a key tool parameter (e.g., TSMiner's 'Maximum number of subpaths' across 3, 4, 5, and unlimited) while holding input data fixed, then comparing recovered results against a baseline tool (DREM v. 2.0) at standard significance cut-offs to quantify tool sensitivity.

## When to use

Use when a published method exposes a discrete structural parameter (e.g., maximum subpaths) whose effect on known-target recovery must be quantified and compared head-to-head with a baseline tool on the same dataset.

## When NOT to use

- Input is single-condition or non-time-series expression data — TSMiner constructs time-specific regulatory networks specifically from time-series expression profiles
- No curated set of known targets (e.g., known LR-associated TFs) exists to serve as ground truth for recovery counting
- The goal is pathway- or gene-set enrichment alone rather than comparing regulator detection across tool/parameter settings

## Inputs

- Mouse liver regeneration (LR) time-series RNA-seq expression dataset (GSE95135)
- TF-gene interaction file from the 'TF-gene file' directory
- Case-analysis configuration file Case_analysis_of_mouseLR.txt
- Defaults_for_case_analysis.txt parameter file
- Baseline method DREM (v. 2.0) with matched input settings

## Outputs

- Time-specific regulatory network models saved after each TSMiner run (subpaths = 3, 4, 5, unlimited)
- Counts of known LR-associated TFs recovered at P-value cut-offs 0.01 and 0.05 per method per setting
- Tally of uniquely detected known TFs per tool and setting
- Sensitivity curve over P-value cut-offs spanning 0 to 1
- ROC/AUC scores for the 27 cell-proliferation TFs and 10 growth-suppression TFs
- tsminer_drem_comparison_results.csv compiling recovery counts, unique-TF counts, sensitivity values, and AUC scores

## How to apply

Install Java (1.5 or later) and launch TSMiner with a 1024 MB heap via 'java -mx1024M -jar TSMiner.jar'. Load the mouse LR time-series expression dataset (GSE95135) and TF-gene interaction files, configuring the run with Case_analysis_of_mouseLR.txt and the defaults in Defaults_for_case_analysis.txt per TSMiner manual.pdf. Run TSMiner four times on the LR RNA-seq data, varying only the 'Maximum number of subpaths' parameter across 3, 4, 5, and unlimited, saving the time-specific regulatory network model after each run. Run DREM (v. 2.0) on the identical input under matched settings as the baseline. For each method and subpath setting, count recovered known LR-associated TFs at P-value cut-offs of 0.01 and 0.05, tally uniquely recovered TFs per setting, and compute sensitivity curves over P-value cut-offs spanning 0 to 1 plus ROC/AUC analysis for the 27 cell-proliferation TFs and 10 growth-suppression TFs at their designated LR periods. Compile all counts and AUC scores into a single results table (tsminer_drem_comparison_results.csv) for cross-setting comparison.

## Related tools

- **TSMiner** (Primary method whose 'Maximum number of subpaths' parameter is swept (3, 4, 5, unlimited) to produce time-specific TF–pathway regulatory network models for the LR RNA-seq data) — https://github.com/free1234hm/tsminer-tool.git
- **DREM** (Baseline tree-structure method (v. 2.0) run on the same LR RNA-seq input under matched settings for head-to-head known-TF recovery comparison)
- **Java** (Runtime required to launch TSMiner (v1.2.0) from the command line with a minimum 1024 MB Java heap)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- TSMiner runs at all four subpath settings (3, 4, 5, unlimited) complete and each produces a saved time-specific regulatory network model
- Recovery counts at P-value cut-offs 0.01 and 0.05 show TSMiner uniquely identifying more known LR-associated TFs than DREM (expected: 10 vs 1 at 0.01; 9 vs 4 at 0.05)
- Sensitivity curves span P-value cut-offs 0 to 1 for both methods, enabling direct curve comparison
- AUC scores are computed separately for the 27 cell-proliferation TFs and the 10 growth-suppression TFs at their designated LR periods
- All recovery counts, unique-TF counts, sensitivity values, and AUC scores are present in tsminer_drem_comparison_results.csv

## Limitations

- Fewer results are obtained from negative TF–TG interactions than from positive interactions because the known repressors of many genes were limited; negatively correlated TF–TG interaction data still need collection
- DREM depends on initial parameters without effective initialization and cannot determine when TFs exert influence on targets, which may bias baseline comparison results
- Sweeping only the subpaths parameter does not address other parameter sensitivities; precise regulatory mechanisms require more rigorous evaluation beyond computational prediction

## Evidence

- [readme] java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt: [evidence span withheld — anchor status: external_doc]
