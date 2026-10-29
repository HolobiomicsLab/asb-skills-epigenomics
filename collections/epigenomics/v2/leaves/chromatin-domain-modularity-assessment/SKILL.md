---
name: chromatin-domain-modularity-assessment
description: Use when you need to evaluate or rank TLD/TAD predictions from a Hi-C
  matrix (bulk or single-cell) by 2D structural entropy and modularity indices, e.g.,
  to confirm a predictor's first-class performance beyond AMI/WS benchmarks.
when_to_use_negative:
- Input is already an imputed or normalized contact matrix — deDoc2 needs no normalization
  and modularity is computed on the raw matrix with zero diagonals replaced and log2-processing
  applied at assessment time
- Your goal is boundary-accuracy benchmarking only — AMI (sklearn.metrics.adjusted_mutual_info_score)
  and weighted similarity WS are the appropriate metrics, not entropy/modularity indices
- The TLD partition contains unresolved single-bin 'TLD' lines — these must be removed
  as non-TAD regions before building the encoding tree, otherwise entropy is misestimated
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: deDoc2
  role: Produces the TLD partitions (deDoc2.w whole-map and deDoc2.s sliding-window)
    whose 2D structural entropy and modularity are assessed; treats the Hi-C contact
    map as a weighted graph and minimizes 2D structural entropy by dynamic programming
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Baseline TLD predictor (high-dimensional structural entropy) whose partitions
    are scored with the same entropy/modularity indices for comparison
  repo: https://github.com/yinxc/structural-information-minimisation
- name: IS
  role: Competitor TLD predictor run with default settings to generate partitions
    for index comparison on downsampled data
- name: deTOKI
  role: Competitor NMF-based TLD predictor run with default settings for entropy/modularity
    comparison
- name: SpectralTAD
  role: Competitor TLD predictor run with default settings for entropy/modularity
    comparison
- name: GRiNCH
  role: Competitor TLD predictor run with default settings for entropy/modularity
    comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:baa3682371a5d7dd8b49eeafa276f4199b4407b3ebcecd1e61703ac92d51a464
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# chromatin-domain-modularity-assessment

## Summary

Assess the quality of predicted TAD-like domains (TLDs) by computing the 2D structural entropy of a height-2 encoding tree built from each predictor's TLDs, and the modularity of the Hi-C contact matrix after replacing zero diagonal elements and log2-processing the remaining entries. This provides entropy/modularity index values that complement boundary-accuracy metrics like AMI and WS when comparing TLD predictors.

## When to use

Use when you need to evaluate or rank TLD/TAD predictions from a Hi-C matrix (bulk or single-cell) by 2D structural entropy and modularity indices, e.g., to confirm a predictor's first-class performance beyond AMI/WS benchmarks.

## When NOT to use

- Input is already an imputed or normalized contact matrix — deDoc2 needs no normalization and modularity is computed on the raw matrix with zero diagonals replaced and log2-processing applied at assessment time
- Your goal is boundary-accuracy benchmarking only — AMI (sklearn.metrics.adjusted_mutual_info_score) and weighted similarity WS are the appropriate metrics, not entropy/modularity indices
- The TLD partition contains unresolved single-bin 'TLD' lines — these must be removed as non-TAD regions before building the encoding tree, otherwise entropy is misestimated

## Inputs

- Intra-chromosome Hi-C contact matrix (n × n, tab-separated) or sparse format (first line = number of bins; subsequent lines = from_bin to_bin edge_weight), raw/unnormalized, default binsize 40 kb
- TLD prediction partitions per tool (e.g., deDoc2 .TAD and .window.TAD output files: one TLD per line with space-split node ids numbered from 1; single-bin lines removed as non-TAD regions)

## Outputs

- 2D structural entropy (2D-nSE) value per contact matrix / TLD partition
- Modularity value per log2-processed contact matrix
- Per-predictor entropy/modularity index table across sampling rates (0.1%, 0.05%, 0.025%) or single cells for ranking

## How to apply

