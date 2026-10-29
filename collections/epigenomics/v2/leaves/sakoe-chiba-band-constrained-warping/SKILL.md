---
name: sakoe-chiba-band-constrained-warping
description: Use when aligning two or more epigenomic tracks (or other multivariate
  time series of binned coverage vectors) by shape similarity, where the warping path
  must be bounded near the diagonal and strand orientation must be handled.
when_to_use_negative:
- Input regions contain no enrichment structure to align, or peaks are shorter than
  5 or longer than 1000 bins — the paper's filtering excludes such peaks before alignment
- The sequences are already aligned or the analysis requires point-wise positional
  comparison without local rescaling — DTW would warp away genuine positional differences
- A tighter or looser warping constraint is required by the application — the width-12
  band is a paper-specific tuning, not a universal default
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: DGW
  role: Open source Python package that performs the Sakoe-Chiba-band-constrained
    DTW alignment and clustering of multiple epigenomic marks
  repo: github.com/lukauskas/dgw
- name: mlpy
  role: Provides the base DTW implementation (dtw_std / constrained DTW) on which
    DGW's alignment builds; installed from the authors' fork
  repo: github.com/sauliusl/mlpy
- name: dtwco
  role: Standalone constrained-DTW package (successor to mlpy.dtw) implementing the
    Sakoe-Chiba and Itakura constraints in Python
  repo: lukauskas/dtwco
- name: numpy
  role: Array handling of the binned coverage vectors and the DTW dynamic-programming
    cost matrix
- name: scipy
  role: Supporting scientific routines in the alignment/clustering workflow
- name: pandas
  role: Reading the processed binned coverage dataset of peaks and bins
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:4c410d3fff82649da9faeac93d53904097c6f00d3f9e44f1137acca153c8531d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DGW
  - Python
  - dtwco
  - mlpy
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# sakoe-chiba-band-constrained-warping

## Summary

Constrained multidimensional Dynamic Time Warping that aligns sequences of binned epigenomic coverage vectors using a Sakoe-Chiba band of width 12, capping the warping path's departure from the diagonal so that epigenomic marks can be matched by shape with O(k*max(N,M)) run-time.

## When to use

Use when aligning two or more epigenomic tracks (or other multivariate time series of binned coverage vectors) by shape similarity, where the warping path must be bounded near the diagonal and strand orientation must be handled.

## When NOT to use

- Input regions contain no enrichment structure to align, or peaks are shorter than 5 or longer than 1000 bins — the paper's filtering excludes such peaks before alignment
- The sequences are already aligned or the analysis requires point-wise positional comparison without local rescaling — DTW would warp away genuine positional differences
- A tighter or looser warping constraint is required by the application — the width-12 band is a paper-specific tuning, not a universal default

## Inputs

- Processed binned coverage dataset: non-overlapping 50 bp bins per peak region, one coverage vector per bin for each epigenomic mark (from MACS2-called peaks merged across marks)
- Pair of peak sequences (each a sequence of multidimensional coverage vectors) to be aligned
- Sakoe-Chiba band width parameter (12 in the paper's configuration)

## Outputs

- Pairwise DTW distances between peaks (strand-invariant, minimum of forward and reversed alignments)
- Warping paths for each pairwise alignment
- Aligned epigenomic tracks for downstream clustering
- Cluster prototype sequences computed via scaled prioritised shape averaging, stored with warping-path pickles

## How to apply

Represent each peak as a sequence of multidimensional vectors (50 bp non-overlapping bins of ChIP-Seq coverage per epigenomic mark, loaded with pandas/numpy). Compute the DTW of two sequences of vectors using squared Euclidean distance as the local metric, with dynamic programming constrained so that the first and last points are mapped to each other and indices advance by at most one step. Impose the Sakoe-Chiba band constraint of width 12 so the warping path cannot depart further than 12 bins from the diagonal, reducing run-time to O(k*max(N,M)). To handle strand orientation, compute each pairwise distance twice — once with the sequences unchanged and once with one reversed — and return the smaller distance. The aligned tracks then feed the scaled prioritised shape averaging algorithm, which computes a prototype sequence for a dendrogram node while storing prototypes and the warping paths of originals projected onto prototypes.

## Related tools

- **DGW** (Open source Python package that performs the Sakoe-Chiba-band-constrained DTW alignment and clustering of multiple epigenomic marks) — github.com/lukauskas/dgw
- **mlpy** (Provides the base DTW implementation (dtw_std / constrained DTW) on which DGW's alignment builds; installed from the authors' fork) — github.com/sauliusl/mlpy
- **dtwco** (Standalone constrained-DTW package (successor to mlpy.dtw) implementing the Sakoe-Chiba and Itakura constraints in Python) — lukauskas/dtwco
- **numpy** (Array handling of the binned coverage vectors and the DTW dynamic-programming cost matrix)
- **scipy** (Supporting scientific routines in the alignment/clustering workflow)
- **pandas** (Reading the processed binned coverage dataset of peaks and bins)

## Examples

```
pip install -e git://github.com/sauliusl/mlpy.git#egg=mlpy
```

## Evaluation signals

- The warping path never departs more than 12 bins from the diagonal (Sakoe-Chiba band respected) and the first and last points of the two sequences are mapped to each other
- Each pairwise distance equals the minimum of the forward and reversed-sequence alignments, confirming strand handling
- Warping decreases the entropy of TSS/FSS position distributions — the paper reports a 12.91% decrease for TSS and 7.72% for FSS in the selected cluster
- Output prototype and warping-path pickle files are produced and loadable
- Run-time scales as O(k*max(N,M)) rather than O(N*M), consistent with the band constraint

## Limitations

- The band width of 12 is a fixed tuning choice from the paper; other applications may need different widths
- dtwco (a standalone constrained-DTW successor package) is no longer maintained and is read-only as of 2020-10-06; the README suggests tslearn as an alternative
- DGW requires Python 2.7 to run, per the methods section

## Evidence

- [methods] constrained the DTW with a Sakoe-Chiba Band of width 12: "constrained the DTW with a Sakoe-Chiba Band of width 12"
- [methods] we compute two distances for every pairwise peak comparison: one with the two sequences unchanged, and one with one of them reversed: "we compute two distances for every pairwise peak comparison: one with the two sequences unchanged, and one with one of them reversed"
- [methods] here we use the scaled prioritised shape averaging algorithm of [18]: "here we use the scaled prioritised shape averaging algorithm of [18]"
- [methods] we observe a decrease of 12.91 % for TSS and 7.72 % for FSS location distributions in the selected cluster after warping: "we observe a decrease of 12.91 % for TSS and 7.72 % for FSS location distributions in the selected cluster after warping"
- [readme] This code implements a number of constrained Dynamic Time Warping (DTW) algorithms as described in the literature. Particularly, we implement Sakoe & Chiba constraint: "This code implements a number of constrained Dynamic Time Warping (DTW) algorithms as described in the literature.
Particularly, we implement Sakoe & Chiba constraint (Sakoe78)"
