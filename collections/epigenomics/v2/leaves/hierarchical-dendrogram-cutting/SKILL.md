---
name: hierarchical-dendrogram-cutting
description: Use when a dendrogram from complete-linkage agglomerative clustering
  of DTW-aligned peak profiles must be cut into clusters whose count reflects true
  underlying patterns or is chosen interactively near the leaves.
when_to_use_negative:
- The input peaks have not been DTW-warped and contain substantial noise/modifications
  (v or p above ~0.15) — the article shows standard hierarchical clustering without
  warping degrades badly (MCC ~0.294 at v=0.40, p=0.40)
- A fixed, validated cluster count is required but only a purely automatic cut with
  no ground truth or interactive inspection is possible — the article notes cutting
  the dendrogram 'at an appropriate level is a difficult choice'
- The data are already grouped by an external annotation (e.g. known TSS labels) and
  clustering would only duplicate existing structure
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: DGW
  role: Produces the DTW distance matrix and linkage hierarchy; dgw-explorer renders
    the dendrogram for interactive cutting
  repo: https://github.com/lukauskas/dgw
- name: Python
  role: Implementation language; scipy/fastcluster perform the complete-linkage agglomerative
    clustering whose dendrogram is cut
  repo: https://github.com/lukauskas/dgw
- name: fastcluster
  role: Fast computation of the complete-linkage hierarchical clustering dendrogram
    that is then cut
  repo: http://github.com/sauliusl/fastcluster
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:e92a804f87e74622b345de3c47bce7bba77362d48702d50ca2ac5d212ce9a346
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lukauskas/dgw
  tools:
  - DGW
  - Python
  - fastcluster
  - matplotlib
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hierarchical-dendrogram-cutting

## Summary

Cut agglomerative hierarchical clustering dendrograms (built with complete linkage over DTW-warped or raw epigenomic peak profiles) at an appropriate level to recover meaningful clusters of genomic regions, as used by DGW both in simulation validation (5 clusters) and interactive dgw-explorer analysis (45 clusters).

## When to use

Use when a dendrogram from complete-linkage agglomerative clustering of DTW-aligned peak profiles must be cut into clusters whose count reflects true underlying patterns or is chosen interactively near the leaves.

## When NOT to use

- The input peaks have not been DTW-warped and contain substantial noise/modifications (v or p above ~0.15) — the article shows standard hierarchical clustering without warping degrades badly (MCC ~0.294 at v=0.40, p=0.40)
- A fixed, validated cluster count is required but only a purely automatic cut with no ground truth or interactive inspection is possible — the article notes cutting the dendrogram 'at an appropriate level is a difficult choice'
- The data are already grouped by an external annotation (e.g. known TSS labels) and clustering would only duplicate existing structure

## Inputs

- Linkage matrix produced by complete-linkage agglomerative clustering (e.g. {prefix}_linkage.npy from DGW)
- Pairwise DTW distance matrix over binned (50 bp) ChIP-Seq peak coverage profiles
- Known number of ground-truth classes (for simulation, e.g. 5 seed patterns) OR interactive dendrogram view from dgw-explorer

## Outputs

- Cluster assignments for each genomic peak region at the chosen cut level (e.g. 5 clusters of ~100 members, or 45 clusters in dgw-explorer)
- Per-cluster groupings used to render dendrogram and heat-map windows and to export per-cluster BED files

## How to apply

First build the dendrogram by performing agglomerative hierarchical clustering with complete linkage (to avoid chaining) over the pairwise DTW distance matrix; DGW uses scipy/fastcluster for this and imposes strand-aware distances (forward and reversed comparisons) constrained by a Sakoe-Chiba band of width 12. Then decide the cut level: in the simulation study, cut into exactly 5 clusters to match the five seed patterns and score recovery with the multi-class Matthews Correlation Coefficient against seed-of-origin labels; in exploratory analysis with dgw-explorer, cut the dendrogram interactively at a level near the leaves (the paper reports a cut yielding 45 clusters as appropriate). Cutting too high merges distinct peak shapes (the article notes standard hierarchical clustering only succeeds when noise and modifications are small, e.g. v<0.15, p<0.15), while cutting near leaves preserves fine shape distinctions that DGW's warping exposes. Rationale: dendrogram cutting is acknowledged as 'a difficult choice', so prefer validation-driven cuts (known k) or interactive cuts guided by cluster prototypes and downstream POI histogram entropy decreases.

## Related tools

- **DGW** (Produces the DTW distance matrix and linkage hierarchy; dgw-explorer renders the dendrogram for interactive cutting) — https://github.com/lukauskas/dgw
- **Python** (Implementation language; scipy/fastcluster perform the complete-linkage agglomerative clustering whose dendrogram is cut) — https://github.com/lukauskas/dgw
- **fastcluster** (Fast computation of the complete-linkage hierarchical clustering dendrogram that is then cut) — http://github.com/sauliusl/fastcluster

## Examples

```
from scipy.cluster.hierarchy import linkage, fcluster; Z = linkage(D, method='complete'); labels = fcluster(Z, t=5, criterion='maxclust')
```

## Evaluation signals

- Simulation: multi-class MCC against seed-of-origin labels approaches 1.000 after cutting into 5 clusters (DGW ~1.000 for v up to 0.25, e.g. 1.000 vs 0.748 for no-DTW at v=0.25, p=0.25)
- DGW-cut clusters recover 5 groups with approximately 100 members each, corresponding to the five seed patterns
- In dgw-explorer, the interactive cut yielding 45 clusters should produce per-cluster heat-maps whose warped POI position histograms show reduced entropy (12.91% TSS, 7.72% FSS decrease in the selected cluster)
- Linkage matrix files ({prefix}_linkage.npy) load successfully and the dendrogram renders in dgw-explorer before cutting
- Cut count is consistent with the biological expectation (5 seeds in simulation; near-leaf cut for exploratory peak-shape separation)

## Limitations

- Cutting the dendrogram 'at an appropriate level is a difficult choice' — no fully automatic criterion is endorsed by the paper
- Without DTW alignment, complete-linkage hierarchical clustering recovers true structure only when added noise and modifications are small (v<0.15, p<0.15)
- Complete linkage is chosen specifically 'to avoid chaining'; other linkages may behave differently on warped epigenomic profiles

## Evidence

- [methods] Cutting the dendrogram at an appropriate level is a difficult choice.: "Cutting the dendrogram at an appropriate level is a difficult choice."
- [methods] DGW identifies 5 clusters with approximately 100 members each: "In contrast to standard hierarchical clustering DGW identifies 5 clusters with approximately 100 members each, corresponding well to the initial five seed patterns."
- [methods] perform agglomerative hierarchical clustering, using complete linkage to avoid chaining: "perform agglomerative hierarchical clustering, using complete linkage to avoid chaining"
- [other] cut the dendrogram interactively at a level near the leaves (the paper reports a cut yielding 45 clusters as appropriate): [evidence span withheld — anchor status: not_found]
- [methods] constrained the DTW with a Sakoe-Chiba Band of width 12: "constrained the DTW with a Sakoe-Chiba Band of width 12"
