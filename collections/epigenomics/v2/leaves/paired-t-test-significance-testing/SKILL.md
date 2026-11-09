---
name: paired-t-test-significance-testing
description: Use when you have per-task performance values (e.g., Accuracy) for two
  methods evaluated on the same matched set of benchmark tasks and need to show one
  method's advantage is statistically significant, not just numerically higher.
when_to_use_negative:
- Performance values are not paired across the same tasks/datasets for both methods
  (independent samples instead of matched per-task accuracies)
- Input is aggregated summary statistics (mean accuracy only) rather than per-task
  values, since pairing across 50 tasks is required
- The goal is cross-modality celltyping (e.g., scRNA-seq reference for scATAC-seq)
  performance, which the article reports as planned but not implemented
tools:
- name: Cellcano
  role: Method whose per-task accuracies are tested against comparators in the paired
    t-tests
  repo: https://marvinquiet.github.io/Cellcano/
- name: scJoint
  role: Comparator method; its 50-task accuracies are paired against Cellcano's for
    the t-test
- name: Signac
  role: Comparator method; its 50-task accuracies are paired against Cellcano's for
    the t-test
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:8a4c329d0bf525b8a88b3fa60a02e271c536bdb4e787dd0d36cbcb374454ea93
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cellcano
  - Signac
  - scJoint
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# paired-t-test-significance-testing

## Summary

Statistically validates that one computational method's per-task accuracy is significantly better than a comparator's by running two-sided paired t-tests (scipy.stats.ttest_rel) over matched benchmark tasks, as done for Cellcano vs ACTINN, scJoint, and Signac across 50 scATAC-seq celltyping tasks.

## When to use

Use when you have per-task performance values (e.g., Accuracy) for two methods evaluated on the same matched set of benchmark tasks and need to show one method's advantage is statistically significant, not just numerically higher.

## When NOT to use

- Performance values are not paired across the same tasks/datasets for both methods (independent samples instead of matched per-task accuracies)
- Input is aggregated summary statistics (mean accuracy only) rather than per-task values, since pairing across 50 tasks is required
- The goal is cross-modality celltyping (e.g., scRNA-seq reference for scATAC-seq) performance, which the article reports as planned but not implemented

## Inputs

- Per-task Accuracy table for all 50 celltyping tasks (e.g., from the Source Data spreadsheet) with columns for Cellcano, ACTINN, scJoint, and Signac
- Comparator method name to test against Cellcano

## Outputs

- paired_ttest_pvalues.csv with method names and recomputed two-sided paired t-test p-values for the three comparators
- Recomputed p-values expected at 4.857e-3 (ACTINN), 1.645e-3 (scJoint), and 0.023 (Signac)

## How to apply

Load per-task accuracy values for all 50 matched celltyping tasks (e.g., from the paper's Source Data spreadsheet), extracting the accuracy columns for Cellcano and each comparator (ACTINN, scJoint, Signac). For each comparator, run a two-sided paired t-test via scipy.stats.ttest_rel between Cellcano's and the comparator's accuracy vectors across the 50 matched tasks — pairing is essential because the same tasks are shared between methods. Write the method names and recomputed p-values to an output CSV (paired_ttest_pvalues.csv) with one row per comparator. Validate each recomputed p-value against the reported values within tolerance: 4.857e-3 (ACTINN), 1.645e-3 (scJoint), and 0.023 (Signac). Interpret p < 0.05 as evidence of a statistically significant accuracy advantage, matching the paper's claim that Cellcano significantly outperforms all three methods.

## Related tools

- **Cellcano** (Method whose per-task accuracies are tested against comparators in the paired t-tests) — https://marvinquiet.github.io/Cellcano/
- **scJoint** (Comparator method; its 50-task accuracies are paired against Cellcano's for the t-test)
- **Signac** (Comparator method; its 50-task accuracies are paired against Cellcano's for the t-test)

## Examples

```
from scipy.stats import ttest_rel; t, p = ttest_rel(cellcano_acc, actinn_acc)
```

## Evaluation signals

- Recomputed p-value for Cellcano vs ACTINN matches 4.857e-3 within tolerance
- Recomputed p-value for Cellcano vs scJoint matches 1.645e-3 within tolerance
- Recomputed p-value for Cellcano vs Signac matches 0.023 within tolerance
- paired_ttest_pvalues.csv exists and contains all three comparator rows with valid p-values
- Each test uses paired (ttest_rel), not unpaired, testing over the same 50 matched tasks

## Limitations

- Significance is assessed only over the 50 celltyping tasks from human PBMCs and mouse brain; generalization to other datasets is not tested here
- No specific versions were reported for the compared tools (Seurat, Signac, scJoint), which limits reproducibility of the comparison
- Only Accuracy over paired tasks is tested; other metrics (e.g., runtime) are not covered by this significance testing

## Evidence

- [results] Paired t-tests on Accuracy over all 50 celltyping tasks show Cellcano significantly outperforms ACTINN (p = 4.857e-3), scJoint (p = 1.645e-3), and Signac (p = 0.023): [evidence span withheld — anchor status: not_found]
- [discussion] Cellcano significantly outperforms other methods across 50 tasks: "We have shown in 50 celltyping tasks with data from two systems (human PBMCs and mouse brain) that Cellcano significantly outperforms other celltyping methods"
- [abstract] Benchmark scope of 50 tasks: "After systematically benchmarking Cellcano on 50 well-designed celltyping tasks from various datasets"
- [readme] Cellcano tool context from README: "Cellcano is an open source software for supervised cell type identification (celltyping) in scATAC-seq data, published in [Nature Communications](https://doi.org/10.1038/s41467-023-37439-3)."
