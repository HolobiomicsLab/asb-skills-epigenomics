---
name: nested-feature-set-performance-comparison
description: Use when evaluating whether adding sequence-derived epigenomic features
  (reference and/or alternative alleles) to generic functional annotations improves
  a tissue-specific variant classifier, using AUROC/AUPRC medians across tissues.
when_to_use_negative:
- Input is already a scored variant set or precomputed tissue-specific functional
  scores (R/TIVAN-indel.R retrieval path), not a training feature matrix requiring
  ablation
- The task is cross-tissue or external validation (e.g., DICE immune cell types),
  rather than with-tissue 5-fold CV across the 44 GTEx tissues
- Variants are not small indels in non-coding regions; the CADD.indel.rda and DanQ
  indel feature pipeline is specific to nc-sindels
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: TIVAN-indel
  role: Supervised framework whose model is retrained under the three nested feature
    sets for the ablation
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: xgboost
  role: Classifier trained per feature set per tissue during 5-fold cross-validation
- name: DanQ model (via prepare.py)
  role: Predicts the 919 multi-omics epigenomic features from reference and alternative
    genomic sequences
  repo: https://github.com/amanbasu/TIVAN-indel
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Used by prepare.R to extract reference and alternative genomic sequences for
    indels
- name: scikit-learn
  role: Computes AUROC/AUPRC per fold, tissue, and feature set
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad060
    title: 'TIVAN-indel: a computational framework for annotating and predicting non-coding
      regulatory small insertions and deletions'
  source_content_hash: sha256:3c865c586d5b8839a0442167208e15580850cb5e260823dc444723f0b4c0a9b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/TIVAN-indel
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - DanQ model (via prepare.py)
  - Python
  - R
  - TIVAN-indel
  - h5py
  - numpy
  - pandas
  - scikit-learn
  - xgboost
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# nested-feature-set-performance-comparison

## Summary

A nested feature-set ablation for TIVAN-indel that trains the xgboost model on three cumulative feature sets (45 CADD annotations only, CADD + 919 reference-sequence multi-omics features, CADD + reference + alternative-sequence features) across 44 GTEx tissues to quantify the contribution of sequence-derived epigenomic profiles to nc-sindel prediction.

## When to use

Use when evaluating whether adding sequence-derived epigenomic features (reference and/or alternative alleles) to generic functional annotations improves a tissue-specific variant classifier, using AUROC/AUPRC medians across tissues.

## When NOT to use

- Input is already a scored variant set or precomputed tissue-specific functional scores (R/TIVAN-indel.R retrieval path), not a training feature matrix requiring ablation
- The task is cross-tissue or external validation (e.g., DICE immune cell types), rather than with-tissue 5-fold CV across the 44 GTEx tissues
- Variants are not small indels in non-coding regions; the CADD.indel.rda and DanQ indel feature pipeline is specific to nc-sindels

## Inputs

- CADD.indel.rda (CADD annotations for all 1000 Genomes indels)
- Positive and negative indel position files (chr, start, end, ref/alt allele), e.g. train/snp.pos.txt and train/snp.neg.txt
- Reference and alternative genomic FASTA sequences per indel (train folder)
- Pretrained DanQ model weights (checkpoint/ directory) for predicting 919 multi-omics features
- GTEx tissue labels defining the 44 tissue-specific cross-validation settings

## Outputs

- annot.csv (CADD annotations for positive and negative indels)
- seq_features.h5 (919 multi-omics features from reference and alternative sequences)
- cv_results.csv with AUROC and AUPRC per fold, tissue, and feature set
- Median AUROC/AUPRC per feature set across 44 GTEx tissues (CADD-only 0.858/0.843; +ref-seq 0.859/0.845; +ref+alt-seq 0.982/0.984)

## How to apply

First generate the CADD annotation table (annot.csv) and reference/alternative FASTA sequences with prepare.R from CADD.indel.rda plus positive/negative indel position files (chr, start, end, ref/alt allele). Then run src/prepare.py with the pretrained DanQ model to predict 919 multi-omics features from reference and alternative sequences into seq_features.h5. Train TIVAN-indel (train.py, xgboost) under the three nested feature sets and run 5-fold cross-validation independently for each of the 44 GTEx tissues (e.g., Adipose_Subcutaneous through Whole_Blood), recording AUROC and AUPRC per fold, tissue, and feature set. Compute the median AUROC and median AUPRC across tissues per feature set and compare against reported Results 3.1 / Figure 2 values (CADD-only 0.858/0.843; +ref-seq 0.859/0.845; +ref+alt-seq 0.982/0.984). The rationale is that the nested design isolates the marginal value of each feature block, with the large jump from ~0.86 to ~0.98 median AUROC attributable to alternative-sequence epigenomic profiles.

## Related tools

- **TIVAN-indel** (Supervised framework whose model is retrained under the three nested feature sets for the ablation) — https://github.com/lichen-lab/TIVAN-indel
- **xgboost** (Classifier trained per feature set per tissue during 5-fold cross-validation)
- **DanQ model (via prepare.py)** (Predicts the 919 multi-omics epigenomic features from reference and alternative genomic sequences) — https://github.com/amanbasu/TIVAN-indel
- **BSgenome.Hsapiens.UCSC.hg19** (Used by prepare.R to extract reference and alternative genomic sequences for indels)
- **scikit-learn** (Computes AUROC/AUPRC per fold, tissue, and feature set)

## Examples

```
R --slave --args --no-save CADD.indel.rda train/snp.pos.txt train/snp.neg.txt train < prepare.R && python prepare.py --path ../train && python train.py --path ../train --output ../res/model.
```

## Evaluation signals

- cv_results.csv median AUROC for CADD-only is ~0.858 and median AUPRC ~0.843, matching Results 3.1 / Figure 2
- Adding reference-sequence features yields only marginal gains (median AUROC ~0.859, AUPRC ~0.845), confirming the reference-only block is near-redundant with CADD annotations
- The +ref+alt-seq feature set achieves the best median AUROC (0.982) and AUPRC (0.984) across all 44 GTEx tissues
- Per-tissue AUROC/AUPRC are recorded for every fold across all 44 tissues (Adipose_Subcutaneous through Whole_Blood) for each of the three feature sets
- seq_features.h5 contains 919 features per sequence (reference and alternative), and annot.csv contains 45 CADD annotations per indel

## Limitations

- Only marginal improvement from reference-sequence features alone (0.859/0.845 vs 0.858/0.843); the performance gain depends critically on including alternative-sequence epigenomic profiles
- The DanQ feature predictor relies on pretrained weights trained on DeepSEA data; training your own model requires data from http://deepsea.princeton.edu/help/
- CADD.indel.rda covers only 1000 Genomes indels, so indels outside that resource lack generic CADD annotations
- Large CADD.indel.rda and sindel.noncoding.1kg.rda files and precomputed tissue scores are hosted externally (Google Drive), not in the repository itself

## Evidence

- [methods] 919 multi-omics features from reference/alternative genomic sequences: "File `src/prepare.py` calls DanQ model to predict 919 multi-omics features"
- [readme] three nested feature sets in model: "It uses a compilation of both 45 CADD annotations and 919 multi-omics features as generated above. We call ``train.py" to train the model"
- [readme] sequence generation inputs: "R script ``prepare.R" will generated (1) CADD annotations for query sindels and (2) reference genomic sequence and alterntive genomic sequence with sindels"
- [results] ablation result: [evidence span withheld — anchor status: not_found]
- [abstract] labeled data source: "By leveraging labeled nc-sindels identified by cis-expression quantitative trait loci analyses across 44 tissues in Genotype-Tissue Expression (GTEx)"
