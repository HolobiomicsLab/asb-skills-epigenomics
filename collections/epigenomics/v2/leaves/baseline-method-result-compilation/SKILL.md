---
name: baseline-method-result-compilation
description: Use when a paper claims a new model outperforms methods on shared benchmarks
  and the reproduction requires merging locally reproduced metrics with reported baseline
  values into one comparison table.
when_to_use_negative:
- The task requires newly re-running baseline methods rather than comparing reproduced
  results against the paper's reported baseline values
- Input is raw DNA FASTA sequence that has not yet been encoded into dna2vec feature
  vectors by get_datavec.py
- The goal is SE/TE prediction itself (use DeepSE prediction skills) rather than compiling
  a benchmark comparison of results
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3534
tools:
- name: DeepSE
  role: Two-layer CNN model being reproduced and benchmarked against state-of-the-art
    SE/TE discrimination methods
  repo: https://github.com/QiaoyingJi/DeepSE
- name: Keras
  role: Framework in which the two-layer CNN model is implemented and trained
- name: dna2vec
  role: Provides the pre-trained k-mer feature embeddings used to encode SE/TE DNA
    sequences into the feature vectors fed to the CNN
  repo: https://ngpatrick.com/dna2vec/
- name: Python
  role: Runtime environment (>=3.6) for executing the training, evaluation, and table-compilation
    scripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:9ffafcb0c600b9bafa5b9ec08d3f8f508a1dbd3b7dc8af22cb176225b42b3e17
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/QiaoyingJi/DeepSE
  repo_url: https://github.com/QiaoyingJi/DeepSE
  tools:
  - DeepSE
  - Keras
  - Python
  - dna2vec
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3534
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# baseline-method-result-compilation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce a proposed model's performance (here DeepSE, a two-layer Keras CNN distinguishing super-enhancers from typical enhancers from dna2vec sequence embeddings) and compile it side-by-side with reported state-of-the-art baseline metrics into a single benchmark comparison table, validating the paper's claim that DeepSE outperforms all baselines.

## When to use

Use when a paper claims a new model outperforms state-of-the-art methods on shared benchmarks and the reproduction requires merging locally reproduced metrics with reported baseline values into one comparison table.

## When NOT to use

- The task requires newly re-running baseline methods rather than comparing reproduced results against the paper's reported baseline values
- Input is raw DNA FASTA sequence that has not yet been encoded into dna2vec feature vectors by get_datavec.py
- The goal is SE/TE prediction itself (use DeepSE prediction skills) rather than compiling a benchmark comparison of results

## Inputs

- Encoded cell-line feature matrices in './data/<cell_name>' directories (mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87) produced by get_datavec.py from FASTA sequences using pre-trained dna2vec k-mer embeddings (hg19 or mm9)
- Trained DeepSE model checkpoints from train_specific.py and train_integrative1/2/3.py
- Reported state-of-the-art baseline performance values from the paper's reference results

## Outputs

- benchmark_comparison.csv: side-by-side table of reproduced DeepSE metrics and reported baseline metrics per cell line
- Training log files created by LogUtils.py

## How to apply

Load the encoded cell-line datasets (mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87) from './data/<cell_name>' produced by get_datavec.py. Train the DeepSE-specific two-layer Keras CNN per cell line with train_specific.py (e.g. 'python train_specific.py mESC 0.001 80') and the DeepSE-integrative models with train_integrative1.py, train_integrative2.py, and train_integrative3.py per species setting (hg or mm), using learning_rate = 0.001 and epochs = 80, recording logs via LogUtils.py. Evaluate the trained models on the shared cell-line test sets and record the classification metrics. Retrieve the reported state-of-the-art baseline values from the paper's reference results for the same cell-line benchmarks, then merge reproduced DeepSE metrics and reported baseline metrics into a single side-by-side table written to benchmark_comparison.csv. Validate that the compiled table shows DeepSE outperforming all state-of-the-art methods on each shared cell-line benchmark.

## Related tools

- **DeepSE** (Two-layer CNN model being reproduced and benchmarked against state-of-the-art SE/TE discrimination methods) — https://github.com/QiaoyingJi/DeepSE
- **Keras** (Framework in which the two-layer CNN model is implemented and trained)
- **dna2vec** (Provides the pre-trained k-mer feature embeddings used to encode SE/TE DNA sequences into the feature vectors fed to the CNN) — https://ngpatrick.com/dna2vec/
- **Python** (Runtime environment (>=3.6) for executing the training, evaluation, and table-compilation scripts)

## Examples

```
python train_specific.py mESC 0.001 80
```

## Evaluation signals

- The compiled benchmark_comparison.csv shows DeepSE outperforming all state-of-the-art methods on each shared cell-line benchmark, consistent with the paper's reported finding
- Training used the paper's hyperparameters: learning_rate = 0.001 and epochs = 80 for both specific and integrative models
- All eight cell lines (mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87) are represented in the comparison table
- Both DeepSE-specific and DeepSE-integrative (train_integrative1/2/3 with hg and mm species settings) results are reproduced
- Training console output is recorded in log files via LogUtils.py

## Limitations

- Baseline performance values are retrieved from the paper's reported results, not re-run locally, so the comparison depends on the baselines having used the same shared cell-line test sets
- Only two pre-trained embedding matrices are provided (human hg19 and mouse mm9); other genomes require training your own dna2vec model
- Reproduced metrics may vary with random initialization even at the paper's fixed hyperparameters (learning_rate = 0.001, epochs = 80)

## Evidence

- [abstract] With only the DNA sequence information, DeepSE outperformed all state-of-the-art methods: "With only the DNA sequence information, DeepSE outperformed all state-of-the-art methods"
- [methods] model.py: two-layer CNN in Keras: "model.py A two-layer CNN model was implemented by Keras."
- [readme] train_specific.py invocation: [evidence span withheld — anchor status: not_found]
- [methods] uploaded encoded cell data: "We have uploaded all the encoded cell data used in our experiments, including mESC_constituent, mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, and U87."
- [readme] pre-trained embedding matrices: "We have provided two pre-trained embedding matrixs, including human (version hg19) and mouse (mm9)."
