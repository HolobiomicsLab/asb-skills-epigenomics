---
name: random-walk-with-restart-imputation
description: Use when a Hi-C contact matrix is extremely sparse (e.g., 0.025% downsampled
  or single-cell data) and needs imputation prior to structural-entropy TLD detection.
when_to_use_negative:
- When running deDoc2 on bulk or reasonably dense Hi-C data — deDoc2 'decodes TLD
  from single-cell Hi-C data with no need of data imputation', and the paper reports
  it is robust to various imputation algorithms
- When the goal is TLD detection itself rather than matrix preprocessing — RWR here
  is only an optional imputation step, not the domain caller
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3922
tools:
- name: deDoc2
  role: Hosts the optional RWR imputation module invoked with -rwr (restart probability
    set via -rp, default 0.5) before its structural-entropy TLD prediction
  repo: https://github.com/zengguangjie/deDoc2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:847eca3ab2815f66402b659f0e18f1e908f10f9627a6a4699b4d14404ed42b6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - deDoc2
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3922
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# random-walk-with-restart-imputation

## Summary

Optional preprocessing module of deDoc2 that imputes sparse single-cell or downsampled Hi-C contact matrices via random walk with restart (RWR) before TLD prediction, controlled by the -rwr flag and the -rp restart probability (default 0.5).

## When to use

Use when a Hi-C contact matrix is extremely sparse (e.g., 0.025% downsampled or single-cell data) and needs imputation prior to structural-entropy TLD detection.

## When NOT to use

- When running deDoc2 on bulk or reasonably dense Hi-C data — deDoc2 'decodes TLD from single-cell Hi-C data with no need of data imputation', and the paper reports it is robust to various imputation algorithms
- When the goal is TLD detection itself rather than matrix preprocessing — RWR here is only an optional imputation step, not the domain caller

## Inputs

- Sparse or dense intra-chromosome Hi-C contact matrix (n×n tab-split contact matrix format or deDoc sparse format: first line = number of bins, then 'from_bin to_bin edge_weight' lines)
- Restart probability α (default 0.5)

## Outputs

- Converged RWR-imputed contact matrix Qt (e.g., saved as rwr_imputed_matrix.npz)
- Imputed matrix fed forward into deDoc2.w / deDoc2.s TLD prediction (.TAD and .window.TAD outputs)

## How to apply

Load the sparse intra-chromosome contact matrix M (e.g., a downsampled matrix at 0.025% contacts, keeping only rows/columns with at least one nonzero entry) and normalize it into matrix C via sqrtVC normalization, which keeps C symmetric and guarantees convergence of the random walk. Initialize Q0 = I and iteratively apply the RWR recursion Qt = (1−α)·Qt−1·C + α·I with restart probability α (deDoc2 default 0.5, configurable with -rp). Stop when the squared 2-norm of successive iterates satisfies ‖Qt − Qt−1‖2 ≤ 10−2, indicating convergence. The imputed matrix Qt then serves as input to deDoc2.w/deDoc2.s TLD prediction. Note the paper emphasizes deDoc2 works robustly without imputation, so this step is optional.

## Related tools

- **deDoc2** (Hosts the optional RWR imputation module invoked with -rwr (restart probability set via -rp, default 0.5) before its structural-entropy TLD prediction) — https://github.com/zengguangjie/deDoc2

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40 -rwr -rp 0.5
```

## Evaluation signals

- Iteration terminates via the convergence check ‖Qt − Qt−1‖2 ≤ 10−2
- Normalized matrix C is verified to remain symmetric (required for RWR convergence per the paper's methods)
- Restart probability used equals the specified value (deDoc2 default 0.5 via -rp)
- Imputed matrix retains the dimensions of the original nonzero-row/column-filtered contact matrix and can be passed to deDoc2.w/deDoc2.s
- Output matrix contains no all-zero rows/columns that were present in the sparse input

## Limitations

- Convergence of the RWR recursion requires the sqrtVC-normalized matrix C to be symmetric; asymmetric inputs violate this guarantee
- Imputation is optional in deDoc2 — the authors state deDoc2 decodes TLDs 'with no need of data imputation', so imputation is not required for performance
- Bins with no contacts are restricted/removed before imputation (only rows/columns with at least one nonzero entry are retained)

## Evidence

- [methods] The random walk stops when ∥Qt − Qt−1∥2 ≤ 10−2, which means that the process has converged.: [evidence span withheld — anchor status: not_found]
- [methods] it is normalized into a matrix C by sqrtVC normalization,[50] which ensures that the resulting RWR imputation can converge: [evidence span withheld — anchor status: not_found]
- [readme] -rwr|Optional|whether to perform RWR imputation before TLD prediction|N/A|: "-rwr|Optional|whether to perform RWR imputation before TLD prediction|N/A"
- [readme] -rp|Optional|the restart probability of RWR imputation|0.5|: "-rp|Optional|the restart probability of RWR imputation|0.5"
- [discussion] deDoc2 decodes TLD from single‐cell Hi‐C data with no need of data imputation, while most state‐of‐the‐art TLD predictors do: "deDoc2 decodes TLD from single‐cell Hi‐C data with no need of data imputation, while most state‐of‐the‐art TLD predictors do"
