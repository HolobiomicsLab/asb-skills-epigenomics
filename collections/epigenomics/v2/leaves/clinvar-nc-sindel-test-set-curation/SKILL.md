---
name: clinvar-nc-sindel-test-set-curation
description: Use when an independent test set of pathogenic vs benign non-coding small
  indels is needed to benchmark TIVAN-indel or comparable functional-impact predictors
  (CADD, FATHMM-indel, DeepSEA/DanQ) on ClinVar variants.
when_to_use_negative:
- Query variants are coding SNVs rather than non-coding small indels shorter than
  100 bp
- Variants are not on hg19 or lack high review status in ClinVar, so they cannot serve
  as a trustworthy labeled test set
- The goal is to retrieve precomputed tissue-specific functional scores for 1000 Genomes
  nc-sindels (use the R/TIVAN-indel.R retrieval workflow instead of re-benchmarking)
edam_operation: http://edamontology.org/operation_0331
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: TIVAN-indel
  role: Supervised framework whose trained model (train.py, predict.py) scores the
    curated ClinVar nc-sindel test set for AUROC/AUPRC benchmarking
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: R
  role: Runs prepare.R to generate CADD annotations (annot.csv) and reference/alternative
    FASTA sequences for the test sindels using CADD.indel.rda
- name: Python
  role: Runs prepare.py (DanQ prediction of 919 multi-omics features), train.py and
    predict.py to score the test set into ../res/output.txt
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad060
    title: 'TIVAN-indel: a computational framework for annotating and predicting non-coding
      regulatory small insertions and deletions'
  source_content_hash: sha256:3ec7fd92b2fcaa5d711f00f666c814dcb9639f158ff757bb5b243d1c99344002
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/TIVAN-indel
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
  edam_operation: http://edamontology.org/operation_0331
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# clinvar-nc-sindel-test-set-curation

## Summary

Curates a ClinVar-derived benchmark set of 132 pathogenic and 1104 benign non-coding small indels (nc-sindels) on hg19 to test whether TIVAN-indel outperforms CADD/FATHMM-indel (SVM) and DanQ/DeepSEA (Boosted LR) via AUROC and AUPRC under 5-fold cross-validation.

## When to use

Use when an independent test set of pathogenic vs benign non-coding small indels is needed to benchmark TIVAN-indel or comparable functional-impact predictors (CADD, FATHMM-indel, DeepSEA/DanQ) on ClinVar variants.

## When NOT to use

- Query variants are coding SNVs rather than non-coding small indels shorter than 100 bp
- Variants are not on hg19 or lack high review status in ClinVar, so they cannot serve as a trustworthy labeled test set
- The goal is to retrieve precomputed tissue-specific functional scores for 1000 Genomes nc-sindels (use the R/TIVAN-indel.R retrieval workflow instead of re-benchmarking)

## Inputs

- ClinVar-derived pathogenic nc-sindel list (hg19, indels < 100 bp, high review status) formatted as 'chr start end refallele altallele' (snp.pos.txt)
- ClinVar-derived benign nc-sindel list (hg19, indels < 100 bp, high review status) formatted as 'chr start end refallele altallele' (snp.neg.txt)
- CADD.indel.rda — R rda file of CADD annotations for all indels in the 1000 Genomes Project
- Reference genome BSgenome.Hsapiens.UCSC.hg19

## Outputs

- annot.csv — CADD annotations (45 features) for the ClinVar test sindels
- Reference and alternative genomic FASTA sequences for test indels (e.g. snp.pos.ref.fasta / alternative fasta)
- seq_features.h5 — 919 DanQ-predicted multi-omics features for the test set
- Functional scores for test indels (../res/output.txt)
- auroc_auprc_comparison.csv — per-method mean AUROC and AUPRC across 5 folds

## How to apply