For each TLD prediction (e.g., from deDoc2.w, deDoc2.s, deDoc, IS, deTOKI, SpectralTAD, or GRiNCH), build a height-2 encoding tree over the Hi-C contact map treated as a weighted graph, placing gap-region vertices (non-TAD bins) directly on the root λ, and compute the 2D structural entropy of the tree. For modularity, first replace zero diagonal elements of the contact matrix, then log2-process the remaining entries before computing the modularity score. Apply this on both reference (raw or bulk) data and challenging inputs — downsampled Rao et al. bulk Hi-C at 0.1%, 0.05%, and 0.025% sampling rates with binsize 40 kb, or experimental scHi-C data (e.g., 16 GM12878 cells from Tan's dataset) — so that entropy and modularity can be compared across predictors at matched sparsity. Correct application means the target predictor shows low structural entropy and high modularity relative to competitors, consistent with its AMI/WS ranking.

## Related tools

- **deDoc2** (Produces the TLD partitions (deDoc2.w whole-map and deDoc2.s sliding-window) whose 2D structural entropy and modularity are assessed; treats the Hi-C contact map as a weighted graph and minimizes 2D structural entropy by dynamic programming) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Baseline TLD predictor (high-dimensional structural entropy) whose partitions are scored with the same entropy/modularity indices for comparison) — https://github.com/yinxc/structural-information-minimisation
- **IS** (Competitor TLD predictor run with default settings to generate partitions for index comparison on downsampled data)
- **deTOKI** (Competitor NMF-based TLD predictor run with default settings for entropy/modularity comparison)
- **SpectralTAD** (Competitor TLD predictor run with default settings for entropy/modularity comparison)
- **GRiNCH** (Competitor TLD predictor run with default settings for entropy/modularity comparison)

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40
```

## Evaluation signals

- 2D structural entropy of the height-2 encoding tree (gap-region vertices on root λ) is computed without error on every predictor's partition, and isolated-vertex self-loop value r = 2m/n0 is respected for sparse downsampled inputs
- Modularity is computed on a matrix where zero diagonal elements were replaced and remaining entries were log2-processed — verify the pre-processing was applied, otherwise values are incomparable
- On downsampled Rao et al. data (0.1%/0.05%/0.025%, 40 kb binsize), deDoc2.w should show low structural entropy and high modularity relative to deDoc, IS, deTOKI, scHiCluster, Higashi, SpectralTAD, and GRiNCH
- Entropy/modularity ranking should be consistent with the AMI/WS benchmark in which deDoc2.w achieved the highest scores at every sampling rate
- Per-line TLD outputs containing a single bin are excluded (non-TAD regions) before index computation

## Limitations

- SuperTAD requires several orders of magnitude higher CPU time and memory space to solve high-dimensional structural entropy, making whole-map entropy optimization expensive for large matrices
- Single bins reported in one line of deDoc2 output must be removed as non-TAD regions; naive index computation over them misestimates entropy/modularity
- The paper does not state software version numbers for the compared tools; all predictors except Higashi were run with default settings, and Higashi used nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20 — deviations change index comparability
- The paper did not report accession IDs or version metadata for the downsampled/simulated datasets in all sections, so reproducing the exact index values may require additional data provenance

## Evidence

- [results] We tested the structural entropy and modularity of TLDs on 16 GM12878 cells from Tan's dataset: "We tested the structural entropy and modularity of TLDs on 16 GM12878 cells from Tan's dataset"
- [methods] deDoc2 employs dynamic programming to minimize 2D structural entropy: "deDoc2 employed a dynamic programming algorithm to minimize 2D structural entropy."
- [readme] deDoc2 treats the Hi-C contact map as a weighted graph using structural information theory: "deDoc2 is a TAD-like domain(TLD) prediction tool using structural information theory, it treats the Hi-C contact map as a weighted graph"
- [discussion] SuperTAD high computational cost for structural entropy: "SuperTAD requires several orders of magnitude higher CPU time and memory space to solve high‐dimensional structural entropy (Table S4, Supporting Information)"
