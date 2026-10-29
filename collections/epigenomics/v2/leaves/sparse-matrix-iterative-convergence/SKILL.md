---
name: sparse-matrix-iterative-convergence
description: Use when a very sparse Hi-C contact matrix (e.g., single-cell or downsampled
  to 0.025% sampling rate) needs imputation via RWR diffusion before downstream analysis,
  with symmetric sqrtVC normalization for convergence.
when_to_use_negative:
- The input is bulk or already dense Hi-C data — deDoc2 'needs no normalization process,
  raw Hi-C data is suitable' and decodes TLDs 'with no need of data imputation'.
- The contact matrix cannot be made symmetric — sqrtVC symmetry is required for the
  RWR to converge.
- You only need the TLD partition itself and do not require imputation, since deDoc2
  is robust to various data imputation algorithms.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: deDoc2
  role: Hosts the optional RWR imputation step, enabled with the -rwr flag and restart
    probability -rp (default 0.5), applied before deDoc2.w/deDoc2.s TLD prediction
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Predecessor tool whose sparse contact matrix input format (bins + weighted
    edge list) is reused for RWR input
  repo: https://github.com/yinxc/structural-information-minimisation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:7dee2c6e2db89c58a90bd304be143964a52ce49834523eea0fb7d50d4e7a8121
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - deDoc
  - deDoc2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# sparse-matrix-iterative-convergence

## Summary

Optional random-walk-with-restart (RWR) imputation preprocessing in deDoc2 that densifies sparse single-cell Hi-C contact matrices by iterative diffusion until a squared 2-norm convergence criterion (‖Qt − Qt−1‖2 ≤ 10−2) is met.

## When to use

Use when a very sparse Hi-C contact matrix (e.g., single-cell or downsampled to 0.025% sampling rate) needs imputation via RWR diffusion before downstream analysis, with symmetric sqrtVC normalization for convergence.

## When NOT to use

- The input is bulk or already dense Hi-C data — deDoc2 'needs no normalization process, raw Hi-C data is suitable' and decodes TLDs 'with no need of data imputation'.
- The contact matrix cannot be made symmetric — sqrtVC symmetry is required for the RWR to converge.
- You only need the TLD partition itself and do not require imputation, since deDoc2 is robust to various data imputation algorithms.

## Inputs

- Sparse intra-chromosome Hi-C contact matrix (n × n tab-split contact matrix format or deDoc sparse format: first line = number of bins, then 'from_bin to_bin edge_weight' lines)
- Restart probability α (default 0.5 via -rp, enabled with -rwr)

## Outputs

- Converged RWR-imputed dense contact matrix (e.g., saved as rwr_imputed_matrix.npz)
- Imputed matrix passed on to deDoc2.w / deDoc2.s TLD prediction when -rwr is enabled

## How to apply

Load the sparse Hi-C contact matrix M, retaining only rows/columns with at least one nonzero entry, and normalize it into C using sqrtVC normalization, which keeps C symmetric and guarantees convergence of the random walk. Initialize Q0 = I and iteratively apply the RWR recursion Qt = (1−α)·Qt−1·C + α·I, where α is the restart probability (default 0.5, configurable via the optional -rp parameter after enabling -rwr). Stop when the squared 2-norm of successive iterates satisfies ‖Qt − Qt−1‖2 ≤ 10−2, per the paper's convergence criterion. The imputed matrix Qt can then be used directly or saved (e.g., rwr_imputed_matrix.npz). In deDoc2 the whole step is optional: RWR imputation is toggled by -rwr and the restart probability defaults to 0.5, while raw Hi-C data without imputation remains suitable input.

## Related tools

- **deDoc2** (Hosts the optional RWR imputation step, enabled with the -rwr flag and restart probability -rp (default 0.5), applied before deDoc2.w/deDoc2.s TLD prediction) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Predecessor tool whose sparse contact matrix input format (bins + weighted edge list) is reused for RWR input) — https://github.com/yinxc/structural-information-minimisation

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40 -rwr -rp 0.5
```

## Evaluation signals

- Iteration terminates via the convergence check ‖Qt − Qt−1‖2 ≤ 10−2 rather than a fixed iteration count.
- The sqrtVC-normalized matrix C is verified to remain symmetric, as required for RWR convergence.
- Q0 is initialized to the identity matrix I, and the recursion Qt = (1−α)·Qt−1·C + α·I is applied with the specified restart probability (0.5 by default).
- The imputed output has the same dimensions as the filtered input matrix (rows/columns restricted to bins with ≥1 nonzero contact).
- With -rwr absent, deDoc2 still produces .TAD / .window.TAD partitions from the raw sparse input, confirming imputation is truly optional.

## Limitations

- RWR imputation is optional in deDoc2; the paper emphasizes that deDoc2 decodes TLDs from single-cell Hi-C data with no need of imputation, so imputation may be unnecessary for many inputs.
- The README specifies no normalization of the raw input; any pre-normalization beyond sqrtVC could affect the walk's convergence guarantees.
- Downsampled inputs at very low sampling rates (e.g., 0.025%) are the motivating use case; behavior on moderately dense matrices is not characterized by the convergence criterion here.

## Evidence

- [methods] The random walk stops when ∥Qt − Qt−1∥2 ≤ 10−2, which means that the process has converged.: [evidence span withheld — anchor status: not_found]
- [methods] it is normalized into a matrix C by sqrtVC normalization,[50] which ensures that the resulting RWR imputation can converge: [evidence span withheld — anchor status: not_found]
- [discussion] deDoc2 decodes TLD from single‐cell Hi‐C data with no need of data imputation, while most state‐of‐the‐art TLD predictors do: "deDoc2 decodes TLD from single‐cell Hi‐C data with no need of data imputation, while most state‐of‐the‐art TLD predictors do"
- [readme] -rp|Optional|the restart probability of RWR imputation|0.5: "Optional|the restart probability of RWR imputation|0.5"
- [readme] deDoc2 supports both contact matrix format and sparse format as in deDoc. deDoc2 needs no normalization process, raw Hi-C data is suitable.: "deDoc2 supports both contact matrix format and sparse format as in deDoc. deDoc2 needs no normalization process, raw Hi-C data is suitable."
