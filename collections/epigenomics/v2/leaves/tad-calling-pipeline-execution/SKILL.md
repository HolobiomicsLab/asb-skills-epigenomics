---
name: tad-calling-pipeline-execution
description: Use when NxN KR-normalized intra-chromosomal Hi-C contact matrices (e.g.,
  GM12878 at 5 Kb, 25 Kb, or 50 Kb resolution dumped from .hic files) must be segmented
  into topologically associated domains.
when_to_use_negative:
- Input is raw .hic/.cool data not yet dumped to an NxN intra-chromosomal matrix —
  dump with Juicer tools first
- The goal is enhancer–promoter interaction prediction (that is EPIANN's scope, not
  TAD calling)
- A method requiring configurable resolution/scale parameters is preferred (TOAST
  competitors like Armatus offer multiscale modeling; TOAST uses fixed 400x400 partitioning
  and 200 Kb minimum TAD length)
edam_operation: http://edamontology.org/operation_3903
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: TOAST
  role: 'Main pipeline (toast.py): GAE training on 400x400 submatrices, HDBSCAN clustering,
    TAD generation'
  repo: https://github.com/ghaiyan/TOAST
- name: HDBSCAN
  role: Unsupervised clustering that assigns labels to each genomic bin's 16-dimensional
    GAE embedding
- name: PyTorch
  role: Implements and trains the 2-layer graph auto-encoder (hidden dim 64, embedding
    dim 16, Adam, 200 epochs)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:9771956959ebca8144ea308d5233b75d4900681eb2da6454b77be63914e10207
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
  - PyTorch
  - TOAST
  edam_operation: http://edamontology.org/operation_3903
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# tad-calling-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Execute the TOAST TAD-calling pipeline, which represents genomic bins as graph nodes with the Hi-C contact matrix as the adjacency matrix, learns 16-dimensional embeddings with a 2-layer graph auto-encoder, clusters them with HDBSCAN, and delineates TADs as contiguous same-label regions longer than 200 Kb.

## When to use

Use when NxN KR-normalized intra-chromosomal Hi-C contact matrices (e.g., GM12878 at 5 Kb, 25 Kb, or 50 Kb resolution dumped from .hic files) must be segmented into topologically associated domains.

## When NOT to use

- Input is raw .hic/.cool data not yet dumped to an NxN intra-chromosomal matrix — dump with Juicer tools first
- The goal is enhancer–promoter interaction prediction (that is EPIANN's scope, not TAD calling)
- A method requiring configurable resolution/scale parameters is preferred (TOAST competitors like Armatus offer multiscale modeling; TOAST uses fixed 400x400 partitioning and 200 Kb minimum TAD length)

## Inputs

- NxN tab-separated KR-normalized intra-chromosomal Hi-C contact matrix (from juicer_tools dump of a .hic file)
- GM12878 .hic file (4DNFI1UEG1HD, GEO GSE63525) as raw source
- Simulated Hi-C contact matrices (Forcato et al., Wang et al., Trussart et al.) for validation

## Outputs

- TAD file at the merge file per chromosome (TAD boundaries with coordinates)
- TAD heatmap plots from plot_heatmaps.py
- TAD evaluation metrics (e.g., FMS) from evaluate_TAD.py

## How to apply

Dump KR-normalized NxN intra-chromosomal matrices from the GM12878 .hic file (4DNFI1UEG1HD, GEO GSE63525) using Juicer tools at 5 Kb, 25 Kb, or 50 Kb resolution. Run toast.py, which partitions each contact matrix into non-overlapping 400x400 submatrices along the diagonal, trains a 2-layer GAE (hidden dimension 64, embedding dimension 16, learning rate 0.001, 200 epochs, Adam optimizer) to embed each bin, then clusters the 16-dimensional embeddings with HDBSCAN. Contiguous regions with the same label that exceed 200 Kb are called as TADs and written to the output TAD file. The README's /src scripts extend the run: plot_heatmaps.py visualizes TADs, evaluate_TAD.py scores them (e.g., FMS), and evaluate-achor-count.py computes ChIP-seq anchor counts.

## Related tools

- **TOAST** (Main pipeline (toast.py): GAE training on 400x400 submatrices, HDBSCAN clustering, TAD generation) — https://github.com/ghaiyan/TOAST
- **HDBSCAN** (Unsupervised clustering that assigns labels to each genomic bin's 16-dimensional GAE embedding)
- **PyTorch** (Implements and trains the 2-layer graph auto-encoder (hidden dim 64, embedding dim 16, Adam, 200 epochs))

## Examples

```
python toast.py  # edit input/output folders in /src to point at the NxN KR-normalized contact matrix and TAD output directory
```

## Evaluation signals

- FMS around 0.98 on Wang et al. simulated data across noise levels, with TAD number difference vs ground truth of 1–8 and average TAD length difference up to 5 Kb
- Average anchoring ratios of TAD boundaries should approximate the reported values (e.g., CTCF 0.66, SMC3 0.47, RAD21 0.54)
- All called TADs are contiguous same-label regions exceeding 200 Kb in length
- TOAST outperforms existing algorithms (Armatus, Arrowhead, TopDom, HiCSeg, TADtree, PSYCHIC, TADbit) on simulated matrices at different noise levels
- Sequential runtime on GM12878 should be on the order of 2 h (50 Kb), 4 h (25 Kb), 11 h (5 Kb); parallel run completes in roughly 0.5 h

## Limitations

- Existing TAD identification methods, TOAST included, face trade-offs among complex parameter settings, lengthy computation time, or low accuracy
- Optimal results require the specific architecture: 2-layer GAE with hidden_dim 64 and output_dim 16 (a single-layer network with output_dim 128 was comparable but hidden_dim=64/output_dim=16 was most robust)
- Input must be an NxN equal-sized-region intra-chromosomal contact matrix; the pipeline was validated on GM12878 (chromosomes 1–22) and simulated datasets, not other cell lines

## Evidence
