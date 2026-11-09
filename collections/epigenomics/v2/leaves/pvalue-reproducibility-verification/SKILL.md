---
name: pvalue-reproducibility-verification
description: Use when a benchmarking claim of statistical significance over matched
  tasks (e.g., reported paired t-test p-values on Accuracy) must be independently
  recomputed and validated against published values.
when_to_use_negative:
- The task is running cell type prediction itself rather than verifying published
  significance statistics — use the Cellcano package pipeline instead
- Accuracy values are not task-paired (different task sets per method), since a paired
  t-test requires matched tasks across methods
- Only unsupervised clustering comparisons are involved, which this paper's 50-task
  benchmark over ACTINN/scJoint/Signac does not cover
edam_operation: http://edamontology.org/operation_2239
tools:
- name: Cellcano
  role: Method whose accuracy values across the 50 celltyping tasks are compared against
    ACTINN, scJoint, and Signac in the paired t-tests
  repo: https://marvinquiet.github.io/Cellcano/
- name: scipy.stats.ttest_rel
  role: Two-sided paired t-test used to recompute p-values from matched per-task accuracies
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:d274fdeedb2f2147d98ac47d27ef2961218ff9bde29e727c19daf37b607d6a06
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cellcano
  - scipy.stats.ttest_rel
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pvalue-reproducibility-verification

## Summary

Reproduces and verifies reported statistical significance results (paired t-test p-values) by recomputing them from per-task accuracy values across the 50 celltyping tasks comparing Cellcano against ACTINN, scJoint, and Signac.

## When to use

Use when a benchmarking claim of statistical significance over matched tasks (e.g., reported paired t-test p-values on Accuracy) must be independently recomputed and validated against published values.

## When NOT to use

- The task is running cell type prediction itself rather than verifying published significance statistics — use the Cellcano package pipeline instead
- Accuracy values are not task-paired (different task sets per method), since a paired t-test requires matched tasks across methods
- Only unsupervised clustering comparisons are involved, which this paper's 50-task benchmark over ACTINN/scJoint/Signac does not cover

## Inputs

- Source Data spreadsheet with per-task Accuracy values for all 50 celltyping tasks (columns for Cellcano, ACTINN, scJoint, Signac)
- Reported p-values to verify: 4.857e-3 (ACTINN), 1.645e-3 (scJoint), 0.023 (Signac)

## Outputs

- paired_ttest_pvalues.csv with one row per comparator method (ACTINN, scJoint, Signac) and its recomputed two-sided paired t-test p-value

## How to apply

Load the per-task accuracy values for all 50 celltyping tasks from the paper's Source Data spreadsheet, extracting the accuracy columns for Cellcano, ACTINN, scJoint, and Signac. For each comparator, run a two-sided paired t-test using scipy.stats.ttest_rel between Cellcano's accuracies and the comparator's accuracies across the 50 matched tasks (pairing is by task, so row order must be preserved). Write the method names and recomputed p-values to paired_ttest_pvalues.csv. Validate that each recomputed p-value matches the reported value within tolerance: p = 4.857e-3 for ACTINN, p = 1.645e-3 for scJoint, and p = 0.023 for Signac; discrepancies indicate mismatches in task pairing or extracted accuracy columns.

## Related tools

- **Cellcano** (Method whose accuracy values across the 50 celltyping tasks are compared against ACTINN, scJoint, and Signac in the paired t-tests) — https://marvinquiet.github.io/Cellcano/
- **scipy.stats.ttest_rel** (Two-sided paired t-test used to recompute p-values from matched per-task accuracies)

## Examples

```
from scipy.stats import ttest_rel; import pandas as pd; df = pd.read_csv('source_data_50tasks.csv'); print(ttest_rel(df['Cellcano'], df['ACTINN']))
```

## Evaluation signals

- Recomputed p-value for Cellcano vs ACTINN equals 4.857e-3 within tolerance
- Recomputed p-value for Cellcano vs scJoint equals 1.645e-3 within tolerance
- Recomputed p-value for Cellcano vs Signac equals 0.023 within tolerance
- paired_ttest_pvalues.csv exists and contains all three comparator rows
- Each t-test uses exactly 50 matched task pairs (n = 50 per method)

## Limitations

- The Source Data spreadsheet must contain all 50 per-task accuracies for every method; missing or reordered rows break the task pairing
- Tolerance matching depends on the article reporting p-values rounded to the printed precision (e.g., 0.023 for Signac), so exact equality should not be expected
- No specific versions of the compared tools (Seurat, Signac, scJoint) are reported, which may limit exact reproduction of the underlying accuracies

## Evidence

- [discussion] We have shown in 50 celltyping tasks with data from two systems (human PBMCs and mouse brain) that Cellcano significantly outperforms other celltyping methods: "We have shown in 50 celltyping tasks with data from two systems (human PBMCs and mouse brain) that Cellcano significantly outperforms other celltyping methods and integration with label transfer"
- [intro] we demonstrate that Cellcano is significantly more accurate, computationally efficient, and scalable compared to existing methods: "we demonstrate that Cellcano is significantly more accurate, computationally efficient, and scalable compared to existing methods."
- [other] For each comparator (ACTINN, scJoint, Signac), run a two-sided paired t-test (scipy.stats.ttest_rel): [evidence span withheld — anchor status: not_found]
- [readme] Cellcano requires the following: python (3.8 recommended), R, tensorflow (2.7.1), anndata (0.7.4), scanpy (1.8.2), numpy (1.19.2): "Cellcano requires the following:

- [python](https://www.python.org/) (3.8 recommended)
- [R](https://www.r-project.org/)
- [tensorflow](https://www.tensorflow.org/) (2.7.1)
-"
