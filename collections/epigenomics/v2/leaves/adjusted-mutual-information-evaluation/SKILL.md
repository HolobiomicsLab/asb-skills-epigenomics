---
name: adjusted-mutual-information-evaluation
description: Use when comparing TLD/TAD partitions predicted by multiple algorithms
  (e.g., deDoc2, deTOKI, scHiCluster, Higashi) on downsampled, simulated, or single-cell
  Hi-C matrices at 40 kb binsize and you need an accuracy ranking.
when_to_use_negative:
- The task is TLD prediction itself rather than benchmarking — use deDoc2.w/deDoc2.s
  directly on the contact matrix instead of computing comparison metrics
- Predictions and references come from different genomic regions or binsizes (e.g.,
  mixing 40 kb with other resolutions), making the bin-level partition comparison
  invalid
- No reference (raw-data or ground-truth) TLD partition exists — AMI/WS are undefined
  without a reference partition in K
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: deDoc2
  role: Primary TLD predictor under benchmark (deDoc2.w whole-map and deDoc2.s sliding-window
    variants); its predictions form T in AMI/WS comparisons
  repo: https://github.com/zengguangjie/deDoc2
- name: scikit-learn
  role: Provides adjusted_mutual_info_score used to compute AMI between predicted
    (T) and reference (K) partitions
- name: deDoc
  role: Competitor structural-entropy TLD predictor run with default settings in the
    benchmark
  repo: https://github.com/yinxc/structural-information-minimisation
- name: Higashi
  role: Embedding-based competitor configured with nbr=0, embedding_epoch=1, no_nbr_epoch=20;
    TLDs called from embeddings via call_tads, then k-means clustering
- name: deTOKI
  role: Competitor NMF-based TLD predictor run with default settings
- name: scHiCluster
  role: Competitor imputation-based TLD predictor run with default settings
- name: SpectralTAD
  role: Competitor TLD predictor run with default settings on downsampled bulk data
- name: GRiNCH
  role: Competitor TLD predictor run with default settings on downsampled bulk data
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:227ff8f2b9542a5aece667f885479b045595328605f9a2bee2c1e9f547815359
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - GRiNCH
  - Higashi
  - IS
  - SpectralTAD
  - deDoc
  - deDoc2
  - deTOKI
  - scHiCluster
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# adjusted-mutual-information-evaluation

## Summary

Benchmark protocol for scoring TAD-like domain (TLD) predictions from sparse or single-cell Hi-C by computing adjusted mutual information (AMI) between predicted and reference partitions, complemented by weighted similarity (WS) against raw-data TLDs.

## When to use

Use when comparing TLD/TAD partitions predicted by multiple algorithms (e.g., deDoc2, deTOKI, scHiCluster, Higashi) on downsampled, simulated, or single-cell Hi-C matrices at 40 kb binsize and you need an accuracy ranking.

## When NOT to use

- The task is TLD prediction itself rather than benchmarking — use deDoc2.w/deDoc2.s directly on the contact matrix instead of computing comparison metrics
- Predictions and references come from different genomic regions or binsizes (e.g., mixing 40 kb with other resolutions), making the bin-level partition comparison invalid
- No reference (raw-data or ground-truth) TLD partition exists — AMI/WS are undefined without a reference partition in K

## Inputs

- Predicted TLD partitions (one TLD per line, space-separated bin IDs numbered from 1, .TAD / .window.TAD outputs)
- Reference TLD partitions (from raw bulk Hi-C, e.g., Rao et al., or ground-truth partitions from simulated chr18:50–55 Mb ensemble)
- Downsampled bulk Hi-C contact matrices at 0.1%/0.05%/0.025% sampling rates, 40 kb binsize
- Simulated ~100 single-cell Hi-C contact matrices (chr18:50–55 Mb, Weight(i,j) = D − distance(i,j))
- Reduced embeddings for embedding-based predictors (Higashi, scHiCluster) for k-means partitioning

## Outputs

- AMI score table per predictor per sampling rate / simulation threshold (sklearn adjusted_mutual_info_score)
- Weighted similarity (WS) scores comparing raw-data vs downsampled-data TLDs (intermediate windows excluded)
- 2D structural entropy and modularity values per predictor's TLD partition
- Final predictor ranking (expected: deDoc2 highest AMI and WS at every tested condition)

## How to apply

