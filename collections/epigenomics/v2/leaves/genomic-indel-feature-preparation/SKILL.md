---
name: genomic-indel-feature-preparation
description: Use when building supervised predictors for non-coding small indels (sindels)
  that need both generic functional annotations and tissue-specific epigenomic sequence-derived
  features, e.g. replicating the TIVAN-indel feature-set ablation.
when_to_use_negative:
- Input is already a compiled feature table (annot.csv plus seq_features.h5) — skip
  directly to train.py/predict.py
- Variants are SNVs rather than small insertions/deletions; CADD.indel.rda and the
  sequence-extraction workflow are specific to indels
- The goal is only to retrieve precomputed tissue-specific functional scores (e.g.
  Adipose_Subcutaneous.score.rda) — use R/TIVAN-indel.R instead of regenerating features
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: TIVAN-indel
  role: Supervised framework whose feature-preparation stage this skill implements
    (prepare.R + prepare.py + train.py)
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: R 4.1.1
  role: Runs prepare.R to extract CADD annotations and reference/alternative FASTA
    sequences using BSgenome.Hsapiens.UCSC.hg19 and data.table
- name: Python 3.9.6
  role: Runs src/prepare.py (DanQ model via tensorflow) to predict the 919 multi-omics
    features and train.py to assemble the model
- name: h5py
  role: Writes/reads seq_features.h5 holding the 919 DanQ-predicted epigenomic features
- name: xgboost
  role: Learning backend for the TIVAN-indel model trained on the prepared 45+919
    feature set
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor package used by prepare.R to extract reference and alternative
    genomic sequences flanking each indel
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad060
    title: 'TIVAN-indel: a computational framework for annotating and predicting non-coding
      regulatory small insertions and deletions'
  source_content_hash: sha256:aa74aaad76cc5fd1f7f3abe7ebb9d815093946b314881d338e7ad13990bfb991
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/TIVAN-indel
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - Python
  - Python 3.9.6
  - R
  - R 4.1.1
  - TIVAN-indel
  - h5py
  - numpy
  - pandas
  - scikit-learn
  - xgboost
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# genomic-indel-feature-preparation

## Summary

Prepares training/prediction features for non-coding regulatory small indels by combining 45 CADD functional annotations with 919 DanQ-predicted epigenomic multi-omics features derived from both reference and alternative genomic sequences, as used by TIVAN-indel across 44 GTEx tissues.

## When to use

Use when building supervised predictors for non-coding small indels (sindels) that need both generic functional annotations and tissue-specific epigenomic sequence-derived features, e.g. replicating the TIVAN-indel feature-set ablation.

## When NOT to use

- Input is already a compiled feature table (annot.csv plus seq_features.h5) — skip directly to train.py/predict.py
- Variants are SNVs rather than small insertions/deletions; CADD.indel.rda and the sequence-extraction workflow are specific to indels
- The goal is only to retrieve precomputed tissue-specific functional scores (e.g. Adipose_Subcutaneous.score.rda) — use R/TIVAN-indel.R instead of regenerating features

## Inputs

- CADD.indel.rda — R rda file of CADD annotations for all indels in the 1000 Genomes Project
- Positive indel position file (e.g. train/snp.pos.txt) with 'chr start end refallele altallele' per line
- Negative indel position file (e.g. train/snp.neg.txt) with the same columns
- Reference genome (hg19) accessed via BSgenome.Hsapiens.UCSC.hg19 in R
- Pretrained DanQ model weights in checkpoint/ and model code in models/ (optionally retrained from DeepSEA data)
- Query indel files for prediction mode (e.g. test/snp.pos.txt, test/snp.neg.txt)

## Outputs

- annot.csv — CADD annotations for both positive and negative indels
- Reference and alternative FASTA files per class (snp.pos.ref.fasta, snp.pos.alt.fasta, snp.neg.ref.fasta, snp.neg.alt.fasta) in the output folder
- seq_features.h5 — 919 DanQ-predicted multi-omics features for reference and alternative sequences
- Compiled feature matrix (45 CADD annotations + 919 multi-omics features) consumed by train.py

## How to apply

