---
name: synthetic-benchmark-dataset-generation
description: Use when you need a ground-truth-labelled benchmark to test whether a
  clustering or alignment method (e.g. DGW vs hierarchical clustering) can recover
  known seed patterns from epigenomic peak-profile data.
when_to_use_negative:
- The goal is to measure clustering performance on real, unlabelled epigenomic data
  where no seed-of-origin ground truth exists (use the ENCODE evaluation instead).
- The input is already a curated labelled benchmark or feature table, not raw seed
  profiles from which synthetic variants must be seeded.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: DGW
  role: Downstream clustering/alignment tool whose recovery of the 5 seed clusters
    (multi-class MCC) the synthetic dataset is built to benchmark
  repo: https://github.com/lukauskas/dgw
- name: UCSC genome browser
  role: Source of the knownGenes table used to extract the initial 2 kb of five seed
    genes
- name: Python
  role: Language in which the perturbation simulation (Gaussian noise, bin duplication/removal,
    orientation flipping) is implemented
  repo: https://github.com/lukauskas/dgw
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:c62355a707d175534aa4e82335807432fa563a88a12ed4bdb8791686ae4a8fee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lukauskas/dgw
  tools:
  - DGW
  - Python
  - UCSC genome browser
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# synthetic-benchmark-dataset-generation

## Summary

Generates a labelled synthetic benchmark of ~500 ChIP-Seq peak profiles by perturbing five real seed patterns (ENCODE K562 H3K4me3 read-count profiles from UCSC knownGenes 2 kb regions) with parameterised multiplicative noise, bin duplication/deletion, and orientation switching, so that clustering performance can be scored against known seed-of-origin labels.

## When to use

Use when you need a ground-truth-labelled benchmark to test whether a clustering or alignment method (e.g. DGW vs hierarchical clustering) can recover known seed patterns from epigenomic peak-profile data.

## When NOT to use

- The goal is to measure clustering performance on real, unlabelled epigenomic data where no seed-of-origin ground truth exists (use the ENCODE evaluation instead).
- The input is already a curated labelled benchmark or feature table, not raw seed profiles from which synthetic variants must be seeded.

## Inputs

- First 2 kb of five gene regions from the UCSC knownGenes table
- ENCODE K562 H3K4me3 aligned read data (e.g. wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam)
- Perturbation parameter grid (v, p, fp), e.g. v=0.25, p=0.25, fp=0.1

## Outputs

- A 500-profile peak dataset per (v, p, fp) parameter set (5 seed groups x 99 simulated peaks)
- Seed-of-origin labels enabling multi-class MCC scoring of clustering results

## How to apply

Extract the first 2 kb of five genes from the UCSC knownGenes track and pull H3K4me3 read-count profiles for these regions from the ENCODE K562 dataset (wgEncodeBroadHistoneK562H3k4me3StdAlnRep1), choosing regions that contain their first splicing site and span bimodal and single-peak pattern types. Bin coverage into non-overlapping 50 bp windows, then for each seed apply multiplicative Gaussian noise with variance v (e.g. v=0.25), randomly remove or duplicate each bin with probability p (e.g. p=0.25, allowing duplicate-of-duplicate bins), and flip the peak orientation with probability fp (e.g. fp=0.1) to mimic anti-sense transcription. Repeat the perturbation 99 times per seed for each (v, p, fp) grid point, producing a 500-peak dataset per parameter set with seed-of-origin labels. Score downstream clustering against these labels using the multi-class Matthews Correlation Coefficient; as a sanity check on difficulty, note that standard hierarchical clustering succeeds only for small perturbations (v<0.15, p<0.15).

## Related tools

- **DGW** (Downstream clustering/alignment tool whose recovery of the 5 seed clusters (multi-class MCC) the synthetic dataset is built to benchmark) — https://github.com/lukauskas/dgw
- **UCSC genome browser** (Source of the knownGenes table used to extract the initial 2 kb of five seed genes)
- **Python** (Language in which the perturbation simulation (Gaussian noise, bin duplication/removal, orientation flipping) is implemented) — https://github.com/lukauskas/dgw

## Examples

```
git clone git://github.com/sauliusl/dgw.git  # see also the simulation script gist.github.com/sauliusl/5735144
```

## Evaluation signals

- Dataset size invariant: exactly 5 seed groups x 99 simulated peaks = 500 profiles per (v, p, fp) parameter set.
- Every simulated peak is traceable to its seed of origin, enabling multi-class Matthews Correlation Coefficient scoring.
- Seed regions span both bimodal and single-peak pattern types and contain their first splicing site.
- Difficulty gradient behaves as reported: standard hierarchical clustering recovers seed groups only when v<0.15 and p<0.15, while DGW still identifies 5 clusters of ~100 members at stronger perturbations.

## Limitations

- Only five real seed patterns are used; cluster structure is artificially balanced (~100 members each), which may flatter methods.
- Perturbations are limited to multiplicative Gaussian noise, bin removal/duplication, and orientation flipping; other real-data artefacts are not modelled.
- Parameter choices (v=0.25, p=0.25, fp=0.1) are one stress setting; results depend on the (v, p, fp) grid explored.

## Evidence

- [other] The simulation generates peak-profile data seeded from five initial seed patterns, with peaks distributed among them so that each pattern yields a cluster of approximately 100 members: [evidence span withheld — anchor status: not_found]
- [methods] In contrast to standard hierarchical clustering DGW identifies 5 clusters: "In contrast to standard hierarchical clustering DGW identifies 5 clusters with approximately 100 members each, corresponding well to the initial five seed patterns."
- [methods] Standard Hierarchical clustering is able to correctly group the simulated peaks: "Standard Hierarchical clustering is able to correctly group the simulated peaks according to the pattern they are originally derived from only if the added noise and modifications are small"
- [methods] coverage within peak regions are binned into non-overlapping windows spanning 50 bp: "coverage within peak regions are binned into non-overlapping windows spanning 50 bp"
- [readme] This script is available as a Gist at https://gist.github.com/sauliusl/5735144: "This script is available as a Gist at https://gist.github.com/sauliusl/5735144"
