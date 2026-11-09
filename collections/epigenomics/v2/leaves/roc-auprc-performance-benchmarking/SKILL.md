---
name: roc-auprc-performance-benchmarking
description: Use when predicted functional scores for nc-sindels and gold-standard
  positive/negative eQTL labels are available and you must compare classifier performance
  (AUROC/AUPRC) across multiple tissues or cell types.
when_to_use_negative:
- Input is already a feature table (annot.csv CADD annotations or seq_features.h5
  multi-omics features) rather than predicted scores — feature generation, not benchmarking,
  is needed
- The goal is within-GTEx tissue prediction rather than the independent DICE cross-study
  evaluation described here
- No labeled positive/negative sindel sets exist for the test cell types, since AUROC/AUPRC
  cannot be computed without ground truth
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: TIVAN-indel
  role: Produces the functional scores whose AUROC/AUPRC are benchmarked against CADD/FATHMM-indel
    (SVM) and DanQ/DeepSEA (Boosted LR) baselines
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: scikit-learn
  role: Computes AUROC and AUPRC metrics from predicted scores and DICE positive/negative
    labels
- name: Python
  role: Runtime (3.9.6) for predict.py scoring and metric aggregation across the 15
    cell types
- name: numpy
  role: Median aggregation of per-cell-type AUROC/AUPRC values
- name: Database of Immune Cell Expression (DICE)
  role: 'Independent test resource: 15 immune cell types providing held-out labeled
    nc-sindels for cross-tissue evaluation'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad060
    title: 'TIVAN-indel: a computational framework for annotating and predicting non-coding
      regulatory small insertions and deletions'
  source_content_hash: sha256:8a3329dc5fa8db9e360182d513b687577c696f5eac2bdf92d5040ffa95e39067
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/TIVAN-indel
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - Database of Immune Cell Expression (DICE)
  - Python
  - R
  - TIVAN-indel
  - TensorFlow
  - h5py
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# roc-auprc-performance-benchmarking

## Summary

Benchmark TIVAN-indel's predicted functional scores against competing indel-prioritization methods (CADD/FATHMM-indel SVM and DanQ/DeepSEA with Boosted LR) by computing median AUROC and AUPRC across 15 independent DICE immune cell types.

## When to use

Use when predicted functional scores for nc-sindels and gold-standard positive/negative eQTL labels are available and you must compare classifier performance (AUROC/AUPRC) across multiple tissues or cell types.

## When NOT to use

- Input is already a feature table (annot.csv CADD annotations or seq_features.h5 multi-omics features) rather than predicted scores — feature generation, not benchmarking, is needed
- The goal is within-GTEx tissue prediction rather than the independent DICE cross-study evaluation described here
- No labeled positive/negative sindel sets exist for the test cell types, since AUROC/AUPRC cannot be computed without ground truth

## Inputs

- Predicted functional scores per indel from predict.py (../res/output.txt)
- Test indel labels for 15 DICE immune cell types (positive cis-eQTL sindels vs negative sindels, e.g. test/snp.pos.txt and test/snp.neg.txt)
- Baseline predicted scores from CADD/FATHMM-indel (SVM) and DanQ/DeepSEA (Boosted LR) for the same indels

## Outputs

- Per-cell-type AUROC and AUPRC values for each method across the 15 DICE immune cell types
- Median AUROC and median AUPRC summary per method (expected: TIVAN-indel 0.973 AUROC / 0.978 AUPRC)
- Cross-tissue performance comparison as reported in Figure 5

## How to apply

Run predict.py on the test indels to obtain functional scores (--path ../test, --model ../res/model., --output ../res/output.txt), then, for each of the 15 DICE immune cell types, compute AUROC and AUPRC by treating DICE cis-eQTL-derived positive/negative sindel labels as ground truth and the predicted scores as the ranking statistic. Aggregate performance by taking the median AUROC and median AUPRC across the 15 cell types rather than pooling, so no single cell type dominates. Report the medians side-by-side for TIVAN-indel, CADD/FATHMM-indel (SVM), and DanQ/DeepSEA (Boosted LR) to reproduce the cross-tissue comparison of Figure 5. Correct application yields median AUROC 0.973 (TIVAN-indel) vs 0.773 (CADD/FATHMM-indel) vs 0.744 (DanQ/DeepSEA) and median AUPRC 0.978 vs 0.792 vs 0.733, with TIVAN-indel winning in all 15 cell types.

## Related tools

- **TIVAN-indel** (Produces the functional scores whose AUROC/AUPRC are benchmarked against CADD/FATHMM-indel (SVM) and DanQ/DeepSEA (Boosted LR) baselines) — https://github.com/lichen-lab/TIVAN-indel
- **scikit-learn** (Computes AUROC and AUPRC metrics from predicted scores and DICE positive/negative labels)
- **Python** (Runtime (3.9.6) for predict.py scoring and metric aggregation across the 15 cell types)
- **numpy** (Median aggregation of per-cell-type AUROC/AUPRC values)
- **Database of Immune Cell Expression (DICE)** (Independent test resource: 15 immune cell types providing held-out labeled nc-sindels for cross-tissue evaluation)

## Examples

```
python predict.py --path ../test --model ../res/model. --output ../res/output.txt
```

## Evaluation signals

- Median AUROC across the 15 DICE immune cell types equals 0.973 for TIVAN-indel (vs 0.773 CADD/FATHMM-indel and 0.744 DanQ/DeepSEA)
- Median AUPRC across the 15 DICE immune cell types equals 0.978 for TIVAN-indel (vs 0.792 and 0.733 for the baselines)
- TIVAN-indel consistently outperforms both baselines in each of the 15 cell types, not only on the median
- Scores in ../res/output.txt are produced by predict.py with --model ../res/model. and cover the same test indels used for the baselines

## Limitations

- The model evaluated is trained on the 'Whole Blood' GTEx tissue, so the DICE benchmark measures cross-tissue/generalization performance, not with-tissue performance
- Metric aggregation is by median across cell types; per-cell-type variability beyond the reported medians is what Figure 5 conveys and should not be inferred from summary numbers alone
- The public discussion/changelog content is absent from the source document, so caveats beyond those stated here are not available

## Evidence

- [full_text] TIVAN-indel consistently outperforms CADD/FATHMM-indel (SVM) and DanQ/DeepSEA (Boosted LR) across all 15 DICE immune cell types: [evidence span withheld — anchor status: not_found]
- [results] achieving the highest median AUROC (0.973 vs 0.773 vs 0.744) and AUPRC (0.978 vs 0.792 vs 0.733): [evidence span withheld — anchor status: not_found]
- [abstract] we train TIVAN-indel from the 'Whole Blood' tissue in GTEx and test the model using 15 immune cell types from an independent study named Database of Immune Cell Expression: "we train TIVAN-indel from the 'Whole Blood' tissue in GTEx and test the model using 15 immune cell types from an independent study named Database of Immune Cell Expression"
- [readme] python predict.py --path ../test --model ../res/model. --output ../res/output.txt: [evidence span withheld — anchor status: not_found]
