---
name: pearson-correlation-model-comparison
description: Use when comparing CNN, HyenaDNA, or Caduceus variants against an expert
  model on base-pair-resolution regression of transcription initiation signals over
  100,000-bp sequences in DNALONGBENCH.
when_to_use_negative:
- Task requires binary classification metrics such as AUROC (e.g., enhancer-target
  gene or eQTL tasks in DNALONGBENCH) rather than nucleotide-wise regression correlation
- Input sequences are short-range; this skill targets long-range dependencies up to
  1 million bp and the 100,000-bp TISP setting
- Evaluating models not among the five benchmarked methods (expert, CNN, HyenaDNA,
  Caduceus-Ph, Caduceus-PS) without adapting the per-assay PCC aggregation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3474
tools:
- name: HyenaDNA
  role: Fine-tuned DNA foundation model baseline evaluated on TISP via the HazyResearch/hyena-dna
    codebase
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: Fine-tuned DNA foundation model baseline run inside the caduceus_env environment
    per experiments/Caduceus_RSAP_TISP/README.md
- name: Caduceus-PS
  role: Fine-tuned DNA foundation model baseline run inside the caduceus_env environment
    per experiments/Caduceus_RSAP_TISP/README.md
- name: conda
  role: Environment manager for dnalongbench (Python 3.9) and caduceus_env used to
    train and run inference
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:be9900388df5b19642683949bff03f3068e4630f3db79bd63f92401569095f22
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
  - Caduceus-PS
  - Caduceus-Ph
  - HyenaDNA
  - conda
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pearson-correlation-model-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark protocol for comparing a task-specific expert model (Puffin) against a CNN baseline and fine-tuned DNA foundation models (HyenaDNA, Caduceus-Ph, Caduceus-PS) on nucleotide-wise transcription initiation signal prediction (TISP) using per-assay Pearson correlation (PCC) on held-out test chromosomes.

## When to use

Use when comparing CNN, HyenaDNA, or Caduceus variants against an expert model on base-pair-resolution regression of transcription initiation signals over 100,000-bp sequences in DNALONGBENCH.

## When NOT to use

- Task requires binary classification metrics such as AUROC (e.g., enhancer-target gene or eQTL tasks in DNALONGBENCH) rather than nucleotide-wise regression correlation
- Input sequences are short-range; this skill targets long-range dependencies up to 1 million bp and the 100,000-bp TISP setting
- Evaluating models not among the five benchmarked methods (expert, CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS) without adapting the per-assay PCC aggregation

## Inputs

- TISP bed files (transcription initiation signal prediction) from Dataverse deposit doi:10.7910/DVN/VXQKWO
- TISP train/valid/test splits loaded via dnalongbench load_data with the transcription initiation signal task name
- 100,000-bp genomic DNA sequences with 10 channel-wise signal outputs
- Held-out test chromosomes chr8 and chr9

## Outputs

- Per-assay nucleotide-wise Pearson correlation (PCC) values on test chromosomes chr8 and chr9
- Average PCC per model for Puffin, CNN, HyenaDNA, Caduceus-Ph, and Caduceus-PS
- Model comparison table written to tisp_pcc_results.csv

## How to apply

Set up the dnalongbench conda environment (Python 3.9), clone ma-compbio/DNALONGBENCH, install with pip, and download the TISP bed files from the Dataverse deposit (https://doi.org/10.7910/DVN/VXQKWO). Load the TISP train/valid/test splits with the dnalongbench load_data utility, keeping chr8 and chr9 as held-out test chromosomes. Train and run inference with the CNN baseline (10 channel-wise outputs on 100,000-bp sequences) per experiments/CNN/README.md, fine-tune HyenaDNA per experiments/HyenaDNA_RSAP_TISP/README.md, and fine-tune Caduceus-Ph and Caduceus-PS inside the caduceus_env environment per experiments/Caduceus_RSAP_TISP/README.md. Compute nucleotide-wise Pearson correlation per assay on the test chromosomes, aggregate to an average per model, and write the five-model comparison table to tisp_pcc_results.csv. The expected outcome is that the expert model Puffin substantially outperforms the foundation models (PCC 0.733 vs 0.042–0.132).

## Related tools

- **HyenaDNA** (Fine-tuned DNA foundation model baseline evaluated on TISP via the HazyResearch/hyena-dna codebase) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (Fine-tuned DNA foundation model baseline run inside the caduceus_env environment per experiments/Caduceus_RSAP_TISP/README.md)
- **Caduceus-PS** (Fine-tuned DNA foundation model baseline run inside the caduceus_env environment per experiments/Caduceus_RSAP_TISP/README.md)
- **conda** (Environment manager for dnalongbench (Python 3.9) and caduceus_env used to train and run inference)

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='transcription_initiation_signal_prediction', subset='TISP', batch_size=16)
```

## Evaluation signals

- tisp_pcc_results.csv contains a PCC value for each of the five models (Puffin, CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS)
- Per-assay and average PCC entries are both present in the output table
- Relative ordering matches the paper's finding: Puffin (0.733) > HyenaDNA (0.132) > Caduceus-Ph (0.109) > Caduceus-PS (0.108) > CNN (0.042)
- PCC is computed nucleotide-wise on held-out test chromosomes chr8 and chr9, not on train/valid splits
- Foundation model predictions should be inspected for underpredicted signal intensities or missed peaks relative to ground truth

## Limitations

- DNA foundation models underpredict signal intensities or miss peaks on the TISP task, so fine-tuned scores (PCC 0.042–0.132) are far below the expert model
- Simple fine-tuning may not be sufficient to outperform highly specialized expert architectures on long-range tasks
- The benchmark's TISP evaluation uses 100,000-bp sequences and 10 channel-wise outputs; other resolutions or output shapes require protocol adaptation

## Evidence

- [methods] TISP benchmark finding: [evidence span withheld — anchor status: not_found]
- [methods] Workflow step 7: [evidence span withheld — anchor status: not_found]
