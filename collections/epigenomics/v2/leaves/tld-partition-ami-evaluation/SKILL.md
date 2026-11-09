---
name: tld-partition-ami-evaluation
description: Use when comparing TLD/TAD predictor outputs on single-cell Hi-C contact
  matrices and you need quantitative, per-predictor quality metrics (2D structural
  entropy, modularity, AMI) to rank methods on raw, non-imputed data.
when_to_use_negative:
- Input Hi-C data is bulk or already imputed/normalized — deDoc2 needs no normalization
  and this benchmark specifically targets raw single-cell data to test performance
  without imputation
- Predicted TLD counts are too sparse to compare (e.g., cells excluded when insufficient
  TLDs are identified, as with the '2n DNA' stage)
- You need fine-grained per-domain biological validation rather than genome-wide partition
  quality ranking
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3922
tools:
- name: deDoc2
  role: Primary TLD predictor under benchmark; deDoc2.w minimizes 2D structural entropy
    over the whole contact map, deDoc2.s over sliding windows; run via java -jar deDoc2.jar
    with -binsize 40
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Baseline structural-entropy TLD predictor (deDoc(E)/deDoc(M) communities);
    also provides onednSE.jar to compute 1D-nSE of the Hi-C graph
  repo: https://github.com/yinxc/structural-information-minimisation
- name: scHiCluster
  role: Imputation-based comparison TLD predictor run with default settings
- name: deTOKI
  role: NMF-based comparison TLD predictor run with default settings
- name: Higashi
  role: Comparison predictor run with nbr = 4, embedding_epoch = 1, no_nbr_epoch =
    20 on multicell datasets to reduce runtime
- name: HiCS
  role: Comparison predictor (HiCS_de variant) included in the benchmark
- name: scikit-learn
  role: Provides sklearn.metrics.adjusted_mutual_info_score for computing AMI between
    predicted TLD partitions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:6e565b95e27c5f12835be902e89a2bc7071d00a353cd7fdacc8fa9558a8ce052
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - HiCS
  - Higashi
  - deDoc
  - deDoc2
  - deTOKI
  - scHiCluster
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3922
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# tld-partition-ami-evaluation

## Summary

Benchmark TAD-like domain (TLD) predictions from multiple single-cell Hi-C predictors (deDoc2.w, deDoc2.s, deDoc, scHiCluster, deTOKI, Higashi, HiCS_de) on raw, non-imputed contact matrices by scoring 2D structural entropy, modularity, and adjusted mutual information (AMI). It quantifies which predictor yields well-defined TLDs without imputation.

## When to use

Use when comparing TLD/TAD predictor outputs on single-cell Hi-C contact matrices and you need quantitative, per-predictor quality metrics (2D structural entropy, modularity, AMI) to rank methods on raw, non-imputed data.

## When NOT to use

- Input Hi-C data is bulk or already imputed/normalized — deDoc2 needs no normalization and this benchmark specifically targets raw single-cell data to test performance without imputation
- Predicted TLD counts are too sparse to compare (e.g., cells excluded when insufficient TLDs are identified, as with the '2n DNA' stage)
- You need fine-grained per-domain biological validation rather than genome-wide partition quality ranking

## Inputs

- Raw (non-imputed) single-cell Hi-C intra-chromosome contact matrices (n x n tab-split matrix or sparse from_bin to_bin edge_weight format with bin count on the first line)
- TLD partition files (.TAD and .window.TAD outputs; one TLD per line, space-split bin ids numbering from 1, single bins removed as non-TAD regions)
- Dataset references: Tan's GM12878 scHi-C (16 cells), Nagano's mouse ES cell scHi-C (20 cells / 1171-cell dataset), Lee's human prefrontal cortex scHi-C (4238 cells)

## Outputs

- Per-predictor 2D structural entropy values for height-2 encoding trees of each contact matrix
- Per-predictor modularity index values (gap-region loci as singleton communities)
- Per-predictor adjusted mutual information (AMI) values computed with sklearn.metrics.adjusted_mutual_info_score
- Aggregated benchmark table tld_benchmark_results.csv

