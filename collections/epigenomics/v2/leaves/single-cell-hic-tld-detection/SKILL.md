---
name: single-cell-hic-tld-detection
description: Use when raw (non-imputed) single-cell Hi-C contact matrices (e.g., Tan
  GM12878, Nagano mouse ES, Flyamer, Lee PFC datasets) need hierarchical TAD-like
  domain calls, and when benchmarking TLD predictors on sparse per-cell data.
when_to_use_negative:
- Input is bulk Hi-C with deep coverage where hierarchical TAD callers such as deDoc
  or SuperTAD suffice — deDoc2's sparsity robustness is the reason for choosing it
- The task requires imputation-dependent embedding pipelines as the primary method
  (e.g., Higashi embeddings for cell clustering) rather than direct entropy-based
  TLD calls
- Stages with too few detectable TLDs (e.g., the '2n DNA' stage excluded in the article)
  — insufficient TLDs make downstream hierarchy analysis invalid
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3420
- http://edamontology.org/topic_0602
tools:
- name: deDoc2
  role: 'Primary TLD detector: dynamic programming minimization of 2D structural entropy
    on the contact-map graph; deDoc2.w for whole-map and deDoc2.s for sliding-window
    partitions'
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Predecessor tool using high-dimensional structural entropy; baseline comparator
    and source of the sparse Hi-C graph input format and 2D-nSE/1D-nSE reporting
  repo: https://github.com/yinxc/structural-information-minimisation
- name: scHiCluster
  role: Imputation-based comparator TLD predictor run with default settings during
    benchmarking
- name: deTOKI
  role: NMF-based comparator TLD predictor run with default settings during benchmarking
- name: Higashi
  role: Embedding-based comparator; configured with nbr = 0 (simulated) or nbr = 4
    (multicell), embedding_epoch = 1, no_nbr_epoch = 20; TADs called via its call_tads
    function
- name: scikit-learn
  role: adjusted_mutual_info_score used to compare predicted TLD partitions against
    reference partitions
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:6337db6b6d8d33652e75bcce8fc263ba75e0ef6afb68b91e7ea20141b941d59a
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
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3420
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-hic-tld-detection

## Summary

Detect TAD-like domains (TLDs) in single-cell Hi-C contact matrices using deDoc2, a structural-information-theory tool that treats the contact map as a weighted graph and globally minimizes 2D structural entropy via dynamic programming, without data imputation.

## When to use

Use when raw (non-imputed) single-cell Hi-C contact matrices (e.g., Tan GM12878, Nagano mouse ES, Flyamer, Lee PFC datasets) need hierarchical TAD-like domain calls, and when benchmarking TLD predictors on sparse per-cell data.

## When NOT to use

- Input is bulk Hi-C with deep coverage where hierarchical TAD callers such as deDoc or SuperTAD suffice — deDoc2's sparsity robustness is the reason for choosing it
- The task requires imputation-dependent embedding pipelines as the primary method (e.g., Higashi embeddings for cell clustering) rather than direct entropy-based TLD calls
- Stages with too few detectable TLDs (e.g., the '2n DNA' stage excluded in the article) — insufficient TLDs make downstream hierarchy analysis invalid

## Inputs

- n×n intra-chromosome single-cell Hi-C contact matrix (tab-split, raw counts, e.g., 40 kb binsize)
- Sparse-format Hi-C graph: first line = number of bins, then 'from_bin to_bin edge_weight' lines
- Simulated scHi-C matrices from chr18:50–55 Mb 3D structure ensemble at 40 kb resolution (for benchmarking)

## Outputs

- deDoc2.w partition file (*.TAD): higher-level, larger TLDs, one TLD per line as space-split bin ids from 1
- deDoc2.s partition file (*.window.TAD): lower-level, smaller TLDs from 10 Mb sliding windows
- Optional NDI values for binsize determination; 2D-nSE printed by the underlying deDoc algorithm
- Benchmark table of per-predictor structural entropy, modularity, and AMI values (e.g., tld_benchmark_results.csv)

## How to apply

