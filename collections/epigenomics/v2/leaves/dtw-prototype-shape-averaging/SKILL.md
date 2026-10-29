---
name: dtw-prototype-shape-averaging
description: Use when clustering multiple epigenomic marks (e.g. ChIP-Seq H3K4me3/H3K9ac
  coverage vectors per 50 bp bin) by shape and you need a warping-invariant pairwise
  distance plus per-cluster prototype sequences.
when_to_use_negative:
- Peaks shorter than 5 or longer than 1000 bins, or with fewer than 10 counts — these
  were filtered out before alignment in the DGW workflow
- Unexpressed regions lacking any bin with more than 10 reads (the --min-pileup constraint)
  — such regions are ignored upstream
- When accurate alignment requires external landmarks such as transcription start
  sites or gene annotation — DGW is designed to align without such information
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3168
tools:
- name: DGW
  role: Open source Python package that performs the constrained multidimensional
    DTW alignment and scaled prioritised shape averaging of epigenomic marks
  repo: https://github.com/lukauskas/dgw
- name: mlpy
  role: Base DTW implementation (dtw_std / Sakoe-Chiba constrained variants) whose
    API dtwco mirrors; installed from the forked repository
  repo: https://github.com/sauliusl/mlpy
- name: dtwco
  role: Stripped-down constrained DTW package implementing Sakoe-Chiba band and Itakura
    parallelogram constraints, released under GPL-3.0
  repo: https://github.com/lukauskas/dtwco
- name: fastcluster
  role: Provides the agglomerative hierarchical clustering (complete linkage) over
    the DTW distances whose nodes receive prototype sequences
  repo: https://github.com/lukauskas/fastcluster
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:74595a4d5362226cea2a9af82aa13dab46d83ad29e765fd745194702feab15df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lukauskas/dgw
  tools:
  - DGW
  - Python
  - dtwco
  - fastcluster
  - mlpy
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# dtw-prototype-shape-averaging

## Summary

Multidimensional DTW alignment of epigenomic coverage tracks (binned 50 bp per peak) constrained by a Sakoe-Chiba band of width 12, followed by scaled prioritised shape averaging to derive a prototype sequence per dendrogram node in DGW.

## When to use

Use when clustering multiple epigenomic marks (e.g. ChIP-Seq H3K4me3/H3K9ac coverage vectors per 50 bp bin) by shape and you need a warping-invariant pairwise distance plus per-cluster prototype sequences.

## When NOT to use

- Peaks shorter than 5 or longer than 1000 bins, or with fewer than 10 counts — these were filtered out before alignment in the DGW workflow
- Unexpressed regions lacking any bin with more than 10 reads (the --min-pileup constraint) — such regions are ignored upstream
- When accurate alignment requires external landmarks such as transcription start sites or gene annotation — DGW is designed to align without such information

## Inputs

- Binned coverage dataset: one coverage vector per 50 bp non-overlapping bin per epigenomic mark, one such vector sequence per peak region (pandas/numpy readable)
- Dendrogram/hierarchical clustering structure over the peak sequences (node membership lists for prototype computation)

## Outputs

- Constrained multidimensional DTW pairwise distances (strand-aware, minimum of forward and reversed alignments)
- Prototype sequences computed by scaled prioritised shape averaging for each dendrogram node
- Pickle files storing prototypes and the warping paths of original sequences projected onto prototypes

## How to apply

Represent each peak as a sequence of multidimensional coverage vectors (one vector per 50 bp non-overlapping bin per mark) loaded with pandas/numpy. Compute the DTW alignment of two vector sequences using squared Euclidean local distance and dynamic programming where the first and last points are forced to map to each other and indices advance by at most one step; constrain the warping path with a Sakoe-Chiba band of width 12 so run-time is O(k*max(N,M)). For each pairwise peak comparison compute the distance twice — once with sequences unchanged and once with one sequence reversed — and return the smaller value to handle strand orientation. Then apply the scaled prioritised shape averaging algorithm to compute a prototype sequence for all sequences attached to a dendrogram node, storing the prototypes and the warping paths of the original sequences projected onto the prototypes so hierarchical clustering (complete linkage) can proceed on the prototype space.

## Related tools

- **DGW** (Open source Python package that performs the constrained multidimensional DTW alignment and scaled prioritised shape averaging of epigenomic marks) — https://github.com/lukauskas/dgw
- **mlpy** (Base DTW implementation (dtw_std / Sakoe-Chiba constrained variants) whose API dtwco mirrors; installed from the forked repository) — https://github.com/sauliusl/mlpy
- **dtwco** (Stripped-down constrained DTW package implementing Sakoe-Chiba band and Itakura parallelogram constraints, released under GPL-3.0) — https://github.com/lukauskas/dtwco
- **fastcluster** (Provides the agglomerative hierarchical clustering (complete linkage) over the DTW distances whose nodes receive prototype sequences) — https://github.com/lukauskas/fastcluster

## Examples

```
from dgw.dtw import dtw; dist = min(dtw(x, y, metric='sqeuclidean', constraint='sakoe_chiba', band_width=12), dtw(x, y[::-1], metric='sqeuclidean', constraint='sakoe_chiba', band_width=12))
```

## Evaluation signals

- Warping decreases the entropy of TSS/FSS position distributions in the selected cluster (paper reports 12.91% TSS and 7.72% FSS decrease after warping)
- Output prototype and warping-path pickle files are produced and loadable
- Every warping path satisfies the Sakoe-Chiba band constraint of width 12, maps first points to first points and last points to last points
- Each pairwise distance equals the minimum of the forward and reversed-sequence DTW distances (strand symmetry)
- DGW clustering recovers the 5 seed patterns in the simulation study (~100 members each) where standard hierarchical clustering fails

## Limitations

- Requires Python 2.7 to run; the dgw and dtwco repositories are archived/read-only (dtwco as of 2020-10-06)
- Cutting the dendrogram at an appropriate level is a difficult choice and not resolved automatically
- Standard hierarchical clustering without warping only groups peaks correctly under small noise/modification levels (v < 0.15, p < 0.15), so the warping step is essential for noisy epigenomic data
- fastcluster must be installed from source (numpy API deprecation commented out in the fork)

## Evidence

- [methods] constrained the DTW with a Sakoe-Chiba Band of width 12: "constrained the DTW with a Sakoe-Chiba Band of width 12"
- [methods] strand-specific distance: "we compute two distances for every pairwise peak comparison: one with the two sequences unchanged, and one with one of them reversed"
- [methods] shape averaging prototype: "here we use the scaled prioritised shape averaging algorithm of [18]"
- [methods] binning: "coverage within peak regions are binned into non-overlapping windows spanning 50 bp"
- [methods] entropy decrease: "we observe a decrease of 12.91 % for TSS and 7.72 % for FSS location distributions in the selected cluster after warping"
- [readme] dtwco constraints: [evidence span withheld — anchor status: external_doc]
