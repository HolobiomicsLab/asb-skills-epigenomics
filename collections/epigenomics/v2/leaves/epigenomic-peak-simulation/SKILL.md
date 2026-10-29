---
name: epigenomic-peak-simulation
description: 'Use when benchmarking peak-shape clustering/alignment methods against
  a known ground truth: you have seed epigenomic peaks (e.g. ENCODE H3K4me3 K562)
  and need a controlled (v, p) noise grid with verifiable cluster labels.'
when_to_use_negative:
- When the goal is to validate biological alignment of genomic landmarks (TSS/FSS
  location entropy) on real data — the paper does that separately on real ENCODE data,
  not on simulations.
- When no ground-truth seed-of-origin labels can be defined for the regions — the
  multi-class MCC against seed labels would be meaningless.
- When the input is already a fixed experimental dataset without controlled perturbations
  — no (v, p) grid can be swept and no accuracy benchmark against known patterns is
  possible.
edam_operation: http://edamontology.org/operation_0310
tools:
- name: DGW
  role: 'Method under test: computes DTW pairwise distances on simulated peaks before
    complete-linkage agglomerative clustering'
  repo: github.com/lukauskas/dgw
- name: fastcluster
  role: Performs complete-linkage agglomerative hierarchical clustering on the simulated-peak
    distance matrices
  repo: http://github.com/sauliusl/fastcluster
- name: Python
  role: Implementation language for the simulation, DTW distances, and MCC table assembly
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:b0be62713734db9b140e800016c0f1c246d3e2e97fdd33c602e7393824aeea82
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DGW
  - Python
  - fastcluster
  edam_operation: http://edamontology.org/operation_0310
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# epigenomic-peak-simulation

## Summary

Generates synthetic ChIP-Seq peak benchmark datasets by perturbing seed ENCODE H3K4me3 K562 peak regions with multiplicative Gaussian noise, bin deletion/duplication, and orientation flips, in order to quantify (via multi-class MCC) how well clustering methods recover the known seed-of-origin labels.

## When to use

Use when benchmarking peak-shape clustering/alignment methods against a known ground truth: you have seed epigenomic peaks (e.g. ENCODE H3K4me3 K562) and need a controlled (v, p) noise grid with verifiable cluster labels.

## When NOT to use

- When the goal is to validate biological alignment of genomic landmarks (TSS/FSS location entropy) on real data — the paper does that separately on real ENCODE data, not on simulations.
- When no ground-truth seed-of-origin labels can be defined for the regions — the multi-class MCC against seed labels would be meaningless.
- When the input is already a fixed experimental dataset without controlled perturbations — no (v, p) grid can be swept and no accuracy benchmark against known patterns is possible.

## Inputs

- Five seed peak regions from ENCODE H3K4me3 K562 ChIP-Seq data (binned enrichment profiles)
- Noise grid parameters: variance v in 0.00–0.40, bin deletion/duplication probability p in 0.00–0.40, orientation flip probability fp=0.1
- Number of simulated peaks per seed (99) and number of clusters to cut (5)

## Outputs

- Simulated 500-peak datasets indexed by each (v, p) grid point, with seed-of-origin ground-truth labels
- mcc_simulation_table.csv: MCC values for DGW vs standard hierarchical clustering indexed by p and v

## How to apply

Start from five seed ENCODE H3K4me3 K562 peak regions and generate 99 simulated variants per seed (500 peaks total) by applying multiplicative Gaussian noise with variance v sweeping 0.00–0.40, bin deletion/duplication with probability p sweeping 0.00–0.40, and flipping peak orientation with fixed probability fp=0.1. For each grid point (v, p), run the method under test (e.g. DGW, which computes DTW pairwise distances) and the no-warping baseline (standard hierarchical clustering), both using complete-linkage agglomerative clustering via scipy/fastcluster to avoid chaining. Cut both dendrograms into 5 clusters to match the five seed patterns, then compute the multi-class Matthews Correlation Coefficient against the seed-of-origin labels. Assemble the MCC values for both methods into a table indexed by p and v and save it as mcc_simulation_table.csv. The rationale is that simulated data provide ground truth that real ENCODE data lack, allowing the paper to show that DGW achieves near-perfect MCC (~1.000 for v up to 0.25) while the no-DTW baseline degrades with stronger modifications.

## Related tools

- **DGW** (Method under test: computes DTW pairwise distances on simulated peaks before complete-linkage agglomerative clustering) — github.com/lukauskas/dgw
- **fastcluster** (Performs complete-linkage agglomerative hierarchical clustering on the simulated-peak distance matrices) — http://github.com/sauliusl/fastcluster
- **Python** (Implementation language for the simulation, DTW distances, and MCC table assembly)

## Evaluation signals

- DGW attains near-perfect MCC (~1.000) for variance v up to 0.25 (e.g. 1.000 vs 0.748 for no-DTW at v=0.25, p=0.25).
- Standard no-DTW hierarchical clustering degrades to MCC ~0.294 at the extreme grid point v=0.40, p=0.40.
- DGW identifies 5 clusters with approximately 100 members each, matching the initial five seed patterns.
- MCC table rows/columns cover the full (p, v) sweep of 0.00–0.40 and are saved as mcc_simulation_table.csv.
- Dendrograms are cut at exactly 5 clusters, matching the number of seed patterns.

## Limitations

- Standard hierarchical clustering without warping correctly groups simulated peaks only when added noise and modifications are small (v<0.15, p<0.15), so baseline comparisons are only informative across the full (v, p) grid.
- Cutting the dendrogram at an appropriate level is a difficult choice, which introduces arbitrariness in the cluster counts.
- DGW requires Python 2.7; the archived dtwco repository recommends tslearn for constrained DTW instead, so dependencies may need substitution on modern stacks.

## Evidence

- [methods] DGW identifies 5 clusters of approximately 100 members in simulation: "In contrast to standard hierarchical clustering DGW identifies 5 clusters with approximately 100 members each, corresponding well to the initial five seed patterns."
- [methods] Baseline clustering succeeds only for small perturbations: "Standard Hierarchical clustering is able to correctly group the simulated peaks according to the pattern they are originally derived from only if the added noise and modifications are small"
- [readme] DTW constraints used for pairwise distance computation: [evidence span withheld — anchor status: external_doc]
