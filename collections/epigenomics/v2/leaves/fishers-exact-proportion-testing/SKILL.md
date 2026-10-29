---
name: fishers-exact-proportion-testing
description: Use when two discrete cell samples have been clustered (e.g. k=5 ChromSCape
  clusters) and you need a significance test that the proportion of cells falling
  in a target cluster differs between the samples.
when_to_use_negative:
- Input is raw reads or an unfiltered count matrix — clustering via ChromSCape QC,
  normalization and PCA must precede any proportion test
- Comparing more than two samples or more than two cluster proportions — Fisher's
  exact test on a 2x2 table does not apply; use a k-way test instead
- Clusters are of single-cell origin rather than cell-type/phenotype labels and the
  question is clustering performance (use replicate assignment scores, not proportion
  tests)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ChromSCape
  role: Produces the k=5 cell clusters on the GSE99172 scATAC-seq count matrix whose
    per-sample membership proportions are compared with Fisher's exact test
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: R
  role: Language in which the Fisher's exact test is computed and recomputed metrics
    are written to CSV
- name: Bioconductor
  role: Distribution channel of the ChromSCape package used within the R analysis
    pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:88c8a84935c966163bf758c37f2abfcf000e4521f88cc5508f15c3c74586dcd9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/ChromSCape_benchmarking
  tools:
  - Bioconductor
  - ChromSCape
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# fishers-exact-proportion-testing

## Summary

Apply Fisher's exact test on 2x2 contingency tables to compare the proportion of cells assigned to a cluster (e.g. a monocyte cluster) between two patient-derived single-cell epigenomic samples, as in the ChromSCape analysis of the scATAC-seq GSE99172 benchmarking dataset.

## When to use

Use when two discrete cell samples have been clustered (e.g. k=5 ChromSCape clusters) and you need a significance test that the proportion of cells falling in a target cluster differs between the samples.

## When NOT to use

- Input is raw reads or an unfiltered count matrix — clustering via ChromSCape QC, normalization and PCA must precede any proportion test
- Comparing more than two samples or more than two cluster proportions — Fisher's exact test on a 2x2 table does not apply; use a k-way test instead
- Clusters are of single-cell origin rather than cell-type/phenotype labels and the question is clustering performance (use replicate assignment scores, not proportion tests)

## Inputs

- ChromSCape cluster assignment per cell (k=5 clusters) for two patient-derived samples (e.g. SU070 and SU353)
- Cell-level metadata mapping cells to samples/conditions
- Definition of the target cluster (e.g. monocyte cluster, or dominant cluster C1 after sub-clustering)

## Outputs

- 2x2 contingency table of cells by sample and target-cluster membership
- Fisher's exact test p-value (expected ~0.0025 for SU070 vs SU353)
- Metrics CSV (e.g. gse99172_chromscape_metrics.csv) containing the recomputed p-value with replicate assignment scores

## How to apply

After ChromSCape quality-control filtering (minimum 1600 unique reads per cell, removal of top 5% coverage cells, keeping regions detected in at least 1% of cells), PCA (50 PCs) and k=5 clustering, build a 2x2 contingency table counting cells in versus out of the target cluster (e.g. the monocyte cluster) for each patient-derived sample (SU070 vs SU353). Run R's fisher.test() on this table to obtain an exact p-value for the difference in proportions; the paper-reported value for SU070 vs SU353 is 0.0025. Optionally sub-cluster the dominant cluster (C1) before testing if the comparison requires finer resolution. Write the recomputed p-value alongside other metrics (K562 replicate score 99.7%, GM12878 98.3%, average correct-cluster assignment 99.4%) to a metrics CSV for validation against paper-reported values.

## Related tools

- **ChromSCape** (Produces the k=5 cell clusters on the GSE99172 scATAC-seq count matrix whose per-sample membership proportions are compared with Fisher's exact test) — https://github.com/vallotlab/ChromSCape_benchmarking
- **R** (Language in which the Fisher's exact test is computed and recomputed metrics are written to CSV)
- **Bioconductor** (Distribution channel of the ChromSCape package used within the R analysis pipeline)

## Examples

```
Rscript -e 'fisher.test(matrix(c(n_SU070_mono, n_SU070_other, n_SU353_mono, n_SU353_other), nrow=2))'
```

## Evaluation signals

- The recomputed Fisher's exact p-value for SU070 vs SU353 monocyte-cluster proportions matches the paper-reported 0.0025 within a small tolerance
- The contingency table row/column sums equal the post-QC cell counts of each sample (cells lost to the 1600-read minimum and top-5% coverage filters must not be counted)
- K562 and GM12878 technical-replicate assignment scores of 99.7% and 98.3% and average correct-cluster assignment of 99.4% are reproduced alongside the p-value in the output CSV
- All recomputed metrics are written to gse99172_chromscape_metrics.csv as specified by the workflow

## Limitations

- Fisher's exact test is only meaningful after reliable clustering; it inherits any errors in the upstream k=5 ChromSCape clustering and QC filtering
- Most R dependency versions are unreported in the paper, limiting environment reproducibility of the recomputation
- The tested comparison (monocyte cluster between SU070 and SU353) is dataset-specific; thresholds and expected p-values do not transfer to other datasets

## Evidence

- [methods] Fisher's exact test on monocyte-cluster proportions between SU070 and SU353: [evidence span withheld — anchor status: not_found]
- [results] Validation of recomputed metrics against paper values: [evidence span withheld — anchor status: not_found]
- [discussion] ChromSCape identifies sub-populations that the proportion test quantifies: "It also manages to identify sub-populations within a complex scATAC-seq benchmarking dataset"
