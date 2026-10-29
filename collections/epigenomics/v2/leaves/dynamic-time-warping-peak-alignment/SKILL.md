---
name: dynamic-time-warping-peak-alignment
description: Use when epigenomic enrichment peaks (e.g. ChIP-Seq H3K4me3/H3K9ac coverage
  in 50 bp bins) must be aligned by shape and clustered despite variable lengths,
  strand orientation, or local rescaling of genomic features like TSSs and FSSs.
when_to_use_negative:
- Peaks are already fixed-length and positionally aligned, so plain hierarchical clustering
  without warping suffices (the article's no-DTW baseline only succeeds when noise
  v and modifications p are small, e.g. v<0.15, p<0.15)
- The analysis goal is visualisation of individual enrichment peaks rather than shape-based
  clustering, where the UCSC genome browser is the article's recommended tool
- Input data are raw BAM alignments rather than binned peak coverage; MACS2 peak calling
  and binning must precede DTW alignment
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: DGW
  role: Open source Python package performing the DTW alignment and clustering of
    multiple epigenomic marks; provides the constrained DTW and shape-averaging machinery
  repo: github.com/lukauskas/dgw
- name: dtwco
  role: Standalone implementation of constrained DTW algorithms (Sakoe-Chiba band,
    Itakura parallelogram) extracted from mlpy, usable for the band-constrained distance
    computation
  repo: lukauskas/dtwco
- name: mlpy
  role: Original machine learning package whose dtw.dtw_std routine underlies the
    DGW DTW implementation; installed from the authors' fork
  repo: github.com/sauliusl/mlpy
- name: fastcluster
  role: Performs the complete-linkage agglomerative hierarchical clustering on the
    DTW distance matrix; installed from source for compatibility with recent numpy
  repo: github.com/sauliusl/fastcluster
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:e4575d199a4e0b01b5996e8ac8a676c79b633ab453e6fb316c38cab3dc44372a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DGW
  - Python
  - dtwco
  - fastcluster
  - mlpy
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# dynamic-time-warping-peak-alignment

## Summary

Constrained multidimensional Dynamic Time Warping (Sakoe-Chiba band, width 12) for aligning binned epigenomic peak coverage profiles so that marks can be matched by shape rather than strict positional correspondence, as implemented in the DGW Python package.

## When to use

Use when epigenomic enrichment peaks (e.g. ChIP-Seq H3K4me3/H3K9ac coverage in 50 bp bins) must be aligned by shape and clustered despite variable lengths, strand orientation, or local rescaling of genomic features like TSSs and FSSs.

## When NOT to use

- Peaks are already fixed-length and positionally aligned, so plain hierarchical clustering without warping suffices (the article's no-DTW baseline only succeeds when noise v and modifications p are small, e.g. v<0.15, p<0.15)
- The analysis goal is visualisation of individual enrichment peaks rather than shape-based clustering, where the UCSC genome browser is the article's recommended tool
- Input data are raw BAM alignments rather than binned peak coverage; MACS2 peak calling and binning must precede DTW alignment

## Inputs

- Binned ChIP-Seq coverage dataset (50 bp non-overlapping bins per peak, one coverage vector per bin per epigenomic mark), e.g. from MACS2-called H3K4me3/H3K9ac K562 ENCODE peaks
- Peak regions filtered to lengths of 5-1000 bins and at least 10 counts

## Outputs

- Pairwise DTW distance matrix (strand-aware, band-constrained) over peak sequences
- Prototype sequences per dendrogram node from scaled prioritised shape averaging
- Warping-path pickle files of originals projected onto prototypes, loadable for downstream use

## How to apply

Represent each peak as a sequence of multidimensional vectors (one coverage vector per 50 bp non-overlapping bin for each epigenomic mark), read with pandas/numpy. Compute pairwise DTW distances using squared Euclidean local distance and dynamic programming, constraining the first and last points to map to each other with indices advancing at most one step per move. Apply a Sakoe-Chiba band of width 12 to cap the warping path's departure from the diagonal, reducing run-time to O(k*max(N,M)). For every pairwise peak comparison compute two distances — one with sequences unchanged, one with one sequence reversed — and return the smaller to handle strand orientation. Feed the aligned tracks into scaled prioritised shape averaging to derive cluster prototypes, storing prototypes and the warping paths of originals projected onto prototypes for downstream hierarchical clustering.

## Related tools

- **DGW** (Open source Python package performing the DTW alignment and clustering of multiple epigenomic marks; provides the constrained DTW and shape-averaging machinery) — github.com/lukauskas/dgw
- **dtwco** (Standalone implementation of constrained DTW algorithms (Sakoe-Chiba band, Itakura parallelogram) extracted from mlpy, usable for the band-constrained distance computation) — lukauskas/dtwco
- **mlpy** (Original machine learning package whose dtw.dtw_std routine underlies the DGW DTW implementation; installed from the authors' fork) — github.com/sauliusl/mlpy
- **fastcluster** (Performs the complete-linkage agglomerative hierarchical clustering on the DTW distance matrix; installed from source for compatibility with recent numpy) — github.com/sauliusl/fastcluster

## Examples

```
import dtwco; dist, cost, path = dtwco.dtw(x, y, constraint='sakoe_chiba', dist_only=False)
```

## Evaluation signals

- Warping decreases entropy of TSS/FSS position distributions in a selected cluster — paper reports 12.91% TSS and 7.72% FSS decrease after warping
- Strand-aware distance check: the returned distance equals the minimum of forward and reversed-sequence DTW distances
- Band constraint check: every warping path point stays within the Sakoe-Chiba band of width 12 of the diagonal
- Prototype and warping-path pickle files are produced and loadable after clustering
- On simulated 500-peak data (99 peaks per seed, variance v up to 0.25, deletion/duplication p up to 0.25, flip probability fp=0.1), DGW with DTW attains MCC ~ 1.000 cutting into 5 clusters, e.g. 1.000 vs 0.748 for no-DTW at v=0.25, p=0.25

## Limitations

- Requires Python 2.7 to run; the mlpy dependency is a dead project, and dtwco is no longer maintained (read-only since 2020-10-06), with tslearn suggested as an alternative
- Sakoe-Chiba band width 12 caps warping flexibility; features displaced further than the band from the diagonal cannot be aligned
- Dendrogram cutting to define clusters is a difficult choice and affects the final grouping
- The dtwco package has not been released to PyPI (intended once version hits 1.0.0); installation is by cloning and pip install -e .

## Evidence

- [methods] constrained the DTW with a Sakoe-Chiba Band of width 12: "constrained the DTW with a Sakoe-Chiba Band of width 12"
- [methods] strand-aware pairwise distance: "we compute two distances for every pairwise peak comparison: one with the two sequences unchanged, and one with one of them reversed"
- [methods] entropy decrease validation: "we observe a decrease of 12.91 % for TSS and 7.72 % for FSS location distributions in the selected cluster after warping"
- [methods] binning of coverage: "coverage within peak regions are binned into non-overlapping windows spanning 50 bp"
- [readme] dtwco constraints and installation: [evidence span withheld — anchor status: external_doc]
