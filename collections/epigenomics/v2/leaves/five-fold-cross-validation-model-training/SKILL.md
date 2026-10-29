---
name: five-fold-cross-validation-model-training
description: Use when labeled noncoding sindels from cis-eQTL analyses across 44 GTEx
  tissues are available as positive/negative indel position files and a model's tissue-specific
  prediction performance (AUROC/AUPRC) must be estimated via 5-fold cross-validation
  under different feature sets.
when_to_use_negative:
- You only need functional scores for new query indels using existing precomputed
  tissue-specific scores — use the provided R/TIVAN-indel.R retrieval script instead
  of retraining with cross-validation.
- Your variants are SNVs rather than noncoding small insertions/deletions — the feature
  pipeline (CADD.indel.rda and reference/alternative indel sequences) is specific
  to sindels.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: TIVAN-indel
  role: Supervised computational framework being trained and cross-validated on labeled
    nc-sindels across 44 GTEx tissues
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: xgboost
  role: Gradient-boosted model used to train TIVAN-indel on 45 CADD annotations +
    919 multi-omics features
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: prepare.py
  role: Calls the pretrained DanQ model to predict the 919 multi-omics features from
    reference/alternative sequences, writing seq_features.h5
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: train.py
  role: Trains the TIVAN-indel model used in the 5-fold cross-validation
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: prepare.R
  role: R script that generates CADD annotations (annot.csv) and reference/alternative
    genomic sequences for the training indels
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: scikit-learn
  role: Python package (0.24.2) used in the training/evaluation pipeline for cross-validation
    metrics
  repo: https://github.com/lichen-lab/TIVAN-indel
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad060
    title: 'TIVAN-indel: a computational framework for annotating and predicting non-coding
      regulatory small insertions and deletions'
  source_content_hash: sha256:a3094e694d9bdbda0e420487263e29d9bd531602f2961f4b5693db15fe6e31dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/TIVAN-indel
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - Python
  - R
  - TIVAN-indel
  - h5py
  - numpy
  - pandas
  - prepare.R
  - prepare.py
  - scikit-learn
  - train.py
  - xgboost
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42105263157894735
n_claims: 19
---

# five-fold-cross-validation-model-training

## Summary

Trains the TIVAN-indel supervised model (xgboost on 45 CADD annotations + 919 DanQ-predicted multi-omics features) under nested feature sets and evaluates it with 5-fold cross-validation per GTEx tissue, collecting AUROC and AUPRC per fold per tissue.

## When to use

Use when labeled noncoding sindels from cis-eQTL analyses across 44 GTEx tissues are available as positive/negative indel position files and a model's tissue-specific prediction performance (AUROC/AUPRC) must be estimated via 5-fold cross-validation under different feature sets.

## When NOT to use

- You only need functional scores for new query indels using existing precomputed tissue-specific scores — use the provided R/TIVAN-indel.R retrieval script instead of retraining with cross-validation.
- Your variants are SNVs rather than noncoding small insertions/deletions — the feature pipeline (CADD.indel.rda and reference/alternative indel sequences) is specific to sindels.

## Inputs

- CADD.indel.rda (R rda file with CADD annotations for all indels in the 1000 Genomes Project)
- Positive indel position file (train/snp.pos.txt: chr start end refallele altallele)
- Negative indel position file (train/snp.neg.txt: chr start end refallele altallele)
- annot.csv (CADD annotations for positive and negative indels, produced by prepare.R)
- Reference and alternative genomic sequence FASTA files (snp.pos.ref.fasta, snp.pos.alt.fasta, snp.neg.ref.fasta, snp.neg.alt.fasta)
- seq_features.h5 (919 multi-omics features predicted by the pretrained DanQ model via src/prepare.py)

## Outputs

- Trained TIVAN-indel model (res/model.)
- Per-fold, per-tissue AUROC and AUPRC values for each of the three feature sets across 44 GTEx tissues
- Median AUROC and median AUPRC across tissues per feature set (expected: CADD-only 0.858/0.843; +ref 0.859/0.845; +ref+alt 0.982/0.984)

## How to apply

