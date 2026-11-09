---
name: multiclass-mcc-evaluation
description: Use when clustering outputs must be scored against known ground-truth
  class labels (e.g. 5 simulated peak seed patterns of ~100 members each) across a
  parameter grid, and a single balanced score per condition is needed.
when_to_use_negative:
- Clustering real data with no ground-truth class membership (e.g. the real ENCODE
  H3K4me3/H3K9ac K562 peaks) — MCC requires known seed-of-origin labels
- Binary classification tasks where standard binary MCC or balanced accuracy suffices
  — this skill targets multi-class cluster recovery
- Evaluating alignment quality directly (e.g. TSS/FSS entropy reduction after warping)
  rather than recovery of cluster structure
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DGW
  role: Produces the DTW-based complete-linkage dendrograms whose 5-cluster cuts are
    scored with multiclass MCC in the DGW-vs-baseline comparison
  repo: github.com/lukauskas/dgw
- name: fastcluster
  role: Generates the agglomerative complete-linkage dendrograms (with and without
    DTW distances) that are cut into 5 clusters before MCC computation
  repo: github.com/sauliusl/fastcluster
- name: Python
  role: Implementation language used to cut dendrograms (scipy/fastcluster), compute
    MCC values, and assemble mcc_simulation_table.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:f5d7888137bfdcf01b179201430a6a6a974795d2afcd597bd0e06615ebd884dc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DGW
  - Python
  - fastcluster
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# multiclass-mcc-evaluation

## Summary

Scores cluster recovery by computing the multi-class Matthews Correlation Coefficient of dendrogram-cut cluster assignments against known seed-of-origin labels. In the DGW paper this is the metric that quantifies how much DTW-based alignment improves recovery of simulated epigenomic peak classes over standard hierarchical clustering.

## When to use

Use when clustering outputs must be scored against known ground-truth class labels (e.g. 5 simulated peak seed patterns of ~100 members each) across a parameter grid, and a single balanced score per condition is needed.

## When NOT to use

- Clustering real data with no ground-truth class membership (e.g. the real ENCODE H3K4me3/H3K9ac K562 peaks) — MCC requires known seed-of-origin labels
- Binary classification tasks where standard binary MCC or balanced accuracy suffices — this skill targets multi-class cluster recovery
- Evaluating alignment quality directly (e.g. TSS/FSS entropy reduction after warping) rather than recovery of cluster structure

## Inputs

- Simulated peak datasets (5 seeds × 99 simulated peaks per seed) with known seed-of-origin labels
- Dendrograms from DGW (DTW distances, complete-linkage agglomerative clustering) and from standard hierarchical clustering without DTW on the same datasets
- Grid of multiplicative Gaussian noise variance v (0.00–0.40) and bin modification probability p (0.00–0.40) at which clustering was run

## Outputs

- MCC value per method (DGW vs no-DTW) per (v, p) grid point, assembled into mcc_simulation_table.csv indexed by p and v

## How to apply

Treat the seed pattern each simulated peak was derived from (5 seeds × 99 peaks generated with multiplicative Gaussian noise variance v ∈ 0.00–0.40, bin deletion/duplication probability p ∈ 0.00–0.40, and orientation flip probability fp=0.1) as ground-truth class labels. For each (v, p) grid point, cut both dendrograms — the DGW tree built from DTW pairwise distances with complete linkage, and the standard hierarchical (no-DTW) tree — into 5 clusters matching the five seed patterns. Compute the multi-class Matthews Correlation Coefficient of each cluster assignment against the seed-of-origin labels. Assemble the MCC values for both methods into a comparison table indexed by p and v and save it as mcc_simulation_table.csv. The rationale is that MCC is a balanced single-number score that penalises both over-splitting and merging, cleanly separating the regime where standard hierarchical clustering succeeds only for small noise and modifications (v<0.15, p<0.15) from the regime where DGW's warping keeps recovery near-perfect.

## Related tools

- **DGW** (Produces the DTW-based complete-linkage dendrograms whose 5-cluster cuts are scored with multiclass MCC in the DGW-vs-baseline comparison) — github.com/lukauskas/dgw
- **fastcluster** (Generates the agglomerative complete-linkage dendrograms (with and without DTW distances) that are cut into 5 clusters before MCC computation) — github.com/sauliusl/fastcluster
- **Python** (Implementation language used to cut dendrograms (scipy/fastcluster), compute MCC values, and assemble mcc_simulation_table.csv)

## Examples

```
from scipy.cluster.hierarchy import fcluster; cut = fcluster(Z, 5, criterion='maxclust')  # cut dendrogram into 5 clusters for MCC scoring against seed-of-origin labels
```

## Evaluation signals

- DGW attains near-perfect recovery: MCC ≈ 1.000 for v up to 0.25 (e.g. 1.000 for DGW vs 0.748 for no-DTW at v=0.25, p=0.25)
- Standard hierarchical clustering degrades under heavy perturbation: MCC ≈ 0.294 at v=0.40, p=0.40, consistent with Table 1 of the paper
- At low perturbation (v<0.15, p<0.15) both methods should score highly, since standard hierarchical clustering succeeds when noise and modifications are small
- When recovery is correct, each of the 5 clusters contains approximately 100 members matching one seed pattern
- Output table must be a grid indexed by p and v over 0.00–0.40 with an MCC column for each method, saved as mcc_simulation_table.csv

## Limitations

- Requires ground-truth labels, so it applies only to the simulated-peak benchmark; on real ENCODE data no such labels exist and MCC cannot be computed
- The comparison is only valid if both dendrograms are cut into the same number of clusters (5, matching the five seed patterns); a wrong cut invalidates the MCC comparison
- Reported MCC ranges depend on the simulation parameters used (Gaussian noise v, bin deletion/duplication p, orientation flip fp=0.1); different perturbation regimes shift the degradation point

## Evidence

- [methods] compute the multi-class Matthews Correlation Coefficient against the seed-of-origin labels for each (v, p) grid point: [evidence span withheld — anchor status: not_found]
- [results] DGW near-perfect MCC up to v=0.25: [evidence span withheld — anchor status: not_found]
- [methods] Standard hierarchical clustering only succeeds for small noise/modifications: "Standard Hierarchical clustering is able to correctly group the simulated peaks according to the pattern they are originally derived from only if the added noise and modifications are small"
- [methods] DGW recovers the 5 seed patterns as ~100-member clusters: "In contrast to standard hierarchical clustering DGW identifies 5 clusters with approximately 100 members each, corresponding well to the initial five seed patterns."
- [results] Baseline degrades to MCC 0.294 at extreme perturbation, matching Table 1: [evidence span withheld — anchor status: not_found]