Represent each Hi-C contact map as a weighted graph and let each predictor output its partition; for embedding-based predictors (Higashi with nbr=0, embedding_epoch=1, no_nbr_epoch=20), cluster the reduced embedding with k-means to obtain per-cell partitions. Compute AMI with sklearn.metrics.adjusted_mutual_info_score, placing predicted TLDs in T and reference TLDs in K, and include all predicted TLDs plus intermediate windows in both sets. For the WS metric, place predicted TLDs from raw data in T and those from downsampled data in K, excluding intermediate windows from both. Optionally complement with the 2D structural entropy index (height-2 encoding tree, gap-region vertices placed directly on root λ) and modularity computed after replacing zero diagonal elements and log2-processing remaining entries. Rank predictors by AMI/WS per sampling rate (0.1%, 0.05%, 0.025%) or simulation threshold (D = 750, 1000); deDoc2.w (TLD length capped at 10 Mb) and deDoc2.s (10 Mb sliding windows) should top the ranking if the benchmark reproduces the paper's finding.

## Related tools

- **deDoc2** (Primary TLD predictor under benchmark (deDoc2.w whole-map and deDoc2.s sliding-window variants); its predictions form T in AMI/WS comparisons) — https://github.com/zengguangjie/deDoc2
- **scikit-learn** (Provides adjusted_mutual_info_score used to compute AMI between predicted (T) and reference (K) partitions)
- **deDoc** (Competitor structural-entropy TLD predictor run with default settings in the benchmark) — https://github.com/yinxc/structural-information-minimisation
- **Higashi** (Embedding-based competitor configured with nbr=0, embedding_epoch=1, no_nbr_epoch=20; TLDs called from embeddings via call_tads, then k-means clustering)
- **deTOKI** (Competitor NMF-based TLD predictor run with default settings)
- **scHiCluster** (Competitor imputation-based TLD predictor run with default settings)
- **SpectralTAD** (Competitor TLD predictor run with default settings on downsampled bulk data)
- **GRiNCH** (Competitor TLD predictor run with default settings on downsampled bulk data)

## Examples

```
from sklearn.metrics import adjusted_mutual_info_score; ami = adjusted_mutual_info_score(T, K)
```

## Evaluation signals

- deDoc2.w achieves AMI and WS at or above every competitor (deDoc, IS, deTOKI, scHiCluster, SpectralTAD, GRiNCH, Higashi) at each of the 0.1%/0.05%/0.025% sampling rates
- All predicted TLDs and intermediate windows are present in both T and K for AMI; intermediate windows are excluded from both for WS — verify the inclusion/exclusion convention matches per metric
- Favorable index values accompany top AMI/WS: low 2D structural entropy and high modularity for deDoc2 TLDs
- AMI values lie in a sane range (near 0 for unrelated partitions, approaching 1 for agreement with reference); scores computed only after mapping partitions onto the same bin coordinates
- Predictions reproduced on simulated chr18:50–55 Mb data at D = 750 and D = 1000 show the same top-ranked predictor (deDoc2) as the published benchmark

## Limitations

- Higashi required non-default settings (nbr=0, embedding_epoch=1, no_nbr_epoch=20) to reduce runtime on downsampled and simulated data, so absolute scores are configuration-dependent
- The paper's discussion section omits software version numbers, quantitative metric values, and supporting tables (e.g., Table S4), so exact reproduced values cannot be checked from the main text alone
- Cells with insufficient identified TLDs (e.g., the '2n DNA' stage) must be excluded, which affects the comparison set
- WS requires paired raw/downsampled predictions on the same region, so it cannot be computed for datasets lacking a raw-data reference

## Evidence

- [other] Compute adjusted mutual information between predicted and reference TLDs using sklearn.metrics.adjusted_mutual_info_score, including all predicted TLDs and intermediate windows in T and K.: [evidence span withheld — anchor status: not_found]
- [other] Compute the weighted similarity WS by placing predicted TLDs from raw data in T and those from downsampled data in K, excluding intermediate windows from both sets.: [evidence span withheld — anchor status: not_found]
- [methods] All test TLD predictors were run with default settings, except for Higashi: "All test TLD predictors were run with default settings, except for Higashi"
- [readme] The output files contains partitions as TLD results. Each line represents a TLD, containg the node id of this TLD, space split, numbering from 1.: "The output files contains partitions as TLD results. Each line represents a TLD, containg the node id of this TLD, space split, numbering from 1."
- [results] we generated a series of downsampled data from Rao et al., with sampling rates of 0.1%, 0.05%, and 0.025%: [evidence span withheld — anchor status: not_found]
