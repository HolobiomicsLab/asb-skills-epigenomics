---
name: spearman-correlation-metric-computation
description: Use when scoring predicted 3D genome contact maps from models such as
  Caduceus-Ph or HyenaDNA against observed maps on the DNALONGBENCH contact map prediction
  task, or when quantifying agreement (SCC, alongside PCC) for specific genomic regions
  like chr6:145,205,248-145,614,848.
when_to_use_negative:
- Task output is binary classification (enhancer-target gene or eQTL tasks), which
  the benchmark scores with AUROC rather than SCC
- Input is not a 2D contact map / structure-related prediction — SCC contact-map agreement
  is specific to the 3D genome organization task
- You need a training-time loss rather than an evaluation metric; SCC here is a post-inference
  test-split evaluation
tools:
- name: DNALONGBENCH
  role: Provides the contact map prediction task data loaders and defines SCC as the
    task metric
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: Caduceus-Ph
  role: Fine-tuned foundation model whose predicted contact maps are scored with SCC
    across context-length ablations
- name: HyenaDNA
  role: Fine-tuned foundation model whose contact map predictions are scored with
    SCC/PCC in the case study
  repo: https://github.com/HazyResearch/hyena-dna.git
- name: Akita
  role: Task-specific expert model whose SCC scores serve as the upper reference in
    comparisons
- name: conda
  role: Environment management for the dnalongbench (Python 3.9) and Caduceus environments
    used in the evaluation
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:2d22ba25e11b99289678133b6d5fa3dcc1ada7ae8115d8b69cd03c02d288da18
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ma-compbio/DNALONGBENCH
  repo_url: https://github.com/ma-compbio/DNALONGBENCH.git
  tools:
  - Akita
  - Caduceus-Ph
  - DNALONGBENCH
  - HyenaDNA
  - conda
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# spearman-correlation-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the Spearman correlation coefficient (SCC) between predicted and ground-truth contact maps on the DNALONGBENCH contact map prediction (HFF subset) test split, the benchmark's reported metric for evaluating how well DNA foundation models capture long-range 3D genomic dependencies.

## When to use

Use when scoring predicted 3D genome contact maps from models such as Caduceus-Ph or HyenaDNA against observed maps on the DNALONGBENCH contact map prediction task, or when quantifying agreement (SCC, alongside PCC) for specific genomic regions like chr6:145,205,248-145,614,848.

## When NOT to use

- Task output is binary classification (enhancer-target gene or eQTL tasks), which the benchmark scores with AUROC rather than SCC
- Input is not a 2D contact map / structure-related prediction — SCC contact-map agreement is specific to the 3D genome organization task
- You need a training-time loss rather than an evaluation metric; SCC here is a post-inference test-split evaluation

## Inputs

- Predicted contact maps from a fine-tuned DNA foundation model (HyenaDNA, Caduceus-Ph) or the Akita expert model on the contact map prediction task
- Ground-truth contact maps from the DNALONGBENCH Contact Map Prediction dataset (doi:10.7910/DVN/AZM25S), HFF subset, held-out test split
- Region specifications for case studies (e.g., chr6:145,205,248-145,614,848 and chr3:139,341,824-139,751,424)

## Outputs

- SCC value(s) between predicted and observed contact maps on the test split
- caduceus_ph_cmp_scc_scores.csv with SCC scores at 204,800 / 307,200 / 409,600 bp input lengths
- Annotated figure panels with PCC/SCC values (figure_2_contact_maps.png)

## How to apply

Load the held-out test split of the Contact Map Prediction dataset (doi:10.7910/DVN/AZM25S) via dnalongbench.utils.load_data with task_name='contact_map_prediction', subset='HFF', batch_size=16, then run model inference (e.g., fine-tuned Caduceus-Ph per experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP/README.md, or HyenaDNA per experiments/HyenaDNA_ETGP_CMP_eQTLP/README.md) to obtain predicted maps. Compute the Spearman correlation coefficient between each predicted map and the ground-truth map on the test split; for the context-length ablation, repeat this at 409,600 bp (200 bins), 307,200 bp (150 bins), and 204,800 bp (100 bins) and assemble the three SCC values ordered by increasing context length into caduceus_ph_cmp_scc_scores.csv. For case-study figures, compute both Pearson (PCC) and Spearman (SCC) per region and annotate each figure panel (e.g., figure_2_contact_maps.png rendered with matplotlib). Decision point: SCC is the correct metric here because the task output is a 2D contact map at base-pair resolution, not a binary classification label scored by AUROC.

## Related tools

- **DNALONGBENCH** (Provides the contact map prediction task data loaders and defines SCC as the task metric) — https://github.com/ma-compbio/DNALONGBENCH.git
- **Caduceus-Ph** (Fine-tuned foundation model whose predicted contact maps are scored with SCC across context-length ablations)
- **HyenaDNA** (Fine-tuned foundation model whose contact map predictions are scored with SCC/PCC in the case study) — https://github.com/HazyResearch/hyena-dna.git
- **Akita** (Task-specific expert model whose SCC scores serve as the upper reference in comparisons)
- **conda** (Environment management for the dnalongbench (Python 3.9) and Caduceus environments used in the evaluation)

## Examples

```
from dnalongbench.utils import load_data; train_loader, valid_loader, test_loader = load_data(root=root, task_name='contact_map_prediction', subset='HFF', batch_size=16)
```

## Evaluation signals

- The SCC table contains exactly one SCC value per input size (204,800, 307,200, and 409,600 bp / 100, 150, 200 bins)
- SCC increases monotonically as input context length grows from 204,800 to 409,600 bp, reproducing the trend in Tables S6-S11
- Caduceus-Ph SCC remains below the expert model's (Akita) score on the same task
- Case-study figure annotates PCC and SCC beneath each of the four panels, with the expert model attaining the highest agreement scores
- Metric is computed on the test split loaded via load_data(task_name='contact_map_prediction', subset='HFF') rather than train/valid splits

## Limitations

- DNA foundation models show limited capacity to predict domain structures; SCC values are expected to lag the expert model, so high SCC should not be assumed
- Foundation model performance varies substantially across tasks, so SCC behavior on contact maps may not transfer to the other four DNALONGBENCH tasks
- Simple fine-tuning may be insufficient for foundation models to outperform specialized expert architectures on long-range tasks

## Evidence
