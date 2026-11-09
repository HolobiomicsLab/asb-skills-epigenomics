---
name: per-task-benchmark-accuracy-aggregation
description: Use when comparing two or more celltyping methods evaluated on the same
  set of matched tasks (e.g., 50 Cellcano benchmark tasks) and you need a statistical
  test of whether one method's accuracy advantage is significant.
when_to_use_negative:
- Accuracies are not task-paired (methods evaluated on different task sets) — an unpaired
  or rank-based comparison is required instead
- Input is already aggregated summary statistics (e.g., mean accuracy only) without
  per-task values — paired t-tests cannot be recomputed
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Cellcano
  role: Method under test whose per-task accuracies across the 50 celltyping tasks
    are compared against competitors
  repo: https://marvinquiet.github.io/Cellcano/
- name: scipy.stats.ttest_rel
  role: Two-sided paired t-test used to compute p-values for Cellcano vs each comparator
    over matched tasks
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:9e313e711ad3fd53c64ad2ff8f55aaf71b2e75436b8086bfaf563100568d3eb0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cellcano
  - scipy.stats.ttest_rel
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# per-task-benchmark-accuracy-aggregation

## Summary

Aggregate per-task celltyping accuracy values across matched benchmark tasks and test significance of method differences with two-sided paired t-tests, as done for Cellcano vs ACTINN, scJoint, and Signac over 50 celltyping tasks.

## When to use

Use when comparing two or more celltyping methods evaluated on the same set of matched tasks (e.g., 50 Cellcano benchmark tasks) and you need a statistical test of whether one method's accuracy advantage is significant.

## When NOT to use

- Accuracies are not task-paired (methods evaluated on different task sets) — an unpaired or rank-based comparison is required instead
- Input is already aggregated summary statistics (e.g., mean accuracy only) without per-task values — paired t-tests cannot be recomputed

## Inputs

- Source Data spreadsheet with per-task accuracy columns for Cellcano, ACTINN, scJoint, and Signac across 50 celltyping tasks
- Task-matched accuracy vectors (one accuracy value per task per method)

## Outputs

- paired_ttest_pvalues.csv containing method names and recomputed two-sided paired t-test p-values for each comparator

## How to apply

Load per-task accuracy values for all 50 celltyping tasks from the paper's Source Data spreadsheet, extracting the accuracy columns for Cellcano, ACTINN, scJoint, and Signac. For each comparator, run a two-sided paired t-test (scipy.stats.ttest_rel) between Cellcano's accuracies and the comparator's accuracies across the 50 matched tasks, ensuring task pairing is preserved so the same task is compared for both methods. Write the method names and recomputed p-values to paired_ttest_pvalues.csv. Validate that each recomputed p-value matches the reported value within tolerance — 4.857e-3 (ACTINN), 1.645e-3 (scJoint), and 0.023 (Signac) — and that the output file exists with all three comparator rows.

## Related tools

- **Cellcano** (Method under test whose per-task accuracies across the 50 celltyping tasks are compared against competitors) — https://marvinquiet.github.io/Cellcano/
- **scipy.stats.ttest_rel** (Two-sided paired t-test used to compute p-values for Cellcano vs each comparator over matched tasks)

## Examples

```
from scipy.stats import ttest_rel; t, p = ttest_rel(cellcano_acc, actinn_acc)  # p expected ~4.857e-3
```

## Evaluation signals

- Recomputed p-value for ACTINN matches 4.857e-3 within tolerance
- Recomputed p-value for scJoint matches 1.645e-3 within tolerance
- Recomputed p-value for Signac matches 0.023 within tolerance
- paired_ttest_pvalues.csv exists and contains all three comparator rows
- Each paired t-test uses exactly 50 matched task pairs (equal-length accuracy vectors aligned by task)

## Limitations

- Comparison is limited to Cellcano, ACTINN, scJoint, and Signac; no specific versions were reported for the compared tools in this section
- Conclusion rests on 50 tasks drawn from only two systems (human PBMCs and mouse brain), so scope of generalization is bounded by that benchmark design

## Evidence

- [other] Paired t-tests on Accuracy over all 50 celltyping tasks: [evidence span withheld — anchor status: not_found]
- [abstract] Benchmark scope of 50 tasks: "After systematically benchmarking Cellcano on 50 well-designed celltyping tasks from various datasets"
- [discussion] Two systems covered by benchmark: "We have shown in 50 celltyping tasks with data from two systems (human PBMCs and mouse brain) that Cellcano significantly outperforms other celltyping methods"