Run the R script prepare.R with four arguments (CADD.indel.rda, positive indel positions, negative indel positions, output folder), where each position file has columns 'chr start end refallele altallele'; this produces annot.csv with CADD annotations plus reference and alternative FASTA sequences (snp.pos.ref.fasta, snp.neg.ref.fasta, etc.) in the train folder. Then call src/prepare.py with the pretrained DanQ model (weights in checkpoint/, model code in models/) to predict 919 multi-omics features from the reference and alternative sequences, writing seq_features.h5 into the desired folder (e.g. python prepare.py --path ../train). The final feature matrix compiled for train.py combines 45 CADD annotations with the 919 sequence-derived features, supporting nested feature sets (CADD-only, +reference-sequence features, +reference+alternative-sequence features) for ablation. Rationale: epigenomic profiles predicted from both reference and alternative sequences carry tissue-specific regulatory signal absent from generic CADD scores, lifting median AUROC/AUPRC across 44 GTEx tissues from 0.858/0.843 (CADD-only) to 0.982/0.984.

## Related tools

- **TIVAN-indel** (Supervised framework whose feature-preparation stage this skill implements (prepare.R + prepare.py + train.py)) — https://github.com/lichen-lab/TIVAN-indel
- **R 4.1.1** (Runs prepare.R to extract CADD annotations and reference/alternative FASTA sequences using BSgenome.Hsapiens.UCSC.hg19 and data.table)
- **Python 3.9.6** (Runs src/prepare.py (DanQ model via tensorflow) to predict the 919 multi-omics features and train.py to assemble the model)
- **h5py** (Writes/reads seq_features.h5 holding the 919 DanQ-predicted epigenomic features)
- **xgboost** (Learning backend for the TIVAN-indel model trained on the prepared 45+919 feature set)
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor package used by prepare.R to extract reference and alternative genomic sequences flanking each indel)

## Examples

```
R --slave --args --no-save CADD.indel.rda train/snp.pos.txt train/snp.neg.txt train < prepare.R; python prepare.py --path ../train; python train.py --path ../train --output ../res/model.
```

## Evaluation signals

- Output folder (e.g. train/) contains exactly annot.csv plus four FASTA files (ref/alt for positive and negative sets)
- annot.csv row count matches the union of positive and negative indel position files, with 45 CADD annotation columns
- seq_features.h5 contains 919 feature columns for both reference and alternative sequences of every indel
- 5-fold cross-validation per GTEx tissue: median AUROC/AUPRC across 44 tissues should reach ~0.982/0.984 with CADD+ref+alt features vs ~0.858/0.843 with CADD-only
- Alternative-sequence features must show a performance delta over reference-only features (+0.123 median AUROC), confirming alt-allele profiles were correctly generated and merged

## Limitations

- Requires hg19-matched coordinates; indel positions must follow 'chr start end refallele altallele' formatting
- CADD.indel.rda and precomputed score files must be downloaded externally (Google Drive links in the README), so offline reproducibility depends on these hosted artifacts
- The DanQ model predicts 919 fixed epigenomic tracks; extending to other omics tracks requires retraining the model with DeepSEA data
- The ablation shows only modest gains from reference-sequence features alone (+0.001 median AUROC); the large gain depends specifically on including alternative-sequence profiles

## Evidence

- [methods] R script ``prepare.R" will generated (1) CADD annotations for query sindels and (2) reference genomic sequence and alterntive genomic sequence with sindels: "R script ``prepare.R" will generated (1) CADD annotations for query sindels and (2) reference genomic sequence and alterntive genomic sequence with sindels"
- [methods] File `src/prepare.py` calls DanQ model to predict 919 multi-omics features: "File `src/prepare.py` calls DanQ model to predict 919 multi-omics features"
- [readme] It uses a compilation of both 45 CADD annotations and 919 multi-omics features as generated above.: "It uses a compilation of both 45 CADD annotations and 919 multi-omics features as generated above. We call ``train.py" to train the model"
- [readme] A R rda file ``CADD.indel.rda" contains the CADD annotations for all indels in 1000 Genome Project: "A R rda file ``CADD.indel.rda" contains the CADD annotations for all indels in 1000 Genome Project can be downloaded [Here]"
- [readme] The trained weights of DanQ model are provided in the `checkpoint/` directory, while the model code is present in `models/` directory.: "The trained weights of DanQ model are provided in the `checkpoint/` directory, while the model code is present in `models/` directory. You can train your own model by using data from DeepSEA website"
