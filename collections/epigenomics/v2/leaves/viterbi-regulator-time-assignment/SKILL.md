---
name: viterbi-regulator-time-assignment
description: Use when a trained tree-structure model of a time-series expression matrix
  (e.g., mouse liver-regeneration RNA-seq) exists and each regulator must be assigned
  to the time point at which it influences its targets.
when_to_use_negative:
- The expression data are not time-series (no ordered time points), since bifurcation
  times and TF action times are defined by temporal fold-change comparisons between
  consecutive time points.
- No TF-gene interaction information is available, because the tree-structure model's
  branches and regulator assignments depend on integrating the time-series matrix
  with TF-TG interactions.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: TSMiner
  role: Java tool that trains the tree-structure model with Baum-Welch and runs Viterbi
    decoding to time TF activity; results are serialized to saved models validated
    against the 'Saved model' directory examples
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: DREM (v. 2.0)
  role: Predecessor method using a Baum-Welch-trained tree-structure model; cited
    as the limitation baseline TSMiner's Viterbi-based TF-time assignment improves
    upon
- name: Java
  role: Runtime language/environment for the TSMiner core algorithms and interfaces
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:4709d11953a0d6071fb5aade1d71a87a911444f1b604ab13360d0478dff13462
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
  - DREM (v. 2.0)
  - Java
  - TSMiner
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# viterbi-regulator-time-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run Viterbi decoding on TSMiner's trained tree-structure (input-output hidden Markov) model to assign each transcription factor to the specific time point at which it acts on its targets, turning a trained bifurcation model into time-specific TF-target assignments.

## When to use

Use when a trained tree-structure model of a time-series expression matrix (e.g., mouse liver-regeneration RNA-seq) exists and each regulator must be assigned to the time point at which it influences its targets.

## When NOT to use

- The expression data are not time-series (no ordered time points), since bifurcation times and TF action times are defined by temporal fold-change comparisons between consecutive time points.
- No TF-gene interaction information is available, because the tree-structure model's branches and regulator assignments depend on integrating the time-series matrix with TF-TG interactions.

## Inputs

- Time-series expression matrix (e.g., mouse LR RNA-seq time course, GEO accession GSE95135)
- TF-gene interaction file from the 'TF-gene file' directory
- Trained tree-structure (IOHMM) model with converged Baum-Welch likelihood

## Outputs

- Per-TF assignment to the specific time point at which it acts on its targets
- Viterbi-derived bifurcation structure of the tree-structure model
- Serialized saved-model file loadable in TSMiner

## How to apply

First train the tree-structure model: load the time-series expression data from the 'Time-series data' directory and the TF-gene interaction file, classify genes at each candidate bifurcation time point as up-, down-, or unchanged relative to the preceding time point, compute the three-dimensional comparison vector nc, and apply a likelihood ratio test to define candidate branching points. Initialize the Baum-Welch algorithm with the bifurcation patterns and iterate training until the likelihood score converges. Then run Viterbi decoding on the trained input-output hidden Markov model so every regulator (TF) is decoded to the specific time point at which it acts on its targets. This directly addresses a stated DREM limitation — DREM cannot determine when TFs exert influence on their targets — so the decoded TF-time assignments, not the bifurcation tree alone, are the deliverable. Serialize the result to a saved-model file and validate it by reloading in TSMiner.

## Related tools

- **TSMiner** (Java tool that trains the tree-structure model with Baum-Welch and runs Viterbi decoding to time TF activity; results are serialized to saved models validated against the 'Saved model' directory examples) — https://github.com/free1234hm/tsminer-tool.git
- **DREM (v. 2.0)** (Predecessor method using a Baum-Welch-trained tree-structure model; cited as the limitation baseline TSMiner's Viterbi-based TF-time assignment improves upon)
- **Java** (Runtime language/environment for the TSMiner core algorithms and interfaces)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- The Baum-Welch training loop is repeated until the likelihood score converges before decoding.
- The saved model reloads successfully in TSMiner.
- Viterbi-derived bifurcation structure and TF assignment times are consistent with the example saved models in the 'Saved model' directory.
- Each regulator is decoded to exactly one time point of action on its targets.
- On the mouse LR case analysis with Defaults_for_case_analysis.txt parameters, decoded TFs reproduce the reported activated/repressed TF counts (e.g., 389 activators and 49 repressors across the LR process).

## Limitations

- DREM-derived approaches depend on initial parameters without effective initialization, so decoding quality hinges on the bifurcation-pattern initialization step.
- Fewer results are obtained from negative TF-TG interactions than positive ones because known repressors are limited.
- Although experimental results supported predictions, the precise regulatory mechanisms require more rigorous evaluation.
- Applicability to other time-series omics data types (e.g., proteomics with PPI information) is not yet demonstrated.

## Evidence

- [other] Run Viterbi decoding on the trained model to assign each regulator (TF) to the specific time point at which it acts on its targets.: [evidence span withheld — anchor status: not_found]
- [intro] DREM limitation: cannot determine when TFs exert influence on targets: [evidence span withheld — anchor status: not_found]
- [other] Training loop with bifurcation initialization: [evidence span withheld — anchor status: not_found]