## How to apply

Load the raw single-cell Hi-C contact matrices (e.g., Lee's human prefrontal cortex, 4238 cells; Nagano's mouse ES cells, 1171 cells) and interpret each matrix as a weighted graph. Run all TLD predictors with default settings, except Higashi which is set to nbr = 4 with embedding_epoch = 1 and no_nbr_epoch = 20 for multicell datasets to reduce runtime. For each predictor's TLD calls, build a height-2 encoding tree per contact matrix — placing gap-region vertices directly under the root λ — and compute the 2D structural entropy index. Compute modularity after replacing zero diagonal elements and log2-processing remaining entries, treating each gap-region locus as a singleton community. Compute AMI with sklearn.metrics.adjusted_mutual_info_score, including all predicted TLDs and intermediate windows of TLDs in T and K, then aggregate results into tld_benchmark_results.csv.

## Related tools

- **deDoc2** (Primary TLD predictor under benchmark; deDoc2.w minimizes 2D structural entropy over the whole contact map, deDoc2.s over sliding windows; run via java -jar deDoc2.jar with -binsize 40) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Baseline structural-entropy TLD predictor (deDoc(E)/deDoc(M) communities); also provides onednSE.jar to compute 1D-nSE of the Hi-C graph) — https://github.com/yinxc/structural-information-minimisation
- **scHiCluster** (Imputation-based comparison TLD predictor run with default settings)
- **deTOKI** (NMF-based comparison TLD predictor run with default settings)
- **Higashi** (Comparison predictor run with nbr = 4, embedding_epoch = 1, no_nbr_epoch = 20 on multicell datasets to reduce runtime)
- **HiCS** (Comparison predictor (HiCS_de variant) included in the benchmark)
- **scikit-learn** (Provides sklearn.metrics.adjusted_mutual_info_score for computing AMI between predicted TLD partitions)

## Examples

```
java -jar deDoc2.jar -inputfile ./14.clean.con.chr18.matrix -binsize 40; python -c "from sklearn.metrics import adjusted_mutual_info_score; ami = adjusted_mutual_info_score(T, K)"
```

## Evaluation signals

- On experimental single-cell Hi-C datasets (Tan's and Nagano's), deDoc2.w and deDoc2.s should achieve the first and second smallest structural entropies, respectively
- deDoc2.w/HiCS_de should appear among the largest values for both structural entropy and modularity columns in tld_benchmark_results.csv on Lee's data
- deDoc2.s should show the highest AMI on Lee's dataset
- Output partition files must contain one TLD per line with space-split bin ids numbering from 1, and single-bin lines removed as non-TAD regions
- All gap-region loci must be handled as singleton communities in modularity and placed directly under the root λ in the height-2 encoding tree

## Limitations

- Higashi required non-default settings (nbr = 4, embedding_epoch = 1, no_nbr_epoch = 20) on multicell datasets purely to reduce runtime, so its results are not strictly comparable under default configuration
- No version numbers stated for deDoc2 or any compared tool; software version numbers were not recorded in the methods
- Cells/stages with insufficient TLDs (e.g., '2n DNA' stage) must be excluded from the analysis
- deDoc2 output marks single bins in one line as non-TAD regions that must be removed; gap-region handling affects the entropy and modularity calculations

## Evidence

- [results] On the experimental single-cell Hi-C datasets, deDoc2.w and deDoc2.s achieved the first and second smallest structural entropies, respectively, indicating well-defined TLD predictions without imputation.: [evidence span withheld — anchor status: not_found]
- [methods] workflow: compute AMI via sklearn including all predicted TLDs and intermediate windows: [evidence span withheld — anchor status: not_found]
- [results] The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher and lower level TLDs: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher and lower level TLDs"
- [methods] README parameter: Higashi settings on multicell datasets: "All test TLD predictors were run with default settings, except for Higashi"
- [readme] README output format of deDoc2 TLD partitions: "The output files contains partitions as TLD results. Each line represents a TLD, containg the node id of this TLD, space split, numbering from 1."
