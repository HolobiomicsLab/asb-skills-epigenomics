---
name: baum-welch-hidden-markov-model-training
description: Use when a time-series expression matrix must be segmented into bifurcating
  submatrices and an input-output HMM tree-structure model needs parameter initialization
  and Baum–Welch training, e.g. to assign TFs to the time points at which they act.
when_to_use_negative:
- The data are not time-series expression profiles and no candidate bifurcation (split)
  time points can be defined from fold changes between consecutive time points
- You only need static clustering of expression profiles without temporal dynamics
  of regulators — the article notes clustering methods cannot evaluate the temporal
  dynamics of regulators or pathways, but conversely this skill is not a substitute
  for simple module clustering
- You cannot supply a TF-gene interaction file, since the input-output HMM of the
  tree-structure model is trained jointly on the expression matrix and TF–TG interaction
  information
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0204
tools:
- name: TSMiner
  role: Implements the bifurcation-pattern division and Baum–Welch training of the
    input-output hidden Markov model of the tree structure, plus Viterbi decoding;
    written in Java (v1.2.0 used in the paper).
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime language/environment for TSMiner's core algorithms and interfaces;
    requires Java 1.5 or later and a minimum 1024 MB Java heap.
- name: DREM
  role: Prior tree-structure model trainer using a version of the Baum–Welch algorithm;
    TSMiner improves on its lack of effective initialization of initial parameters.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:64322d2db632c94ba01556466bd4ed5833becd9538e532848f71041c492ca41e
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
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# baum-welch-hidden-markov-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains the input-output hidden Markov model of a tree-structure model on a time-series expression matrix by initializing Baum–Welch with bifurcation-pattern-derived parameters and iterating until the likelihood score converges, as implemented in TSMiner (Java) for time-specific gene regulatory network construction.

## When to use

Use when a time-series expression matrix must be segmented into bifurcating submatrices and an input-output HMM tree-structure model needs parameter initialization and Baum–Welch training, e.g. to assign TFs to the time points at which they act.

## When NOT to use

- The data are not time-series expression profiles and no candidate bifurcation (split) time points can be defined from fold changes between consecutive time points
- You only need static clustering of expression profiles without temporal dynamics of regulators — the article notes clustering methods cannot evaluate the temporal dynamics of regulators or pathways, but conversely this skill is not a substitute for simple module clustering
- You cannot supply a TF-gene interaction file, since the input-output HMM of the tree-structure model is trained jointly on the expression matrix and TF–TG interaction information

## Inputs

- Time-series expression matrix (e.g. mouse liver regeneration RNA-seq dataset, GSE95135) from the 'Time-series data' directory
- TF-gene interaction file from the 'TF-gene file' directory
- Parameter settings file Defaults_for_case_analysis.txt
- Optional: gene annotation files (GO, KEGG/Reactome pathway files) from the 'Gene annotation file' directory

## Outputs

- Trained tree-structure (input-output HMM) model serialized to a saved-model file (example saved models in the 'Saved model' directory)
- Viterbi-derived bifurcation structure of the time-series expression matrix
- Assignment of each TF to the specific time point at which it acts on its targets

## How to apply

Load the time-series expression data from the 'Time-series data' directory and the TF-gene interaction file from the 'TF-gene file' directory, with parameters from Defaults_for_case_analysis.txt. Classify each gene at every candidate bifurcation time point as up-regulated, down-regulated, or unchanged relative to the preceding time point, and compute the three-dimensional comparison vector nc of up/down/unchanged counts; apply a likelihood ratio test to partition the gene set G and define candidate split (branching) points. TSMiner divides each path in the current model based on a complete set of bifurcation patterns and calculates initial parameters for each division — this initialization is the key improvement over DREM, which uses a version of the Baum–Welch algorithm without effective initialization. Run the Baum–Welch training of the input-output hidden Markov model, repeating until the likelihood score converges. Then run Viterbi decoding on the trained model to assign each regulator (TF) to the specific time point at which it acts on its targets, and serialize the trained tree-structure model to a saved-model file.

## Related tools

- **TSMiner** (Implements the bifurcation-pattern division and Baum–Welch training of the input-output hidden Markov model of the tree structure, plus Viterbi decoding; written in Java (v1.2.0 used in the paper).) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime language/environment for TSMiner's core algorithms and interfaces; requires Java 1.5 or later and a minimum 1024 MB Java heap.)
- **DREM** (Prior tree-structure model trainer using a version of the Baum–Welch algorithm; TSMiner improves on its lack of effective initialization of initial parameters.)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- The Baum–Welch training loop is run until the likelihood score converges (likelihood no longer increases between iterations).
- The trained model loads successfully in TSMiner (acceptance criterion from the source card validation step).
- The Viterbi-derived bifurcation structure of the trained model is consistent with the example saved models provided in the 'Saved model' directory.
- Viterbi decoding assigns each TF to a specific time point at which it acts on its targets, and these TF assignment times are consistent with the example saved models.
- Each path division corresponds to a complete set of bifurcation patterns with initial parameters computed for each division before training.

## Limitations

- DREM-style Baum–Welch training depends on initial parameters without effective initialization; TSMiner addresses this via bifurcation-pattern-based initialization, but results still depend on the candidate bifurcation time points defined by the likelihood ratio test.
- Requires Java 1.5 or later and a minimum of 1024 MB Java heap memory, which may constrain very large time-series matrices.
- Fewer results are obtained from negative TF–TG interactions than from positive interactions because known repressors of many genes are limited, affecting the interaction data fed to the model.

## Evidence

- [methods] Bifurcation-pattern initialization and Baum–Welch convergence: [evidence span withheld — anchor status: not_found]
- [intro] DREM limitation motivating the initialization step: "DREM uses a version of the Baum–Welch algorithm to train the parameters of tree-structure models."
- [methods] Viterbi decoding of the trained model: [evidence span withheld — anchor status: not_found]
- [methods] Implementation language: "The core algorithms and interfaces of TSMiner were written in java language."
- [readme] Java runtime requirements: "Java 1.5 or later must be installed.  [Download link](http://www.java.com) - 1024 MB Java heap memory minimum."
