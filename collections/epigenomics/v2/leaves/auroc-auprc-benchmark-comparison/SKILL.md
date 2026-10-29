---
name: auroc-auprc-benchmark-comparison
description: Use when comparing supervised variant-prioritization classifiers across
  multiple tissue-specific folds or cross-validation splits, where per-fold AUROC
  and AUPRC must be computed and summarized (e.g., median across 44 GTEx tissues or
  mean across 5 folds).
when_to_use_negative:
- Evaluating regression or calibrated-probability-only tasks where rank-based AUROC/AUPRC
  are inappropriate metrics
- Scoring a single model with no baseline competitors, where per-fold benchmark comparison
  tables add no value
- Datasets not formatted as labeled nc-sindel 'chr start end refallele altallele'
  files with a defined positive/negative split
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: TIVAN-indel
  role: Primary xgboost classifier (45 CADD + 919 multi-omics features) whose predictions
    are benchmarked against baselines using AUROC/AUPRC
  repo: https://github.com/amanbasu/TIVAN-indel
- name: scikit-learn
  role: Supplies SVM baseline training and AUROC/AUPRC metric computation across CV
    folds
- name: pandas
  role: Aggregates per-fold/per-tissue metrics and writes the auroc_auprc_comparison.csv
    table
- name: matplotlib
  role: Generates the AUROC/AUPRC comparison figures (Fig. 6, Supplementary Fig. S7)
- name: xgboost
  role: Trains the TIVAN-indel model whose per-fold scores feed the AUROC/AUPRC evaluation
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad060
    title: 'TIVAN-indel: a computational framework for annotating and predicting non-coding
      regulatory small insertions and deletions'
  source_content_hash: sha256:7c8c9fac88fa2ac045dacd20c44df1ccb50f4ff826d1f584c470ea231ddd3ccf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/amanbasu/TIVAN-indel
  tools:
  - Python
  - R
  - TIVAN-indel
  - h5py
  - matplotlib
  - numpy
  - numpy-utils
  - pandas
  - scikit-learn
  - xgboost
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# auroc-auprc-benchmark-comparison

## Summary

Benchmark comparison protocol from TIVAN-indel that evaluates predictive models (TIVAN-indel vs CADD/FATHMM-indel SVM and DanQ/DeepSEA Boosted LR) using AUROC and AUPRC across 44 leave-one-tissue-out GTEx cross-tissue folds and 5-fold ClinVar cross-validation.

## When to use

Use when comparing supervised variant-prioritization classifiers across multiple tissue-specific folds or cross-validation splits, where per-fold AUROC and AUPRC must be computed and summarized (e.g., median across 44 GTEx tissues or mean across 5 folds).

## When NOT to use

- Evaluating regression or calibrated-probability-only tasks where rank-based AUROC/AUPRC are inappropriate metrics
- Scoring a single model with no baseline competitors, where per-fold benchmark comparison tables add no value
- Datasets not formatted as labeled nc-sindel 'chr start end refallele altallele' files with a defined positive/negative split

## Inputs

- Model output files of functional scores per held-out fold (e.g., ../res/output.txt) for TIVAN-indel, CADD/FATHMM-indel SVM, and DanQ/DeepSEA Boosted LR
- Binary labels for held-out nc-sindels: 44 GTEx tissue leave-one-tissue-out splits, or 5-fold splits of 132 pathogenic vs 1104 benign ClinVar nc-sindels
- GTEx tissue class table (44 tissues/cell types, e.g., Adipose_Subcutaneous, Brain_*) for per-tissue summarization

## Outputs

- Per-tissue and per-fold AUROC and AUPRC values for each method
- Cross-method comparison table (auroc_auprc_comparison.csv) with mean AUROC/AUPRC over 5 ClinVar CV folds
- Median cross-tissue AUROC/AUPRC summary across the 44 GTEx tissues (e.g., 0.971/0.964 for TIVAN-indel)

## How to apply

