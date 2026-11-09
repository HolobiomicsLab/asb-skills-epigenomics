---
name: chromatin-contact-map-analysis
description: Use when you have an intra-chromosomal NxN Hi-C contact matrix (e.g.,
  KR-normalized at 50/25/5 Kb from GM12878) and need unsupervised TAD boundary detection
  without complex parameter tuning.
when_to_use_negative:
- Input is raw sequencing reads or .hic files alone without first dumping a normalized
  NxN contact matrix — TOAST requires the matrix format as input
- The analysis goal is enhancer-promoter interaction prediction (e.g., EPIANN's GM12878
  EPI dataset task), which is a different problem from TAD boundary identification
- You need supervised TAD calling against a labeled ground truth on simulated data
  with known boundaries at noise levels where FMS degrades substantially
edam_operation: http://edamontology.org/operation_3903
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: TOAST
  role: 'Main pipeline: data preprocessing into 400x400 submatrices, GAE training,
    clustering, and TAD generation; run via toast.py in the /src folder'
  repo: https://github.com/ghaiyan/TOAST
- name: HDBSCAN
  role: Unsupervised clustering of the 16-dimensional GAE embeddings to assign a label
    to each genomic bin
  repo: https://github.com/ghaiyan/TOAST
- name: Juicer tools (juicer_tools.js)
  role: Dumps KR-normalized .hic data into the NxN intra-chromosomal contact matrix
    format TOAST consumes
- name: PyTorch
  role: Framework for training the 2-layer graph auto-encoder (hidden 64, embedding
    16, Adam, MSE loss)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:0dd0b315bf74d6f92b39baf297a02fd27d0e386bfa2223c6516b88fc402dcc56
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
  - Juicer tools (juicer_tools.js)
  - PyTorch
  - TOAST
  edam_operation: http://edamontology.org/operation_3903
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# chromatin-contact-map-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify topologically associated domains (TADs) from Hi-C contact matrices using TOAST, which encodes each genomic bin as a graph node, derives 16-dimensional embeddings with a 2-layer graph auto-encoder, clusters them with HDBSCAN, and calls contiguous same-label regions longer than 200 Kb as TADs.

## When to use

Use when you have an intra-chromosomal NxN Hi-C contact matrix (e.g., KR-normalized at 50/25/5 Kb from GM12878) and need unsupervised TAD boundary detection without complex parameter tuning.

## When NOT to use

- Input is raw sequencing reads or .hic files alone without first dumping a normalized NxN contact matrix — TOAST requires the matrix format as input
- The analysis goal is enhancer-promoter interaction prediction (e.g., EPIANN's GM12878 EPI dataset task), which is a different problem from TAD boundary identification
- You need supervised TAD calling against a labeled ground truth on simulated data with known boundaries at noise levels where FMS degrades substantially

## Inputs

- Tab-separated NxN intra-chromosomal Hi-C contact matrix (N = number of equal-sized genomic regions per chromosome), dumped from .hic files with juicer_tools
- GM12878 .hic file (4DNFI1UEG1HD from 4D Nucleome Data Portal) or GEO GSE63525 real Hi-C data at 50 Kb, 25 Kb, or 5 Kb resolution
- Simulated Hi-C contact matrices (e.g., Forcato et al. 2017, Wang et al. IMR-90, Trussart et al.) for validation

## Outputs

- TAD boundary file in (start bin, start location, end bin, end location) format
- TAD heatmap plots (plot_heatmaps.py)
- TAD evaluation metrics: FMS, TAD count and average TAD length vs ground truth, TAD_quality (intra-TAD minus inter-TAD average contact frequency)

## How to apply

Dump the .hic file (e.g., accession 4DNFI1UEG1HD from the 4D Nucleome Data Portal) to a tab-separated NxN intra-chromosomal matrix using juicer_tools; TOAST consumes the NxN format. Partition each chromosome's matrix along the diagonal into non-overlapping 400x400 submatrices (n=400 for 50 Kb resolution) and treat each submatrix as a graph adjacency matrix. Train a 2-layer graph auto-encoder (hidden dimension 64, embedding dimension 16, learning rate 0.001, 200 epochs, Adam optimizer, MSE reconstruction loss) to obtain 16-dimensional node embeddings. Cluster embeddings with HDBSCAN to label each bin, group contiguous same-label bins, and retain only regions exceeding 200 Kb (at 50 Kb, at least 4 bins) as TADs, written as (start bin, start location, end bin, end location).

## Related tools

- **TOAST** (Main pipeline: data preprocessing into 400x400 submatrices, GAE training, clustering, and TAD generation; run via toast.py in the /src folder) — https://github.com/ghaiyan/TOAST
- **HDBSCAN** (Unsupervised clustering of the 16-dimensional GAE embeddings to assign a label to each genomic bin) — https://github.com/ghaiyan/TOAST
- **Juicer tools (juicer_tools.js)** (Dumps KR-normalized .hic data into the NxN intra-chromosomal contact matrix format TOAST consumes)
- **PyTorch** (Framework for training the 2-layer graph auto-encoder (hidden 64, embedding 16, Adam, MSE loss))

## Examples

```
python toast.py  # in /src, after setting input (NxN matrix folder) and output folder
```

## Evaluation signals

- TAD_quality (average intra-TAD minus inter-TAD contact frequency) is a positive rational number — on GM12878 50 Kb data TOAST reached 1654.03 with an average of 4736 TADs
- Identified TADs must exceed 200 Kb (at least 4 bins at 50 Kb resolution) and average TAD lengths should fall within the expected 200 Kb–5 Mb interval
- On Wang et al. simulated data with noise, FMS should remain around 0.98, TAD number difference vs ground truth within 1–8, and average length difference up to ~5 Kb
- TAD boundary enrichment for chromatin marks: expect average anchoring ratios of CTCF ~0.66, SMC3 ~0.47, RAD21 ~0.54 (evaluated via evaluate-achor-count.py with ENCODE ChIP-seq accessions)
- Sanity-check output schema: TAD file lists (start bin, start location, end bin, end location) per chromosome

## Limitations

- Performance depends on resolution-dependent submatrix size (400x400 tuned for 50 Kb); other resolutions (25 Kb, 5 Kb) require re-checking the bin-threshold logic for the 200 Kb minimum
- TAD boundaries derived from unsupervised HDBSCAN labels lack parameter-driven control, so anchoring ratios for weaker marks (H3K9me3 0.12, Promoters 0.13) are low
- Contiguous same-label regions shorter than 200 Kb are discarded, so sub-200 Kb chromatin domains are never reported

## Evidence

- [methods] Graph auto-encoder architecture: "the optimal TAD identification result is achieved when the network has 2 layers, an embedding dimension of 16, and a hidden layer dimension of 64"
- [methods] TAD length filter: "If the length of a contiguous region exceeds 200 Kb, it can be identified as a TAD"
- [abstract] Pipeline overview: "the unsupervised clustering algorithm HDBSCAN is utilized to assign labels to each genomic bin, facilitating the identification of contiguous regions with the same label as TADs"
- [readme] Input format: "The input to CASPIAN is a tab seperated N by N intra-chromosomal contact matrix derived from Hi-C data, where N is the number of equal-sized regions of a chromosome."
- [results] GM12878 benchmark result: [evidence span withheld — anchor status: not_found]
- [readme] Evaluation scripts: "evaulate_TAD.py:evalute TADs"
