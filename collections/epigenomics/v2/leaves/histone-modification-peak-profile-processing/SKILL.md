---
name: histone-modification-peak-profile-processing
description: Use when benchmarking peak-shape clustering or alignment methods and
  you need ground-truth-labelled synthetic peak profiles derived from real ChIP-Seq
  (e.g. ENCODE H3K4me3 K562) seed regions with known cluster-of-origin labels.
when_to_use_negative:
- Your input is already a curated set of real peak regions to be clustered for biological
  interpretation — this skill only creates synthetic ground-truth benchmarks
- You need to detect enriched regions from raw BAM files — use a peak caller such
  as MACS2 first; simulation assumes peak regions with binned read counts already
  exist
- The task requires multi-mark (e.g. H3K4me3 + H3K9ac) joint simulation — the described
  procedure perturbs single-mark H3K4me3 seed profiles
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3173
tools:
- name: DGW
  role: Clustering/alignment package within which the simulated peak profiles are
    evaluated against seed-of-origin labels
  repo: https://github.com/lukauskas/dgw
- name: UCSC genome browser
  role: Source of the knownGenes table used to extract the 2 kb seed regions
- name: Python
  role: Implementation language for the simulation and perturbation procedure; DGW
    requires Python 2.7
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:c2a470d80faa7f6f2745b6350dbcea4fbc61a7d368625e2f68bcaa30a090c22d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lukauskas/dgw
  tools:
  - DGW
  - Python
  - UCSC genome browser
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# histone-modification-peak-profile-processing

## Summary

A simulation-based workflow for generating synthetic histone-modification peak-profile datasets seeded from real ENCODE H3K4me3 read-count profiles, perturbing them with multiplicative Gaussian noise, bin duplication/deletion, and orientation flips to create labelled benchmark data for clustering evaluation.

## When to use

Use when benchmarking peak-shape clustering or alignment methods and you need ground-truth-labelled synthetic peak profiles derived from real ChIP-Seq (e.g. ENCODE H3K4me3 K562) seed regions with known cluster-of-origin labels.

## When NOT to use

- Your input is already a curated set of real peak regions to be clustered for biological interpretation — this skill only creates synthetic ground-truth benchmarks
- You need to detect enriched regions from raw BAM files — use a peak caller such as MACS2 first; simulation assumes peak regions with binned read counts already exist
- The task requires multi-mark (e.g. H3K4me3 + H3K9ac) joint simulation — the described procedure perturbs single-mark H3K4me3 seed profiles

## Inputs

- First 2 kb of five genes from the UCSC knownGenes track (regions containing the first splicing site)
- ENCODE K562 H3K4me3 read-count profiles (wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam) for the seed regions
- Simulation parameter grid (v, p, fp), e.g. v = 0.25, p = 0.25, fp = 0.1 for the stress parameter set

## Outputs

- A 500-peak simulated dataset per (v, p, fp) parameter set (5 seed groups x 99 simulated peaks each)
- Seed-of-origin labels for every simulated peak enabling multi-class MCC scoring

## How to apply

Extract the first 2 kb of five genes from the UCSC knownGenes track and pull H3K4me3 read-count profiles for these regions from the ENCODE K562 dataset (wgEncodeBroadHistoneK562H3k4me3StdAlnRep1), ensuring each region contains its first splicing site and spans both bimodal and single-peak pattern types. For each seed region, perturb the read counts per bin: apply multiplicative Gaussian noise with variance v (e.g. v = 0.25), remove or duplicate each bin with probability p (e.g. p = 0.25, allowing repeated duplication to build local stretches), and flip the peak orientation with probability fp (e.g. fp = 0.1) to mimic anti-sense transcription. Repeat the perturbation procedure 99 times per seed region for each (v, p, fp) grid point, assembling a 500-peak dataset (5 seeds x 99 simulations) per parameter set, keeping the seed-of-origin labels. Score downstream clustering against these labels with the multi-class Matthews Correlation Coefficient; in the DGW paper this setup showed DGW recovering 5 clusters of ~100 members each, whereas standard hierarchical clustering succeeded only for small noise/modifications (v < 0.15, p < 0.15).

## Related tools

- **DGW** (Clustering/alignment package within which the simulated peak profiles are evaluated against seed-of-origin labels) — https://github.com/lukauskas/dgw
- **UCSC genome browser** (Source of the knownGenes table used to extract the 2 kb seed regions)
- **Python** (Implementation language for the simulation and perturbation procedure; DGW requires Python 2.7)

## Examples

```
git clone git://github.com/sauliusl/dgw.git && pip install -e git://github.com/sauliusl/mlpy.git#egg=mlpy
```

## Evaluation signals

- Dataset size invariant: exactly 5 seed groups x 99 simulated peaks = 500 profiles per parameter set
- Every simulated peak is traceable to its seed of origin via stored labels
- Clustering on the simulated data recovers 5 clusters with approximately 100 members each corresponding to the five seed patterns
- Multi-class Matthews Correlation Coefficient computed against seed labels quantifies clustering accuracy; standard hierarchical clustering should succeed only for v < 0.15 and p < 0.15
- Bins counts change only through the specified perturbations (Gaussian variance v, deletion/duplication probability p, flip probability fp), e.g. v = p = 0.25, fp = 0.1

## Limitations

- Only small noise/modification levels (v < 0.15, p < 0.15) are recoverable by standard hierarchical clustering, so interpretation of scores depends on the chosen stress parameters
- Orientation flips with probability fp (e.g. 0.1) mimic anti-sense transcription but are a simplification of real antisense signal
- Duplicate bins can be duplicated again, producing local stretches of varying length; extreme p values may distort peak length beyond realistic ranges
- Requires Python 2.7 to run DGW for the downstream evaluation

## Evidence

- [methods] Extract the initial 2 kb of five genes from the UCSC knownGenes track and pull the H3K4me3 read-count profiles for these regions from the ENCODE K562 dataset: "In contrast to standard hierarchical clustering DGW identifies 5 clusters with approximately 100 members each, corresponding well to the initial five seed patterns."
- [methods] apply multiplicative Gaussian noise with variance v to the read counts in each bin: "Standard Hierarchical clustering is able to correctly group the simulated peaks according to the pattern they are originally derived from only if the added noise and modifications are small"
- [abstract] We demonstrate the effectiveness of the approach in a simulation study and on a real epigenomic data set from the ENCODE project.: "We demonstrate the effectiveness of the approach in a simulation study and on a real epigenomic data set from the ENCODE project."
- [readme] This is the repository for the open source DGW library developped by Saulius Lukauskas.: "This is the repository for the open source DGW library developped by Saulius Lukauskas."
- [methods] DGW is written in Python and needs Python 2.7 to run.: "DGW is written in Python and needs Python 2.7 to run."