Interpret each n×n intra-chromosome single-cell contact matrix (default binsize 40 kb; raw contacts, no normalization required) as a weighted graph, storing it either as a tab-split contact matrix or a sparse edge list whose first line is the bin count. Run deDoc2.w, which applies dynamic programming over the whole contact map to find the partition with global minimal 2D structural entropy, and deDoc2.s, which applies the same optimization within 10 Mb sliding windows (maxTLDsize 10 Mb) to recover lower-level, smaller TLDs. Optionally enable RWR imputation (-rwr, restart probability rp = 0.5) or NDI-based binsize selection, but the core advantage is that imputation is unnecessary. Collect the .TAD (deDoc2.w) and .window.TAD (deDoc2.s) outputs, where each line is one TLD as space-split bin ids numbered from 1, removing single-bin lines as non-TAD regions. For benchmarking against deDoc, scHiCluster, deTOKI, Higashi, and HiCS, evaluate calls with 2D structural entropy (via a height-2 encoding tree), modularity, and adjusted mutual information computed with sklearn.metrics.adjusted_mutual_info_score.

## Related tools

- **deDoc2** (Primary TLD detector: dynamic programming minimization of 2D structural entropy on the contact-map graph; deDoc2.w for whole-map and deDoc2.s for sliding-window partitions) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Predecessor tool using high-dimensional structural entropy; baseline comparator and source of the sparse Hi-C graph input format and 2D-nSE/1D-nSE reporting) — https://github.com/yinxc/structural-information-minimisation
- **scHiCluster** (Imputation-based comparator TLD predictor run with default settings during benchmarking)
- **deTOKI** (NMF-based comparator TLD predictor run with default settings during benchmarking)
- **Higashi** (Embedding-based comparator; configured with nbr = 0 (simulated) or nbr = 4 (multicell), embedding_epoch = 1, no_nbr_epoch = 20; TADs called via its call_tads function)
- **scikit-learn** (adjusted_mutual_info_score used to compare predicted TLD partitions against reference partitions)

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40
```

## Evaluation signals

- On experimental single-cell datasets (Tan's, Nagano's), deDoc2.w and deDoc2.s should rank first and second by smallest structural entropy of predicted TLDs
- On simulated chr18:50–55 Mb data at D = 750 and D = 1000, deDoc2 should show the highest AMI and WS against the reference partition
- Output files: .TAD lines each contain ≥2 space-split bin ids; single-bin lines are removed as non-TAD regions; bin ids start from 1
- Input graph symmetry check: the contact matrix must be symmetric for the underlying deDoc graph algorithm
- Predicted partitions should be reproducible from the encoding-tree 2D structural entropy and AMI (with all predicted TLDs and intermediate windows included in T and K) recomputation

## Limitations

- deDoc2 relies on a Java environment (java -jar deDoc2.jar) and offers no installation/dependency details beyond the jar package in the README
- Cells or stages from which a sufficient number of TLDs cannot be identified (e.g., '2n DNA' stage) must be excluded from subsequent analysis
- Higashi comparisons require non-default runtime-reduction settings (embedding_epoch = 1, no_nbr_epoch = 20), and no version numbers are stated for deDoc2 or any compared tool
- SuperTAD requires orders of magnitude higher CPU time and memory for high-dimensional structural entropy, so it is impractical as a per-cell comparator (per Table S4, Supporting Information, not included in the document)

## Evidence

- [readme] deDoc2 treats contact map as weighted graph: "deDoc2 is a TAD-like domain(TLD) prediction tool using structural information theory, it treats the Hi-C contact map as a weighted graph"
- [readme] deDoc2.w and deDoc2.s predictors: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively."
- [readme] TAD output file format: "The output files contains partitions as TLD results. Each line represents a TLD, containg the node id of this TLD, space split, numbering from 1."
- [methods] Dynamic programming minimizes 2D structural entropy: "deDoc2 employed a dynamic programming algorithm to minimize 2D structural entropy."
- [discussion] No imputation needed: "deDoc2 decodes TLD from single‐cell Hi‐C data with no need of data imputation, while most state‐of‐the‐art TLD predictors do"
- [abstract] Outperforms state-of-the-art tools: "deDoc2 outperforms state‐of‐the‐art tools and is one of only two tools able to identify the hierarchy of TLDs in single cells"