Score held-out sindels with each trained model via predict.py (e.g., python predict.py --path ../test --model ../res/model. --output ../res/output.txt) for all 44 leave-one-tissue-out GTEx folds, and repeat prediction for the SVM baseline on 45 CADD/FATHMM-indel annotations and the boosted logistic regression baseline on 919 DanQ/DeepSEA multi-omics features under identical splits. Compute AUROC and AUPRC per method per fold, then summarize: median AUROC/AUPRC across the 44 tissues for cross-tissue evaluation (TIVAN-indel 0.971/0.964 vs SVM 0.831/0.774 vs Boosted LR 0.813/0.722), and mean AUROC/AUPRC across 5 folds for the ClinVar comparison (132 pathogenic vs 1104 benign nc-sindels). Write the per-method comparison to auroc_auprc_comparison.csv and verify all values lie in [0,1]. Judge correctness by confirming TIVAN-indel attains the highest median cross-tissue AUROC/AUPRC over both baselines for every tissue, and that top-performing tissues come mainly from the Brain tissue class.

## Related tools

- **TIVAN-indel** (Primary xgboost classifier (45 CADD + 919 multi-omics features) whose predictions are benchmarked against baselines using AUROC/AUPRC) — https://github.com/amanbasu/TIVAN-indel
- **scikit-learn** (Supplies SVM baseline training and AUROC/AUPRC metric computation across CV folds)
- **pandas** (Aggregates per-fold/per-tissue metrics and writes the auroc_auprc_comparison.csv table)
- **matplotlib** (Generates the AUROC/AUPRC comparison figures (Fig. 6, Supplementary Fig. S7))
- **xgboost** (Trains the TIVAN-indel model whose per-fold scores feed the AUROC/AUPRC evaluation)

## Examples

```
python predict.py --path ../test --model ../res/model. --output ../res/output.txt
```

## Evaluation signals

- All computed AUROC and AUPRC values lie within [0,1]
- Median cross-tissue AUROC/AUPRC ordering matches TIVAN-indel (0.971/0.964) > SVM (0.831/0.774) > Boosted LR (0.813/0.722)
- TIVAN-indel achieves highest AUROC and AUPRC for all 44 tissues in the cross-tissue setting
- Top-performing tissues in the cross-tissue evaluation come mainly from the Brain tissue class
- ClinVar 5-fold mean AUROC/AUPRC per method is consistent with Figure 6 and Supplementary Fig. S7

## Limitations

- Cross-tissue (leave-one-tissue-out) performance is lower than with-tissue performance; the skill's headline numbers are medians over 44 folds, not per-tissue guarantees
- Precomputed functional scores cover only noncoding small indels in 1000 Genomes across the 44 GTEx tissues; other variant classes or tissues require retraining and rescoring
- The ClinVar comparison is limited to 132 pathogenic and 1104 benign nc-sindels on hg19 with indels shorter than 100 bp and high review status

## Evidence

- [results] In the cross-tissue evaluation, TIVAN-indel outperforms CADD/FATHMM-indel (SVM) and DanQ/DeepSEA (Boosted LR) for all tissues, achieving the highest median AUROC (0.971 versus 0.831 versus 0.813): [evidence span withheld — anchor status: not_found]
- [results] Using 5-fold cross-validation on a ClinVar-derived set of 132 pathogenic and 1104 benign nc-sindels, TIVAN-indel outperforms CADD/FATHMM-indel (SVM) and DanQ/DeepSEA (Boosted LR) by achieving the highest AUROC: [evidence span withheld — anchor status: not_found]
- [readme] python predict.py --path ../test --model ../res/model. --output ../res/output.txt: [evidence span withheld — anchor status: not_found]
- [abstract] we demonstrate that TIVAN-indel achieves the best prediction performance in both with-tissue prediction and cross-tissue prediction: "we demonstrate that TIVAN-indel achieves the best prediction performance in both with-tissue prediction and cross-tissue prediction"
- [readme] The precomputed scores for All noncoding small indels in 1000 Genomes across 44 tissues/cell types (e.g., Adipose_Subcutaneous.score.rda) in GTEX: "The precomputed scores for All noncoding small indels in 1000 Genomes across 44 tissues/cell types (e.g., Adipose_Subcutaneous.score.rda) in GTEX"
