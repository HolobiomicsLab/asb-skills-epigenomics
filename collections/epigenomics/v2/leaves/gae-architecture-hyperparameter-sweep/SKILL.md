---
name: gae-architecture-hyperparameter-sweep
description: Use when training a GAE on Hi-C contact matrices and you must choose
  network depth, hidden_dim, and output embedding dimension by comparing downstream
  TAD-calling quality (FMS/RI) across variants.
when_to_use_negative:
- When the goal is TAD calling on real data with an already-fixed, validated architecture
  rather than selecting one (use the optimal hidden_dim=64/output_dim=16 configuration
  directly)
- When the contact data are not resolvable into N-by-N intra-chromosomal matrices
  with a known ground-truth TAD annotation for scoring the sweep
- When a non-GAE TAD caller (Armatus, Arrowhead, TopDom, HiCSeg, TADtree, PSYCHIC,
  TADbit) is being tuned — its hyperparameters differ from GCN layer/embedding dimensions
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TOAST
  role: Method whose GAE module is swept over layer count and dimensions; provides
    toast.py and evaluation scripts
  repo: https://github.com/ghaiyan/TOAST
- name: PyTorch
  role: Framework for training the GAE variants (Adam optimizer, lr=0.001, 200 epochs,
    MSE reconstruction loss)
  repo: https://download.pytorch.org/whl/cpu
- name: HDBSCAN
  role: Unsupervised clustering of the 16–128 dimensional embeddings to label each
    genomic bin before TAD calling
- name: NumPy
  role: Handling of the NxN contact-matrix inputs and 400x400 submatrix partitioning
- name: Juicer
  role: juicer_tools.js dumps .hic files into the N-by-N matrix format used as GAE
    input
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:d4173919676e656f7b2fed9ada3fe424a27875333915f537278caa254d4158d0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ghaiyan/TOAST
  repo_url: https://github.com/ghaiyan/TOAST
  tools:
  - HDBSCAN
  - Juicer
  - NumPy
  - PyTorch
  - TOAST
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gae-architecture-hyperparameter-sweep

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sweep graph auto-encoder (GAE) architecture variants — number of GCN layers, hidden-layer dimension, and embedding output dimension — to find the configuration that yields the best and most robust TAD identification on simulated Hi-C contact matrices, as done in TOAST.

## When to use

Use when training a GAE on Hi-C contact matrices and you must choose network depth, hidden_dim, and output embedding dimension by comparing downstream TAD-calling quality (FMS/RI) across variants.

## When NOT to use

- When the goal is TAD calling on real data with an already-fixed, validated architecture rather than selecting one (use the optimal hidden_dim=64/output_dim=16 configuration directly)
- When the contact data are not resolvable into N-by-N intra-chromosomal matrices with a known ground-truth TAD annotation for scoring the sweep
- When a non-GAE TAD caller (Armatus, Arrowhead, TopDom, HiCSeg, TADtree, PSYCHIC, TADbit) is being tuned — its hyperparameters differ from GCN layer/embedding dimensions

## Inputs

- N-by-N tab-separated intra-chromosomal Hi-C contact matrices (dumped from .hic via juicer_tools)
- Trussart et al. toy-genome simulated Hi-C matrices at noise levels 50/100/150/200 and linear densities 40/75/150 bp/nm
- Non-overlapping 400x400 submatrices partitioned along the diagonal of each contact matrix

## Outputs

- Trained GAE node embeddings (16-, 64-, or 128-dimensional) per architecture variant
- HDBSCAN cluster labels per genomic bin and TAD calls (contiguous same-label regions > 200 Kb)
- Ranked architecture comparison table (e.g., gae_architecture_comparison.csv) with FMS and Rand Index per variant

## How to apply

Download the Trussart et al. toy-genome simulated Hi-C matrices (noise levels 50/100/150/200; linear densities 40/75/150 bp/nm), convert .hic files to N-by-N tab-separated intra-chromosomal matrices with Juicer tools, and partition each matrix along the diagonal into non-overlapping 400x400 submatrices as training inputs. Train each GAE variant (1 vs 2 GCN layers; embedding dimension in {16, 64, 128}; hidden-layer dimension in {64, 256}) with the Adam optimizer (lr=0.001) for 200 epochs under MSE reconstruction loss to obtain node embeddings. Cluster each embedding set with HDBSCAN, call TADs from contiguous same-label regions longer than 200 Kb, and score against ground-truth simulated TADs using Fowlkes-Mallows score (FMS) and Rand Index via evaulate_TAD-simnarity.py. Select the variant with the highest and most stable similarity scores; in the TOAST study the 2-layer network with hidden_dim=64 and output_dim=16 won, with a single-layer output_dim=128 network as the runner-up.

## Related tools

- **TOAST** (Method whose GAE module is swept over layer count and dimensions; provides toast.py and evaluation scripts) — https://github.com/ghaiyan/TOAST
- **PyTorch** (Framework for training the GAE variants (Adam optimizer, lr=0.001, 200 epochs, MSE reconstruction loss)) — https://download.pytorch.org/whl/cpu
- **HDBSCAN** (Unsupervised clustering of the 16–128 dimensional embeddings to label each genomic bin before TAD calling)
- **NumPy** (Handling of the NxN contact-matrix inputs and 400x400 submatrix partitioning)
- **Juicer** (juicer_tools.js dumps .hic files into the N-by-N matrix format used as GAE input)

## Examples

```
python src/toast.py  # set input/output folders, then: python src/evaulate_TAD-simnarity.py
```

## Evaluation signals

- The 2-layer GAE with hidden_dim=64 and output_dim=16 achieves the highest FMS/Rand Index similarity to ground-truth simulated TADs across the six-variant sweep
- FMS on Wang et al. simulated data remains around 0.98 despite decreasing with noise level
- Number-of-TADs difference versus ground truth stays within 1–8 and average-TAD-length difference within ~5 Kb
- evaulate_TAD-simnarity.py successfully compares called TADs to the simulated genomes' ground-truth TADs
- Only contiguous same-label regions exceeding 200 Kb are emitted as TADs (length-filter invariant)

## Limitations

- The sweep was tuned on simulated Hi-C data (Trussart et al. toy genomes; Wang et al. IMR-90); the optimum may shift for real GM12878 data at 50 Kb, 25 Kb, or 5 Kb resolution
- A one-layer network with output_dim=128 performed comparably, so the hidden_dim=64/output_dim=16 choice was justified by robustness, not strictly by a single best score
- Requires a known ground-truth TAD annotation (or reference method output) to compute FMS/RI; cannot be evaluated purely from reconstruction loss

## Evidence
