---
name: multidimensional-epigenomic-track-warping
description: Use when multiple epigenomic tracks (e.g. ChIP-Seq coverage binned into
  50 bp windows per peak) must be aligned by shape across peaks without gene annotation
  or TSS information, as a precursor to hierarchical clustering.
when_to_use_negative:
- When peaks are shorter than 5 or longer than 1000 bins, or have fewer than 10 counts
  — the paper's filtering excludes these before alignment
- When gene annotation or TSS positions are available and a supervised/annotation-based
  alignment is preferred — DGW deliberately solves alignment without such information
- When the input is already an aligned feature table rather than raw binned coverage
  sequences — warping would be redundant
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3295
tools:
- name: DGW
  role: Open source package providing the constrained multidimensional DTW alignment
    and clustering of epigenomic marks implemented by this skill
  repo: https://github.com/lukauskas/dgw
- name: mlpy
  role: Base DTW implementation (dtw_std / constrained DTW) whose Sakoe-Chiba constrained
    algorithm underlies the alignment; installed from the paper's fork
  repo: https://github.com/sauliusl/mlpy
- name: dtwco
  role: Standalone successor package implementing the Sakoe-Chiba constrained DTW
    algorithms extracted from mlpy
  repo: https://github.com/lukauskas/dtwco
- name: numpy
  role: Holds peak sequences as arrays of per-bin coverage vectors and computes squared-Euclidean
    local distances
- name: scipy
  role: Used alongside pandas/numpy for reading and handling the binned coverage data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:944e93a2489a6d206236360f65c78c55606b67696264803a93815db096978b32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lukauskas/dgw
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
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# multidimensional-epigenomic-track-warping

## Summary

Implements DGW's constrained multidimensional Dynamic Time Warping to align sequences of binned epigenomic coverage vectors by shape, then derives a cluster prototype via the scaled prioritised shape averaging algorithm. The Sakoe-Chiba band (width 12) and strand-symmetric distance computation make alignment tractable and orientation-invariant.

## When to use

Use when multiple epigenomic tracks (e.g. ChIP-Seq coverage binned into 50 bp windows per peak) must be aligned by shape across peaks without gene annotation or TSS information, as a precursor to hierarchical clustering.

## When NOT to use

- When peaks are shorter than 5 or longer than 1000 bins, or have fewer than 10 counts — the paper's filtering excludes these before alignment
- When gene annotation or TSS positions are available and a supervised/annotation-based alignment is preferred — DGW deliberately solves alignment without such information
- When the input is already an aligned feature table rather than raw binned coverage sequences — warping would be redundant

## Inputs

- Processed binned coverage dataset: one coverage vector per 50 bp non-overlapping bin per peak for each epigenomic mark (e.g. ENCODE K562 H3K4me3 and H3K9ac)
- Two sequences of multidimensional vectors to align (peaks)
- Sakoe-Chiba band width parameter (12 in the paper)

## Outputs

- Pairwise DTW distances between peaks (strand-symmetric, band-constrained)
- Aligned epigenomic tracks (warping paths)
- Cluster prototype sequences computed by scaled prioritised shape averaging, stored as pickle files
- Warping paths of originals projected onto prototypes, stored as pickle files

## How to apply

Represent each peak as a sequence of multidimensional vectors (one coverage vector per 50 bp non-overlapping bin per mark) loaded with pandas/numpy. Run DTW on two sequences using a squared-Euclidean local distance with dynamic programming, constraining the first and last points to map to each other and indices to advance by at most one step; restrict the warping path with a Sakoe-Chiba band of width 12 so run-time is O(k*max(N,M)). For each pairwise peak comparison compute the distance twice — once with the sequences unchanged and once with one reversed — and return the smaller distance to handle strand orientation. Feed the aligned tracks into the scaled prioritised shape averaging algorithm to compute a prototype sequence for each dendrogram node, storing prototypes and the warping paths of originals projected onto prototypes. Validate that warping decreases the entropy of TSS/FSS position distributions (paper: 12.91% TSS and 7.72% FSS decrease in the selected cluster) and that prototype and warping-path pickle files are produced and loadable.

## Related tools

- **DGW** (Open source package providing the constrained multidimensional DTW alignment and clustering of epigenomic marks implemented by this skill) — https://github.com/lukauskas/dgw
- **mlpy** (Base DTW implementation (dtw_std / constrained DTW) whose Sakoe-Chiba constrained algorithm underlies the alignment; installed from the paper's fork) — https://github.com/sauliusl/mlpy
- **dtwco** (Standalone successor package implementing the Sakoe-Chiba constrained DTW algorithms extracted from mlpy) — https://github.com/lukauskas/dtwco
- **numpy** (Holds peak sequences as arrays of per-bin coverage vectors and computes squared-Euclidean local distances)
- **scipy** (Used alongside pandas/numpy for reading and handling the binned coverage data)

## Examples

```
dist, cost, path = dtwco.dtw(x, y, constraint='sakoe_chiba', dist_only=False)
```

## Evaluation signals

- Warping decreases the entropy of TSS/FSS position distributions by amounts comparable to the paper's reported 12.91% (TSS) and 7.72% (FSS) in the selected cluster
- Warping path stays within the Sakoe-Chiba band of width 12 from the diagonal (|i - j| <= 12)
- First and last points of the two sequences are mapped to each other and path indices advance by at most one step
- Pairwise distance is symmetric under strand reversal: dist(A,B) equals min(dist(A,B), dist(reverse(A),B))
- Prototype and warping-path pickle files are produced and loadable after a run

## Limitations

- The Sakoe-Chiba band of width 12 caps how far the warping path may depart from the diagonal, so large shifts beyond ~12 bins cannot be corrected
- dtwco repository is no longer maintained (read-only as of 2020-10-06); the README suggests using tslearn instead
- Strand handling requires computing each pairwise distance twice (forward and reversed), doubling per-pair cost
- Cutting the resulting dendrogram at an appropriate level is acknowledged as a difficult choice

## Evidence

- [methods] constrained the DTW with a Sakoe-Chiba Band of width 12: "constrained the DTW with a Sakoe-Chiba Band of width 12"
- [methods] we compute two distances for every pairwise peak comparison: one with the two sequences unchanged, and one with one of them reversed: "we compute two distances for every pairwise peak comparison: one with the two sequences unchanged, and one with one of them reversed"
- [methods] coverage within peak regions are binned into non-overlapping windows spanning 50 bp: "coverage within peak regions are binned into non-overlapping windows spanning 50 bp"
- [methods] here we use the scaled prioritised shape averaging algorithm of [18]: "here we use the scaled prioritised shape averaging algorithm of [18]"
- [methods] we observe a decrease of 12.91 % for TSS and 7.72 % for FSS location distributions in the selected cluster after warping: "we observe a decrease of 12.91 % for TSS and 7.72 % for FSS location distributions in the selected cluster after warping"
- [readme] we implement Sakoe & Chiba constraint (Sakoe78), and Itakura Parallelogram (Itakura75): [evidence span withheld — anchor status: external_doc]
