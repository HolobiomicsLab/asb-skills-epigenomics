---
name: tld-predictor-parameter-configuration
description: Use when benchmarking multiple TLD predictors on scHi-C contact matrices
  (real or simulated, e.g., D = 750/1000 thresholds) and you need reproducible, fairness-preserving
  parameter settings per tool, including simulated-data overrides for Higashi.
when_to_use_negative:
- Bulk Hi-C data with deep coverage — cell-level sparsity overrides (e.g., Higashi
  nbr = 0, no_nbr_epoch = 20) and deDoc2's no-imputation rationale apply to sparse
  single-cell inputs, not conventional population maps
- When the goal is TLD hierarchy analysis or downstream biology (cell-cycle dynamics,
  brain cortex cell types) rather than predictor configuration/benchmarking — deDoc2.w
  vs deDoc2.s level selection is out of scope here
- Input matrices already normalized or imputed — deDoc2 expects raw Hi-C data and
  performs its own optional RWR imputation (-rwr, rp = 0.5)
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: deDoc2
  role: Primary structural-entropy TLD predictor run with defaults (binsize 40 kb,
    windowsize 10 Mb, maxTLDsize 10 Mb); java -jar invocation, no normalization or
    imputation required
  repo: https://github.com/zengguangjie/deDoc2
- name: Higashi
  role: Embedding-based comparator configured on simulated data with nbr = 0, embedding_epoch
    = 1, no_nbr_epoch = 20; TADs called from embeddings via call_tads
- name: deTOKI
  role: NMF-based comparator run with default settings in the benchmark
- name: scHiCluster
  role: Imputation-based comparator run with default settings in the benchmark
- name: deDoc
  role: Predecessor structural-entropy comparator for bulk Hi-C, run with defaults
  repo: https://github.com/yinxc/structural-information-minimisation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:efc49d6b043863220e19b6231aab4acb43d33445f889a9483f29e9b3652c271a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - Higashi
  - deDoc
  - deDoc2
  - deTOKI
  - scHiCluster
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# tld-predictor-parameter-configuration

## Summary

Configures TAD-like domain (TLD) predictors (deDoc2, deDoc, deTOKI, scHiCluster, Higashi) for fair benchmarking on single-cell Hi-C contact matrices, e.g., ~100 simulated cells from the chr18:50–55 Mb 3D structure ensemble at 40 kb resolution. Key principle: run all predictors with default settings except for explicit, documented overrides on simulated data.

## When to use

Use when benchmarking multiple TLD predictors on scHi-C contact matrices (real or simulated, e.g., D = 750/1000 thresholds) and you need reproducible, fairness-preserving parameter settings per tool, including simulated-data overrides for Higashi.

## When NOT to use

- Bulk Hi-C data with deep coverage — cell-level sparsity overrides (e.g., Higashi nbr = 0, no_nbr_epoch = 20) and deDoc2's no-imputation rationale apply to sparse single-cell inputs, not conventional population maps
- When the goal is TLD hierarchy analysis or downstream biology (cell-cycle dynamics, brain cortex cell types) rather than predictor configuration/benchmarking — deDoc2.w vs deDoc2.s level selection is out of scope here
- Input matrices already normalized or imputed — deDoc2 expects raw Hi-C data and performs its own optional RWR imputation (-rwr, rp = 0.5)

## Inputs

- Raw intra-chromosome scHi-C contact matrix (n × n, tab-split) or sparse format (first line = bin count, then from_bin to_bin edge_weight lines), per single cell
- ~100 simulated single-cell Hi-C contact matrices at 40 kb resolution for chr18:50–55 Mb, generated with sampling weight Weight(i, j) = D − distance(i, j) at thresholds D = 750 and D = 1000

## Outputs

- Per-cell TLD partitions per predictor (e.g., deDoc2 .TAD and .window.TAD files, one TLD per line with space-split node ids numbered from 1)
- k-means cluster assignments from embeddings for embedding-based predictors (Higashi)
- Per-predictor, per-threshold (D = 750, D = 1000) AMI and WS benchmark score tables

## How to apply

Run each TLD predictor with its default settings, except Higashi on simulated data, which is configured with nbr = 0, embedding_epoch = 1, and no_nbr_epoch = 20 to reduce runtime, with TADs called from Higashi embeddings via its call_tads function. For deDoc2, treat the 40 kb binsize as the default (bin count matching the ~125 bins of the 5 Mb chr18:50–55 Mb region) and optionally use NDI (-NDI) to justify binsize choice; raw, unnormalized contact matrices are suitable input since deDoc2 needs no normalization. For embedding-based predictors (Higashi), cluster per-cell TLD representations with k-means on the reduced embedding to assign cell/domain partitions. Rationale: defaults avoid per-tool tuning bias, while the Higashi overrides (nbr=0, reduced epochs) acknowledge that neighbor-based imputation is inappropriate for sparse simulated data and keep the benchmark tractable.

## Related tools

- **deDoc2** (Primary structural-entropy TLD predictor run with defaults (binsize 40 kb, windowsize 10 Mb, maxTLDsize 10 Mb); java -jar invocation, no normalization or imputation required) — https://github.com/zengguangjie/deDoc2
- **Higashi** (Embedding-based comparator configured on simulated data with nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20; TADs called from embeddings via call_tads)
- **deTOKI** (NMF-based comparator run with default settings in the benchmark)
- **scHiCluster** (Imputation-based comparator run with default settings in the benchmark)
- **deDoc** (Predecessor structural-entropy comparator for bulk Hi-C, run with defaults) — https://github.com/yinxc/structural-information-minimisation

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40
```

## Evaluation signals

- Reproduced AMI ranking shows deDoc2 as the most accurate predictor (highest AMI and WS vs. reference) on simulated chr18:50–55 Mb data at both D = 750 and D = 1000
- Every predictor except Higashi used default settings — audit configs to confirm no undocumented parameter tuning
- Higashi run logs confirm nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20 on simulated data
- deDoc2 output files (.TAD, .window.TAD) contain valid partitions: one TLD per line, node ids space-split, numbered from 1; single-bin lines treated as non-TAD regions
- Adjusted mutual information computed with sklearn.metrics.adjusted_mutual_info_score over all predicted TLDs and intermediate windows in T and K

## Limitations

- Higashi needs non-default settings on simulated data (nbr = 0, reduced epochs) purely for runtime; embedding_epoch = 1 may under-train embeddings
- SuperTAD requires orders of magnitude higher CPU time and memory to solve high-dimensional structural entropy, so it is impractical for this benchmark
- deDoc2 output TLDs with a single bin per line must be removed as non-TAD regions; input matrix must be symmetric for the underlying deDoc graph format
- Software version numbers for deDoc2 and compared tools are not stated in the article

## Evidence

- [methods] All test TLD predictors were run with default settings, except for Higashi: "All test TLD predictors were run with default settings, except for Higashi"
- [results] Simulated data workflow: "We simulated Hi‐C data from a 3D chromosome structure ensemble containing ≈100 single cells"
- [readme] deDoc2 CLI defaults: ">> java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40"
- [readme] deDoc2 needs no normalization: "deDoc2 supports both contact matrix format and sparse format as in deDoc. deDoc2 needs no normalization process, raw Hi-C data is suitable."
- [results] Higashi default exception and runtime reduction: [evidence span withheld — anchor status: not_found]
