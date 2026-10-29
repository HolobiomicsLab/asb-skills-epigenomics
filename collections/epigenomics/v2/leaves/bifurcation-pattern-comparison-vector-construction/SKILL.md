---
name: bifurcation-pattern-comparison-vector-construction
description: Use when a time-series expression matrix must be split at candidate bifurcation
  time points and Baum-Welch training of a tree-structure model needs effective initial
  parameters, e.g., TSMiner runs on mouse liver regeneration RNA-seq time courses.
when_to_use_negative:
- The data are not a time course (no ordered time points with a preceding reference),
  so the fold-change-based up/down/unchanged classification and comparison vector
  nc cannot be computed.
- A trained TSMiner saved model already exists and the task is only Viterbi decoding
  or interactive network visualization - no new bifurcation patterns are needed.
- The goal is a static regulatory network or plain clustering of non-temporal expression
  data; this skill's branching logic is specific to time-series expression profiles
  plus TF-TG interaction files.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0602
tools:
- name: TSMiner
  role: 'Implements the full bifurcation-pattern workflow: fold-change classification,
    comparison vector nc computation, likelihood-ratio partitioning of G, path division
    and Baum-Welch initialization of the input-output HMM; also validates saved models
    via Viterbi structure.'
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime for TSMiner's core bifurcation-pattern and Baum-Welch algorithms (TSMiner.jar
    requires Java 1.5+ and 1024 MB heap memory).
  repo: http://www.java.com
- name: DREM
  role: Predecessor that trains a time-series expression matrix into a tree-structure
    model with Baum-Welch but lacks effective initialization; the bifurcation-pattern
    step in TSMiner addresses this gap.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:2b8476805537db83282f601d66d58e9f9bfe39390d25452c189c09ea18b91207
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
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# bifurcation-pattern-comparison-vector-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs the bifurcation patterns TSMiner uses to divide each path of its tree-structure input-output hidden Markov model: at candidate bifurcation time points, genes are classified as up-regulated, down-regulated, or unchanged by expression fold change and summarized into a three-dimensional comparison vector nc. A likelihood ratio test then partitions the gene set G to define branch points and supply effective initial parameters for Baum-Welch training.

## When to use

Use when a time-series expression matrix must be split at candidate bifurcation time points and Baum-Welch training of a tree-structure model needs effective initial parameters, e.g., TSMiner runs on mouse liver regeneration RNA-seq time courses.

## When NOT to use

- The data are not a time course (no ordered time points with a preceding reference), so the fold-change-based up/down/unchanged classification and comparison vector nc cannot be computed.
- A trained TSMiner saved model already exists and the task is only Viterbi decoding or interactive network visualization - no new bifurcation patterns are needed.
- The goal is a static regulatory network or plain clustering of non-temporal expression data; this skill's branching logic is specific to time-series expression profiles plus TF-TG interaction files.

## Inputs

- Time-series expression matrix (e.g., mouse liver regeneration RNA-seq time course; example sets in the 'Time-series data' directory)
- TF-gene interaction file (example data in the 'TF-gene file' directory)
- Parameter settings file (Defaults_for_case_analysis.txt; details in TSMiner manual.pdf)

## Outputs

- Per-gene three-dimensional comparison vector nc (counts of up-regulated, down-regulated, unchanged over subsequent time points)
- Likelihood-ratio-test partition of the gene set G into branch groups, defining candidate bifurcation (branching) points of the tree-structure model
- Submatrices of the time-series expression matrix divided by bifurcation pattern, with initial parameters for each division fed to Baum-Welch training
- Trained tree-structure (input-output HMM) model serialized to a saved-model file, with Viterbi-derived TF assignment times

## How to apply

Load the time-series expression matrix and TF-gene interaction file (TSMiner ships examples in the 'Time-series data' and 'TF-gene file' directories) and set parameters from Defaults_for_case_analysis.txt / TSMiner manual.pdf. At every candidate bifurcation time point, classify each gene as up-regulated, down-regulated, or unchanged by its expression fold change relative to the preceding time point. For each gene, compute the three-dimensional comparison vector nc that summarizes up/down/unchanged counts over the subsequent time points. Apply a likelihood ratio test to these vectors to partition the gene set G into groups, defining the candidate split (branching) points of the tree-structure model. Divide each path in the current model using the complete set of bifurcation patterns, calculate initial parameters for each division, then train the input-output hidden Markov model with the Baum-Welch algorithm, repeating until the likelihood score converges - this provides the effective initialization that DREM lacks. Check correctness by running Viterbi decoding on the trained model and confirming its bifurcation structure and TF-acting time points are consistent with the example saved models in the 'Saved model' directory.

## Related tools

- **TSMiner** (Implements the full bifurcation-pattern workflow: fold-change classification, comparison vector nc computation, likelihood-ratio partitioning of G, path division and Baum-Welch initialization of the input-output HMM; also validates saved models via Viterbi structure.) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime for TSMiner's core bifurcation-pattern and Baum-Welch algorithms (TSMiner.jar requires Java 1.5+ and 1024 MB heap memory).) — http://www.java.com
- **DREM** (Predecessor that trains a time-series expression matrix into a tree-structure model with Baum-Welch but lacks effective initialization; the bifurcation-pattern step in TSMiner addresses this gap.)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- Every gene at every candidate bifurcation time point receives exactly one of three states (up-regulated, down-regulated, unchanged), and each gene's comparison vector nc entries count states over the subsequent time points.
- The likelihood ratio test partitions the gene set G so each gene is assigned to exactly one branch group per candidate split, yielding a complete set of bifurcation patterns for every path.
- Baum-Welch training initialized with the bifurcation-pattern-derived parameters runs iteratively until the likelihood score converges (no further increase across repetitions).
- Viterbi decoding on the trained model assigns each TF to a specific acting time point on its targets.
- The serialized model loads in TSMiner and its Viterbi-derived bifurcation structure and TF assignment times are consistent with the example saved models in the 'Saved model' directory.

## Limitations

- Validated on time-series expression profiles (mouse liver regeneration RNA-seq); applicability to other omics types, such as proteomics time-series data with PPI information, was not demonstrated and is left as future research.
- Baum-Welch training depends on initial parameters - the very limitation TSMiner fixes relative to DREM - so poorly separated fold-change patterns can yield ambiguous branching or slow convergence.
- Fewer results arise from negative TF-TG interactions than positive ones because known repressors are limited, constraining repressor-side branches of the inferred structure.

## Evidence

- [intro] DREM limitation motivating effective initialization: "However, DREM and its extended methods have three limitations. First, DREM uses a version of the Baum–Welch algorithm to train the parameters of tree-structure models."
- [full_text] Fold-change classification at bifurcation points: [evidence span withheld — anchor status: not_found]
- [full_text] Three-dimensional comparison vector nc: [evidence span withheld — anchor status: not_found]
- [methods] Parameter settings reference: "The data input and parameter settings are described in TSMiner manual.pdf."
- [readme] Example time-series input data location: "Example time-series expression data sets are included in 'Time-series data' directory."
- [readme] Command-line case analysis run: "Otherwise from a command line while in the TSMiner directory type the command: java -mx1024M -jar TSMiner.jar -d Case\_analysis\_of\_mouseLR.txt, then press the 'Run' button."
