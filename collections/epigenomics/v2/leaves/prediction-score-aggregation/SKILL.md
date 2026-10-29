---
name: prediction-score-aggregation
description: Use when a leave-one-out or jackknife evaluation has emitted per-sample
  prediction scores and you need to aggregate them into a headline metric (e.g., ACC)
  for comparison against a published baseline.
when_to_use_negative:
- The evaluation design is 5-fold cross-validation or an independent test set — use
  the classifier_selection.py or csDMA_indepedent_testing.py workflows instead of
  jackknife score aggregation.
- The input is already an aggregated metric (e.g., an AUC or ACC value) rather than
  per-sample prediction scores.
- The dataset is not the paired training positive/negative set with matching Motif
  feature matrices (matrix_motif1/matrix_motif2), since leaveoneout.py requires all
  four inputs.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: csDMA
  role: prediction tool whose leaveoneout.py script performs the jackknife evaluation
    and emits the per-sample score file that is aggregated into ACC
  repo: https://github.com/liuze-nwafu/csDMA
- name: ExtraTrees
  role: classifier retrained once per left-out sample during the LeaveOneOut (jackknife)
    evaluation
  repo: https://github.com/liuze-nwafu/csDMA
- name: i6mA-Pred
  role: existing rice 6mA predictor whose jackknife ACC of 0.831 serves as the comparison
    baseline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:7e14d4d3ce03ddb6366cfcb09b0c1b6e49b6b37ef5a2a4b2d99f5770edfb8467
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liuze-nwafu/csDMA
  tools:
  - ExtraTrees
  - csDMA
  - i6mA-Pred
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# prediction-score-aggregation

## Summary

Aggregates per-sample prediction scores produced by a leave-one-out (jackknife) evaluation into a single accuracy (ACC) metric, as done in csDMA to reproduce a jackknife ACC of ~0.859 on the rice 6mA dataset (880 positive / 880 negative) versus the i6mA-Pred baseline of 0.831.

## When to use

Use when a leave-one-out or jackknife evaluation has emitted per-sample prediction scores and you need to aggregate them into a headline metric (e.g., ACC) for comparison against a published baseline.

## When NOT to use

- The evaluation design is 5-fold cross-validation or an independent test set — use the classifier_selection.py or csDMA_indepedent_testing.py workflows instead of jackknife score aggregation.
- The input is already an aggregated metric (e.g., an AUC or ACC value) rather than per-sample prediction scores.
- The dataset is not the paired training positive/negative set with matching Motif feature matrices (matrix_motif1/matrix_motif2), since leaveoneout.py requires all four inputs.

## Inputs

- rice training positive dataset (880 samples, 41nt 6mA sequences)
- rice training negative dataset (880 samples)
- precomputed Motif feature matrices (matrix_motif1 and matrix_motif2) generated during training
- per-sample LeaveOneOut score file produced by leaveoneout.py

## Outputs

- LeaveOneOut score file with one prediction score per held-out sample
- aggregated jackknife ACC for the rice dataset (~0.859 expected)

## How to apply

Run the csDMA leaveoneout.py script with the ExtraTrees classifier ('python leaveoneout.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2'), which retrains the ExtraTrees model once per left-out sample and predicts each held-out sample. Write the per-sample prediction scores into the generated LeaveOneOut score file. Aggregate the per-sample scores into the jackknife ACC over the 880/880 rice benchmark dataset. Compare the resulting ACC against the i6mA-Pred jackknife baseline of 0.831 (reported in its online abstract) and expect ~0.859 for csDMA; success is validated when the score file exists and the computed ACC reproduces approximately 0.859.

## Related tools

- **csDMA** (prediction tool whose leaveoneout.py script performs the jackknife evaluation and emits the per-sample score file that is aggregated into ACC) — https://github.com/liuze-nwafu/csDMA
- **ExtraTrees** (classifier retrained once per left-out sample during the LeaveOneOut (jackknife) evaluation) — https://github.com/liuze-nwafu/csDMA
- **i6mA-Pred** (existing rice 6mA predictor whose jackknife ACC of 0.831 serves as the comparison baseline)

## Examples

```
python leaveoneout.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2
```

## Evaluation signals

- The LeaveOneOut score file is generated and contains a prediction score for each of the 1,760 rice samples (880 positive / 880 negative).
- The aggregated jackknife ACC reproduces the reported csDMA rice-dataset accuracy of approximately 0.859.
- The computed ACC (0.859) exceeds the i6mA-Pred jackknife baseline of 0.831 acknowledged from its online abstract.
- The evaluation protocol matches the jackknife test (model retrained once per left-out sample) rather than 5-fold cross-validation.

## Limitations

- The i6mA-Pred method could not be run directly for comparison because the paper was still in the corrected proof phase; its 0.831 ACC is known only from the online abstract.
- csDMA performance was relatively low on the rice and cross-species datasets compared to the M. musculus dataset, so the ~0.859 jackknife ACC is dataset-specific.
- csDMA was implemented in Python 2.7, which may affect reproducibility on modern environments.

## Evidence

- [discussion] i6mA-Pred received an ACC of 0.831 by using a jackknife test, as acknowledged from its online abstract.: "we acknowledge from their online abstract that the method received an ACC of 0.831 by using a jackknife test"
- [readme] leaveoneout.py evaluates model performance by LeaveOneOut and generates a score file.: "#This script evaluates the model performance by using LeaveOneOut method.
#The csDMA trained with the ExtraTrees classifier."
- [readme] leaveoneout.py usage requires the training datasets and Motif matrices.: "Usage: python leaveoneout.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2"
- [intro] ExtraTrees was the selected classifier for the optimized model.: "Thus, we used the ExtraTrees algorithm to train the optimized model."
- [intro] The rice benchmark dataset consists of 880 positive and 880 negative samples.: "The benchmark dataset consists of 880 positive samples and 880 negative samples."
