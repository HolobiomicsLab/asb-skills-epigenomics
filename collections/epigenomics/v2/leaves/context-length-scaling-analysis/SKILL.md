---
name: context-length-scaling-analysis
description: Use when a long-range genomics task (e.g., contact map prediction with
  100–200 bins) requires quantifying whether model performance improves monotonically
  as input context length grows, and whether it still trails the task-specific expert
  model.
when_to_use_negative:
- The task involves short-range dependencies or input sequences far below ~204,800
  bp, where context-length scaling is not the relevant variable
- Transformer-based DNA foundation models (DNABERT-1/2, Nucleotide Transformer) are
  the intended subjects, since quadratic self-attention cost renders these ultra-long-range
  tasks infeasible for them
- The goal is to compare foundation models against each other on short-range benchmarks
  (e.g., Genomic Benchmarks, GUE, NT Benchmark) rather than evaluate long-context
  scaling
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Caduceus-Ph
  role: Fine-tuned DNA foundation model evaluated at three context lengths on the
    contact map prediction task
- name: HyenaDNA
  role: Long-range genomic foundation model family (up to 1 million tokens, single
    nucleotide resolution) whose architecture underpins the long-context evaluation
  repo: https://github.com/HazyResearch/hyena-dna
- name: DNALONGBENCH
  role: Benchmark suite providing the contact map prediction task, data loaders, and
    split files used for the ablation
  repo: https://github.com/ma-compbio/DNALONGBENCH
- name: conda
  role: Environment manager used to create the dnalongbench and Caduceus environments
    (caduceus_env.yml)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:ee4f546445b5bcb65888e7d45cef9a955d3a00c6a90082677acbfbe8e68bc584
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ma-compbio/DNALONGBENCH
  repo_url: https://github.com/HazyResearch/hyena-dna
  tools:
  - Caduceus-Ph
  - DNALONGBENCH
  - HyenaDNA
  - conda
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# context-length-scaling-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for testing whether a long-range DNA foundation model's performance scales with input context length, by fine-tuning and evaluating the model (e.g., Caduceus-Ph on DNALONGBENCH contact map prediction) at multiple sequence lengths (204,800, 307,200, and 409,600 bp) and comparing the resulting Spearman correlation coefficients (SCC).

## When to use

Use when a long-range genomics task (e.g., contact map prediction with 100–200 bins) requires quantifying whether model performance improves monotonically as input context length grows, and whether it still trails the task-specific expert model.

## When NOT to use

- The task involves short-range dependencies or input sequences far below ~204,800 bp, where context-length scaling is not the relevant variable
- Transformer-based DNA foundation models (DNABERT-1/2, Nucleotide Transformer) are the intended subjects, since quadratic self-attention cost renders these ultra-long-range tasks infeasible for them
- The goal is to compare foundation models against each other on short-range benchmarks (e.g., Genomic Benchmarks, GUE, NT Benchmark) rather than evaluate long-context scaling

## Inputs

- Contact Map Prediction dataset (doi:10.7910/DVN/AZM25S) with pre-split train/valid/test files, HFF subset
- Caduceus-Ph pretrained model and caduceus_env.yml environment spec from experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP
- Context-length configurations: 409,600 bp (200 bins), 307,200 bp (150 bins), 204,800 bp (100 bins)

## Outputs

- caduceus_ph_cmp_scc_scores.csv containing an SCC value for each of the three input context lengths, ordered by increasing context length
- Predicted contact maps on the test split for each context length

## How to apply

Create and activate the Caduceus conda environment from caduceus_env.yml in experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP. Download the Contact Map Prediction data (doi:10.7910/DVN/AZM25S) and load it with the dnalongbench load_data utility using task_name='contact_map_prediction', subset='HFF', batch_size=16. Following experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP/README.md, fine-tune Caduceus-Ph and run inference at each of the three context lengths: 409,600 bp (200 bins), 307,200 bp (150 bins), and 204,800 bp (100 bins). Compute the SCC between predicted and ground-truth contact maps on the test split for each length, assemble the three scores into a single table ordered by increasing context length, and save it as caduceus_ph_cmp_scc_scores.csv. The rationale is to verify the paper's reported trend that SCC increases monotonically as context grows, while remaining below the expert model's score.

## Related tools

- **Caduceus-Ph** (Fine-tuned DNA foundation model evaluated at three context lengths on the contact map prediction task)
- **HyenaDNA** (Long-range genomic foundation model family (up to 1 million tokens, single nucleotide resolution) whose architecture underpins the long-context evaluation) — https://github.com/HazyResearch/hyena-dna
- **DNALONGBENCH** (Benchmark suite providing the contact map prediction task, data loaders, and split files used for the ablation) — https://github.com/ma-compbio/DNALONGBENCH
- **conda** (Environment manager used to create the dnalongbench and Caduceus environments (caduceus_env.yml))

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='contact_map_prediction', subset='HFF', batch_size=16)
```

## Evaluation signals

- caduceus_ph_cmp_scc_scores.csv contains exactly three SCC values, one for each input size (409,600, 307,200, and 204,800 bp)
- SCC increases monotonically as input context length grows from 204,800 to 409,600 bp, reproducing the trend in Tables S6–S11
- All Caduceus-Ph SCC scores remain below the task-specific expert model's score on the same task
- Data loading succeeds via the dnalongbench load_data utility with task_name='contact_map_prediction', subset='HFF', batch_size=16, yielding train/valid/test loaders
- SCC is computed on the test split only, between predicted and ground-truth contact maps

## Limitations

- Expert models consistently outperform DNA foundation models across all tasks, so scaling context length is not expected to close the gap entirely
- Transformer-based DNA foundation models are infeasible for this ablation because the quadratic cost of self-attention renders ultra-long-range tasks infeasible
- Simple fine-tuning may not be sufficient for foundation models to outperform highly specialized expert architectures
- Model performance varies substantially across tasks, so trends observed on contact map prediction may not transfer to the other DNALONGBENCH tasks

## Evidence