First generate training inputs: run R script prepare.R with CADD.indel.rda and the positive/negative indel position files (chr start end refallele altallele) to produce annot.csv plus reference/alternative FASTA sequences in the train folder; then run src/prepare.py with the pretrained DanQ model to predict the 919 multi-omics features into seq_features.h5. Train with train.py (python train.py --path ../train --output ../res/model.) using xgboost on the compilation of 45 CADD annotations and 919 multi-omics features, nested into three feature sets: CADD-only, CADD + reference-sequence features, and CADD + reference + alternative-sequence features. Run 5-fold cross-validation separately for each of the 44 GTEx tissues (e.g., Adipose_Subcutaneous through Whole_Blood), recording AUROC and AUPRC per fold per tissue per feature set. Compute the median AUROC and median AUPRC across the 44 tissues for each feature set and compare against reported values (CADD-only 0.858/0.843; +ref-seq 0.859/0.845; +ref+alt-seq 0.982/0.984) to confirm the reference+alternative epigenomic profiles drive the improvement.

## Related tools

- **TIVAN-indel** (Supervised computational framework being trained and cross-validated on labeled nc-sindels across 44 GTEx tissues) — https://github.com/lichen-lab/TIVAN-indel
- **xgboost** (Gradient-boosted model used to train TIVAN-indel on 45 CADD annotations + 919 multi-omics features) — https://github.com/lichen-lab/TIVAN-indel
- **prepare.py** (Calls the pretrained DanQ model to predict the 919 multi-omics features from reference/alternative sequences, writing seq_features.h5) — https://github.com/lichen-lab/TIVAN-indel
- **train.py** (Trains the TIVAN-indel model used in the 5-fold cross-validation) — https://github.com/lichen-lab/TIVAN-indel
- **prepare.R** (R script that generates CADD annotations (annot.csv) and reference/alternative genomic sequences for the training indels) — https://github.com/lichen-lab/TIVAN-indel
- **scikit-learn** (Python package (0.24.2) used in the training/evaluation pipeline for cross-validation metrics) — https://github.com/lichen-lab/TIVAN-indel

## Examples

```
python train.py --path ../train --output ../res/model.
```

## Evaluation signals

- Median AUROC across the 44 GTEx tissues for the CADD + reference + alternative feature set reaches ~0.982 and median AUPRC ~0.984 (Results 3.1 / Figure 2).
- CADD-only baseline yields median AUROC ~0.858 and AUPRC ~0.843, so the +ref+alt feature set should show a large gap over CADD-only.
- Per-tissue AUROC/AUPRC must be collected for all 44 tissues (e.g., Adipose_Subcutaneous through Whole_Blood), one entry per fold per feature set.
- train.py must consume both annot.csv (45 CADD annotations) and seq_features.h5 (919 multi-omics features); a run missing either input indicates a broken pipeline step.
- The three nested feature sets must be evaluated with identical fold splits so the comparison between CADD-only, +ref, and +ref+alt is like-for-like.

## Limitations

- Model performance depends on the pretrained DanQ checkpoint (in checkpoint/); training your own DanQ model requires data from the DeepSEA website.
- CADD-only and CADD+reference-sequence feature sets show nearly identical performance (~0.858 vs 0.859 median AUROC), so gains come specifically from including alternative-sequence epigenomic profiles.
- Cross-validation is tissue-specific: 5-fold CV must be run separately for each of the 44 GTEx tissues, making the full evaluation computationally heavy.

## Evidence

- [methods] File `src/prepare.py` calls DanQ model to predict 919 multi-omics features: "File `src/prepare.py` calls DanQ model to predict 919 multi-omics features"
- [readme] The model uses 45 CADD annotations and 919 multi-omics features: "It uses a compilation of both 45 CADD annotations and 919 multi-omics features as generated above. We call `train.py` to train the model"
- [readme] Labeled sindels from 44 GTEx tissues: "By leveraging labelled noncoding sindels identified by cis-expression quantitative trait loci (cis- eQTLs) analyses across 44 tissues in GTEx"
- [readme] prepare.R generates CADD annotations and reference/alternative sequences: "R script `prepare.R` will generated (1) CADD annotations for query sindels and (2) reference genomic sequence and alterntive genomic sequence with sindels"
- [methods] Predict on CADD.indel.rda covering 1000 Genomes indels: "A R rda file ``CADD.indel.rda" contains the CADD annotations for all indels in 1000 Genome Project"
