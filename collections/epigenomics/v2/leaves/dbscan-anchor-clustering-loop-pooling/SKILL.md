---
name: dbscan-anchor-clustering-loop-pooling
description: Use when you already have per-pixel loop probabilities (e.g. from `peakachu
  score_chromosome` or `score_genome`) and need to reduce them to discrete, non-redundant
  loop calls with anchor coordinates and probability/frequency columns.
when_to_use_negative:
- You have not yet scored pixels — pooling requires per-pixel probabilities from score_chromosome/score_genome,
  not a raw .cool or .hic contact map.
- You need an exhaustive per-pixel significance list rather than non-redundant loop
  calls (pooling deliberately collapses clusters to one centroid each).
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Peakachu
  role: Provides the `pool` subcommand that defines 1D anchor regions from high-probability
    pixels and applies DBSCAN clustering to emit non-redundant centroid loop loci
    in bedpe format.
  repo: https://github.com/tariks/peakachu
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:dedbe20af6f8a470c81b436d20492616a7ff157bc4801dc2744f26eb5afc4db5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tariks/peakachu
  tools:
  - Peakachu
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# DBSCAN anchor clustering loop pooling

## Summary

Converts per-pixel loop probabilities from a Peakachu random-forest model into non-redundant centroid loop calls by defining 1D anchor regions enriched for highly scored pixels (P > 0.9) and clustering connected anchors with DBSCAN, as implemented in the `peakachu pool` command.

## When to use

Use when you already have per-pixel loop probabilities (e.g. from `peakachu score_chromosome` or `score_genome`) and need to reduce them to discrete, non-redundant loop calls with anchor coordinates and probability/frequency columns.

## When NOT to use

- You have not yet scored pixels — pooling requires per-pixel probabilities from score_chromosome/score_genome, not a raw .cool or .hic contact map.
- You need an exhaustive per-pixel significance list rather than non-redundant loop calls (pooling deliberately collapses clusters to one centroid each).

## Inputs

- Per-pixel probability output from peakachu score_chromosome or score_genome (tabular file of scored pixels)
- Probability threshold t (e.g. 0.9; article used 0.92 for the H3K27ac model and 0.97 for the CTCF model)

## Outputs

- BEDPE file of non-redundant centroid loop loci (e.g. GM12878-chr2-loops.bedpe) with predicted probability in column 7 and interaction frequency in the final column

## How to apply

Score all intra-chromosomal pixels with a trained random-forest model (`peakachu score_chromosome`/`score_genome`, default distance range 6–300 bins and minimum-prob 0.5), then run `peakachu pool` with a probability threshold (e.g. `-t 0.9`). Pooling first defines 1D loop anchor regions enriched for highly scored (P > 0.9) pixels, then runs DBSCAN between any two connected anchor regions to group adjacent high-probability pixels into clusters. From each cluster, the most significant representative pixel is selected as a non-redundant centroid locus, and pairs of centroids are written to a bedpe file. Rationale: raw per-pixel probabilities over-cluster around the same loop, so greedy pooling within DBSCAN-defined clusters yields one call per biological interaction. The retained output carries the predicted probability in column 7 and the interaction frequency in the final column, and every call should satisfy the threshold used for pooling.

## Related tools

- **Peakachu** (Provides the `pool` subcommand that defines 1D anchor regions from high-probability pixels and applies DBSCAN clustering to emit non-redundant centroid loop loci in bedpe format.) — https://github.com/tariks/peakachu

## Examples

```
peakachu pool -t 0.9 -i GM12878-chr2-scored.txt -O GM12878-chr2-loops.bedpe
```

## Evaluation signals

- Output bedpe file exists and each row's column 7 probability exceeds the pooling threshold t (e.g. all calls > 0.9 when pooled with -t 0.9).
- Final column of each bedpe row contains the interaction frequency, and anchor coordinates are valid genomic intervals on the scored chromosome.
- Centroid loci are non-redundant: no two output rows come from the same DBSCAN cluster of connected high-probability pixels.
- Compare loop counts with expectations from the paper's cutoffs (0.92 for the H3K27ac model, 0.97 for the CTCF model in GM12878).

## Limitations

- No single method captures the complete loop set — Peakachu, HiCCUPS, and Fit-Hi-C each detect a unique set of loops, so pooled Peakachu calls are not exhaustive.
- The pooling threshold is model-dependent (0.92 vs 0.97 in GM12878), and cutoffs for other cell lines are only available in Supplementary Data 2.
- Pooling results depend on upstream scoring parameters (distance bounds 6–300 bins, minimum-prob 0.5); changing them alters cluster connectivity and centroid selection.

## Evidence

- [discussion] Greedy selection of representative loop pixels via 1D anchor peaks and DBSCAN clustering: "define 1D loop anchor regions enriched for highly scored (P > 0.9) pixels, then run DBSCAN between any two connected anchor regions"
- [methods] pool (select the most significant non-redundant results from per-pixel probabilities): "The pool function serves to select the most significant non-redundant results from per-pixel probabilities calculated by the score functions"
- [readme] pool command with threshold: "pool                Print centroid loci from score_genome/score_chromosome output"
- [abstract] unique set of loops compared to enrichment-based methods: "We compare Peakachu with current enrichment-based approaches, and find that Peakachu identifies a unique set of short-range interactions."