Curate pathogenic and benign nc-sindels from ClinVar on hg19, filtering to indels shorter than 100 bp with high review status, and write them as 'chr start end refallele altallele' lines into snp.pos.txt (pathogenic) and snp.neg.txt (benign) test files. Generate features for the test set with R script prepare.R (producing annot.csv with 45 CADD annotations plus reference/alternative FASTA sequences, using CADD.indel.rda of all 1000 Genomes indels) and python prepare.py (running the pretrained DanQ model to predict 919 multi-omics features into seq_features.h5). Train TIVAN-indel with train.py on GTEx cis-eQEL-derived labeled nc-sindels, combining the 45 CADD annotations and 919 multi-omics features, then score the ClinVar test set with predict.py --model ../res/model., writing scores to ../res/output.txt. Repeat prediction for baseline methods (CADD/FATHMM-indel SVM, DanQ/DeepSEA boosted LR) under identical 5-fold cross-validation splits, compute AUROC and AUPRC per fold, average across folds, and write auroc_auprc_comparison.csv.

## Related tools

- **TIVAN-indel** (Supervised framework whose trained model (train.py, predict.py) scores the curated ClinVar nc-sindel test set for AUROC/AUPRC benchmarking) — https://github.com/lichen-lab/TIVAN-indel
- **R** (Runs prepare.R to generate CADD annotations (annot.csv) and reference/alternative FASTA sequences for the test sindels using CADD.indel.rda)
- **Python** (Runs prepare.py (DanQ prediction of 919 multi-omics features), train.py and predict.py to score the test set into ../res/output.txt)

## Examples

```
R --slave --args --no-save CADD.indel.rda test/snp.pos.txt test/snp.neg.txt test < prepare.R && python prepare.py --path ../test && python predict.py --path ../test --model ../res/model. --output ../res/output.txt
```

## Evaluation signals

- Test set composition matches the reported 132 pathogenic vs 1104 benign nc-sindels derived from ClinVar
- All AUROC and AUPRC values fall within [0,1] and TIVAN-indel attains the highest mean AUROC (Fig. 6) and AUPRC (Supplementary Fig. S7) across the compared methods
- Test input files follow the 'chr start end refallele altallele' format and every test indel is < 100 bp and non-coding on hg19
- Feature generation yields annot.csv with 45 CADD annotations and seq_features.h5 with 919 multi-omics features for the test set
- All methods are evaluated on identical 5-fold cross-validation splits with results averaged per fold in auroc_auprc_comparison.csv

## Limitations

- Model and test curation are tied to hg19 (BSgenome.Hsapiens.UCSC.hg19); lifting variants to another build is outside the supported workflow
- Only non-coding small indels shorter than 100 bp with high ClinVar review status are eligible; other variant classes are excluded
- Benchmark conclusions depend on ClinVar curation quality; the training labels come from GTEx cis-eQTL analyses across 44 tissues, so disease-pathogenic labels are conceptually different from regulatory-activity labels

## Evidence

- [results] Using 5-fold cross-validation on a ClinVar-derived set of 132 pathogenic and 1104 benign nc-sindels, TIVAN-indel outperforms CADD/FATHMM-indel (SVM) and DanQ/DeepSEA (Boosted LR) by achieving the highest AUROC (Fig. 6) and AUPRC (Supplementary Fig. S7).: [evidence span withheld — anchor status: not_found]
- [readme] R script `prepare.R` will generated (1) CADD annotations for query sindels and (2) reference genomic sequence and alterntive genomic sequence with sindels: "R script `prepare.R` will generated (1) CADD annotations for query sindels and (2) reference genomic sequence and alterntive genomic"
- [readme] It uses a compilation of both 45 CADD annotations and 919 multi-omics features as generated above.: "It uses a compilation of both 45 CADD annotations and 919 multi-omics features as generated above"
- [abstract] By leveraging labeled nc-sindels identified by cis-expression quantitative trait loci analyses across 44 tissues in Genotype-Tissue Expression (GTEx): "By leveraging labeled nc-sindels identified by cis-expression quantitative trait loci analyses across 44 tissues in Genotype-Tissue Expression"
- [readme] a file with indel information "chr start end refallele and altallele" in the positve set: "a file with indel information "chr start end refallele and altallele" in the positve set"
