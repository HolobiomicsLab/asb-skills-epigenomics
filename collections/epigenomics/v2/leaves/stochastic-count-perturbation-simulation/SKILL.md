---
name: stochastic-count-perturbation-simulation
description: Use when a clustering or alignment method needs a ground-truth benchmark
  of noisy peak-profile signals whose seed-of-origin labels are known, e.g. to score
  DGW against standard hierarchical clustering via a multi-class MCC.
when_to_use_negative:
- The peaks are real measured data and no ground-truth labels are available — simulation
  with seed labels is only meaningful for benchmarking
- The analysis goal is to cluster or align actual ENCODE epigenomic peaks rather than
  to evaluate a clustering method
- The target regions lack the expected landmark structure (first splicing site / TSS)
  needed to produce bimodal and single-peak seed pattern types
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Implementation language for the perturbation pipeline (Gaussian noise, bin
    removal/duplication, orientation flips)
- name: DGW
  role: Downstream consumer of the simulated 500-peak datasets, used to verify the
    seed clusters are recoverable
  repo: github.com/lukauskas/dgw
- name: UCSC genome browser
  role: Source of the knownGenes track used to extract the first 2 kb of the five
    seed genes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:de8a4d902451cd97999ae2ac7c643267f1c6464f0a1a8d5947cba4e83febff4b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DGW
  - Python
  - UCSC genome browser
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# stochastic-count-perturbation-simulation

## Summary

A simulation scheme that generates a labelled 500-peak epigenomic dataset by perturbing five real H3K4me3 seed patterns from ENCODE K562 data with multiplicative Gaussian noise, bin deletion/duplication, and orientation flips, to benchmark clustering methods such as DGW.

## When to use

Use when a clustering or alignment method needs a ground-truth benchmark of noisy peak-profile signals whose seed-of-origin labels are known, e.g. to score DGW against standard hierarchical clustering via a multi-class MCC.

## When NOT to use

- The peaks are real measured data and no ground-truth labels are available — simulation with seed labels is only meaningful for benchmarking
- The analysis goal is to cluster or align actual ENCODE epigenomic peaks rather than to evaluate a clustering method
- The target regions lack the expected landmark structure (first splicing site / TSS) needed to produce bimodal and single-peak seed pattern types

## Inputs

- ENCODE H3K4me3 K562 read-count profiles (first 2 kb of five UCSC knownGenes regions spanning the first splicing site)
- BAM file wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam from ENCODE K562
- Parameter grid over noise variance v, bin modification probability p, and orientation-flip probability fp (e.g. v = 0.25, p = 0.25, fp = 0.1)

## Outputs

- A 500-peak simulated dataset per (v, p, fp) parameter set (5 seed groups x 99 simulated peaks), with binned 50 bp read counts and seed-of-origin labels for scoring multi-class MCC

## How to apply

Extract the first 2 kb of five genes from the UCSC knownGenes track and pull the H3K4me3 read-count profiles for these regions from the ENCODE K562 dataset (wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam), ensuring each region contains its first splicing site and spans both bimodal and single-peak pattern types. For each seed, apply multiplicative Gaussian noise with variance v (e.g. v = 0.25 for the stress parameter set) to the read counts in each 50 bp bin. Then remove or duplicate each bin with probability p (e.g. p = 0.25), allowing duplicated bins to be re-duplicated so local stretches of varying length appear, and flip the peak's orientation with probability fp (e.g. fp = 0.1) to mimic anti-sense transcription. Repeat the perturbation 99 times per seed for each grid point of (v, p, fp), assembling a 500-peak dataset per parameter set with seed-of-origin labels so downstream clustering can be scored with the multi-class Matthews Correlation Coefficient.

## Related tools

- **Python** (Implementation language for the perturbation pipeline (Gaussian noise, bin removal/duplication, orientation flips))
- **DGW** (Downstream consumer of the simulated 500-peak datasets, used to verify the seed clusters are recoverable) — github.com/lukauskas/dgw
- **UCSC genome browser** (Source of the knownGenes track used to extract the first 2 kb of the five seed genes)

## Evaluation signals

- The generated dataset contains exactly 5 seed groups x 99 simulated peaks = 500 profiles per parameter set
- Each simulated peak is traceable to its seed region, enabling scoring with the multi-class Matthews Correlation Coefficient
- Seed regions span both bimodal and single-peak H3K4me3 pattern types and each contains its first splicing site
- Perturbations respect the chosen stress parameters (e.g. v = 0.25, p = 0.25, fp = 0.1), with duplicated bins forming local stretches of varying length

## Limitations

- Standard hierarchical clustering only recovers the seed structure when noise and modifications are small (v < 0.15, p < 0.15), so the benchmark discriminates only at moderate-to-high stress levels
- Simulation models only three perturbation types (Gaussian noise, bin deletion/duplication, orientation flip); real epigenomic variation may include other distortions
- The dataset is seeded from only five real H3K4me3 patterns from K562, limiting pattern diversity

## Evidence

- [methods] Simulation seeds 500 peaks from five ENCODE H3K4me3 K562 seed patterns: "In contrast to standard hierarchical clustering DGW identifies 5 clusters with approximately 100 members each, corresponding well to the initial five seed patterns."
- [methods] Hierarchical clustering only succeeds at low stress levels: "Standard Hierarchical clustering is able to correctly group the simulated peaks according to the pattern they are originally derived from only if the added noise and modifications are small"
- [methods] Peak profiles are binned into 50 bp windows: "coverage within peak regions are binned into non-overlapping windows spanning 50 bp"
- [readme] DGW Python library used for simulation validation: "This is the repository for the open source DGW library developped by Saulius Lukauskas."
