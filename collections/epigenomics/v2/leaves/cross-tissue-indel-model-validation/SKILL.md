---
name: cross-tissue-indel-model-validation
description: Use when evaluating whether a tissue-specific regulatory indel classifier
  generalizes across GTEx tissues, i.e. when labeled cis-eQTL sindel sets for 44 tissues
  are available and generalization must be quantified.
when_to_use_negative:
- Goal is with-tissue (same-tissue) prediction only, where cross-tissue generalization
  is not the question
- The task is scoring user-supplied sindels with existing precomputed tissue-specific
  scores (R/TIVAN-indel.R), not benchmarking generalization
- Input variants are SNVs or large structural variants rather than small non-coding
  indels (sindels)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: TIVAN-indel
  role: Supervised xgboost framework whose cross-tissue generalization is benchmarked
    in the leave-one-tissue-out evaluation
  repo: https://github.com/amanbasu/TIVAN-indel
- name: xgboost
  role: Trains the TIVAN-indel model on the combined 45 CADD + 919 multi-omics feature
    matrix per tissue fold
  repo: https://github.com/amanbasu/TIVAN-indel
- name: h5py
  role: Stores and loads the 919 DanQ-predicted multi-omics features as seq_features.h5
  repo: https://github.com/amanbasu/TIVAN-indel
- name: pandas
  role: Handles CADD annotation table (annot.csv) and per-tissue result summarization
  repo: https://github.com/amanbasu/TIVAN-indel
- name: matplotlib
  role: Visualization of per-tissue AUROC/AUPRC benchmark results
  repo: https://github.com/amanbasu/TIVAN-indel
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad060
    title: 'TIVAN-indel: a computational framework for annotating and predicting non-coding
      regulatory small insertions and deletions'
  source_content_hash: sha256:4568b190237b6360d3a395a53d0ec414048d0e1d891dd7b109499b1ff3de51dc
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
  - pandas
  - xgboost
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# cross-tissue-indel-model-validation

## Summary

Leave-one-tissue-out benchmarking protocol for TIVAN-indel: train the xgboost model on labeled non-coding sindels from one GTEx tissue (45 CADD + 919 DanQ multi-omics features) and test it on the other 43 tissues, comparing median AUROC/AUPRC against SVM and boosted-LR baselines.

## When to use

Use when evaluating whether a tissue-specific regulatory indel classifier generalizes across GTEx tissues, i.e. when labeled cis-eQTL sindel sets for 44 tissues are available and generalization must be quantified.

## When NOT to use

- Goal is with-tissue (same-tissue) prediction only, where cross-tissue generalization is not the question
- The task is scoring user-supplied sindels with existing precomputed tissue-specific scores (R/TIVAN-indel.R), not benchmarking generalization
- Input variants are SNVs or large structural variants rather than small non-coding indels (sindels)

## Inputs

- CADD.indel.rda — CADD annotations for all indels in 1000 Genomes Project
- snp.pos.txt / snp.neg.txt — labeled positive/negative non-coding sindels per GTEx tissue (chr start end refallele altallele)
- Pretrained DanQ model weights in checkpoint/ directory
- Precomputed per-tissue TIVAN-indel functional score files (e.g., Adipose_Subcutaneous.score.rda) for the 44 GTEx tissues

## Outputs

- annot.csv — CADD annotations for positive and negative indels per tissue
- snp.pos/snp.neg ref and alt FASTA sequence files
- seq_features.h5 — 919 DanQ-predicted multi-omics features
- Trained xgboost model (res/model.) per leave-one-tissue-out fold
- res/output.txt — predicted functional scores for held-out tissues
- Per-tissue and median cross-tissue AUROC/AUPRC summary across 44 GTEx tissues

## How to apply

Run prepare.R with CADD.indel.rda plus positive/negative sindel position files (chr start end refallele altallele) to generate annot.csv and reference/alternative FASTA files; then src/prepare.py with the pretrained DanQ checkpoint predicts 919 multi-omics features stored as seq_features.h5. Train TIVAN-indel per GTEx tissue via train.py using xgboost on the combined 45 CADD + 919 feature matrix, in parallel training the CADD/FATHMM-indel SVM and DanQ/DeepSEA boosted logistic regression baselines. For each of the 44 leave-one-tissue-out folds, generate held-out features with predict.R/predict.py and score with predict.py. Compute per-tissue AUROC and AUPRC in both with-tissue and cross-tissue settings; the skill is verified when TIVAN-indel's median cross-tissue AUROC (0.971) and AUPRC (0.964) exceed the SVM (0.831/0.774) and boosted-LR (0.813/0.722) baselines for all tissues, with top tissues concentrated in the Brain class.

## Related tools

- **TIVAN-indel** (Supervised xgboost framework whose cross-tissue generalization is benchmarked in the leave-one-tissue-out evaluation) — https://github.com/amanbasu/TIVAN-indel
- **xgboost** (Trains the TIVAN-indel model on the combined 45 CADD + 919 multi-omics feature matrix per tissue fold) — https://github.com/amanbasu/TIVAN-indel
- **h5py** (Stores and loads the 919 DanQ-predicted multi-omics features as seq_features.h5) — https://github.com/amanbasu/TIVAN-indel
- **pandas** (Handles CADD annotation table (annot.csv) and per-tissue result summarization) — https://github.com/amanbasu/TIVAN-indel
- **matplotlib** (Visualization of per-tissue AUROC/AUPRC benchmark results) — https://github.com/amanbasu/TIVAN-indel

## Examples

```
python predict.py --path ../test --model ../res/model. --output ../res/output.txt
```

## Evaluation signals

- Each of the 44 leave-one-tissue-out folds produces a trained model file (res/model.) and a res/output.txt of held-out scores
- Median cross-tissue AUROC of TIVAN-indel should be 0.971, exceeding CADD/FATHMM-indel SVM (0.831) and DanQ/DeepSEA boosted LR (0.813)
- Median cross-tissue AUPRC of TIVAN-indel should be 0.964 versus 0.774 (SVM) and 0.722 (boosted LR), with TIVAN-indel outperforming baselines for all tissues
- seq_features.h5 must contain exactly 919 multi-omics features per indel from the DanQ model
- Top-performing tissues in the cross-tissue evaluation should come mainly from the Brain tissue class

## Limitations

- Cross-tissue performance depends on labeled cis-eQTL sindel sets from 44 GTEx tissues; tissues outside GTEx coverage or immune cell types require the independent DICE test setting instead
- The 919 multi-omics features rely on the pretrained DanQ/DeepSEA model; training your own DanQ model requires data from the DeepSEA website
- The repository discussion/changelog content is absent from the source document, so failure modes beyond those documented (e.g., missing CADD.indel.rda or checkpoint weights) are not cataloged

## Evidence

- [abstract] we demonstrate that TIVAN-indel achieves the best prediction performance in both with-tissue prediction and cross-tissue prediction: "we demonstrate that TIVAN-indel achieves the best prediction performance in both with-tissue prediction and cross-tissue prediction"
- [results] In the cross-tissue evaluation, TIVAN-indel outperforms CADD/FATHMM-indel (SVM) and DanQ/DeepSEA (Boosted LR) for all tissues: [evidence span withheld — anchor status: not_found]
- [methods] File `src/prepare.py` calls DanQ model to predict 919 multi-omics features: "File `src/prepare.py` calls DanQ model to predict 919 multi-omics features"
- [readme] It uses a compilation of both 45 CADD annotations and 919 multi-omics features as generated above: "The model architecture is described in Figure 1 (Part C). It uses a compilation of both 45 CADD annotations and 919 multi-omics features as generated above."
