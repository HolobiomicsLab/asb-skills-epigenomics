---
name: functional-variant-annotation-annovar
description: Use when non-coding SNVs must be converted from VCF/BED coordinates into
  a per-variant functional feature matrix to train or apply a brain-disorder variant
  prioritization score like ncDeepBrain.
when_to_use_negative:
- The task is step-2 gene prioritization, which instead integrates ncDeepBrain score
  with RVIS, GTEx, haploinsufficiency, Phenolyzer, CNV and de novo mutation gene-level
  scores
- Input is already an assembled feature table for model training, so re-annotation
  with ANNOVAR is redundant
- Variants are coding; this skill targets whole-genome non-coding variants for brain-function
  impact scoring
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: ANNOVAR
  role: Annotates the dsQTL training SNPs and input variants in Annovar/bed format
    to assemble the feature matrix
  repo: http://annovar.openbioinformatics.org/en/latest/
- name: CADD
  role: One of the five non-coding variant predictor scores integrated into the feature
    matrix
  repo: http://cadd.gs.washington.edu/
- name: DANN
  role: One of the five non-coding variant predictor scores integrated into the feature
    matrix
  repo: https://cbcl.ics.uci.edu/public_data/DANN/
- name: GWAVA
  role: One of the five non-coding variant predictor scores integrated into the feature
    matrix
  repo: https://www.sanger.ac.uk/sanger/StatGen_Gwava
- name: FATHMM-MKL
  role: One of the five non-coding variant predictor scores integrated into the feature
    matrix
  repo: http://fathmm.biocompute.org.uk/
- name: bPCA
  role: R code (-B bPCA.R) that imputes missing predictor scores in the feature matrix
- name: TensorFlow
  role: Backend for the Keras deep learning model consuming the annotated features
    to produce the ncDeepBrain score
  repo: https://github.com/WGLab/iMEGES
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:d02a47c5531db89bd7ee66f831be0fd21f94c89b8cf1150cf94dd566f19748bc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - CADD
  - DANN
  - FATHMM-MKL
  - GWAVA
  - TensorFlow
  - bPCA
  - deltaSVM
  - python scikit
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# Functional variant annotation with ANNOVAR

## Summary

Annotate non-coding variants with ANNOVAR and assemble the iMEGES 11-feature matrix (EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD frequency, brain eQTLs, and PsychENCODE/Roadmap enhancer-promoter indicators) that feeds the ncDeepBrain deep learning variant score.

## When to use

Use when non-coding SNVs must be converted from VCF/BED coordinates into a per-variant functional feature matrix to train or apply a brain-disorder variant prioritization score like ncDeepBrain.

## When NOT to use

- The task is step-2 gene prioritization, which instead integrates ncDeepBrain score with RVIS, GTEx, haploinsufficiency, Phenolyzer, CNV and de novo mutation gene-level scores
- Input is already an assembled feature table for model training, so re-annotation with ANNOVAR is redundant
- Variants are coding; this skill targets whole-genome non-coding variants for brain-function impact scoring

## Inputs

- Non-coding variant list in Annovar input or BED format (e.g. input.bed)
- Genome build identifier (hg version) for ANNOVAR annotation
- List of known brain eQTLs from the CommonMind project
- PsychENCODE and Roadmap Epigenomics brain enhancer/promoter region tracks (H3K4me3, H3K4me1, H3K27me3, H3K27ac)
- Pre-computed EIGEN, CADD, DANN, GWAVA, FATHMM scores and GNOMAD allele frequencies

## Outputs

- Per-variant 11-feature annotation matrix (five predictor scores, GNOMAD frequency, brain eQTL score, four binary epigenomic indicators)
- Imputed and class-balanced feature table ready for ncDeepBrain training/inference

## How to apply

Annotate the variant set (e.g. the 574 positive and 27,735 negative MAF>5% dsQTL training SNPs) with ANNOVAR using the genome build passed via --buildver, converting to Annovar input/bed format with -f1. Build the 11-feature matrix: EIGEN, CADD, DANN, GWAVA, FATHMM scores, GNOMAD frequency, a brain eQTL score from CommonMind (default 0.5 when no known brain eQTL exists), and binary enhancer/promoter indicators for H3K4me3, H3K4me1, H3K27me3, and H3K27ac regions from PsychENCODE and Roadmap Epigenomics brain tissues. Impute missing DANN, CADD, EIGEN, GWAVA, and FATHMM values with bPCA fitted on non-missing genome-wide scores, then balance the 574-vs-27,735 class imbalance with the ROSE R library before feeding the matrix to the Keras/TensorFlow network.

## Related tools

- **ANNOVAR** (Annotates the dsQTL training SNPs and input variants in Annovar/bed format to assemble the feature matrix) — http://annovar.openbioinformatics.org/en/latest/
- **CADD** (One of the five non-coding variant predictor scores integrated into the feature matrix) — http://cadd.gs.washington.edu/
- **DANN** (One of the five non-coding variant predictor scores integrated into the feature matrix) — https://cbcl.ics.uci.edu/public_data/DANN/
- **GWAVA** (One of the five non-coding variant predictor scores integrated into the feature matrix) — https://www.sanger.ac.uk/sanger/StatGen_Gwava
- **FATHMM-MKL** (One of the five non-coding variant predictor scores integrated into the feature matrix) — http://fathmm.biocompute.org.uk/
- **bPCA** (R code (-B bPCA.R) that imputes missing predictor scores in the feature matrix)
- **TensorFlow** (Backend for the Keras deep learning model consuming the annotated features to produce the ncDeepBrain score) — https://github.com/WGLab/iMEGES

## Examples

```
python ncDeepBrain.py -f1 input.bed -E brain_eqtls.bed -B bPCA.R -o out --buildver hg19
```

## Evaluation signals

- Every input variant resolves to a complete or bPCA-imputed value across all 11 features; brain eQTL score defaults to 0.5 only when no CommonMind brain eQTL is known
- Binary enhancer/promoter indicators are 0/1 and trace to H3K4me3, H3K4me1, H3K27me3, H3K27ac tracks from PsychENCODE or Roadmap Epigenomics brain tissues
- Downstream ncDeepBrain ten-fold cross-validation ROC should reproduce approximately 80% AUC on deltaSVM training dataset 1
- Annotation output coordinates match the submitted -f1 bed file and the specified --buildver genome version

## Limitations

- Some procedures rely on imputed GWAS data, where hits may be proxy markers rather than true causal variants, making the model less reliable than ideal
- Non-coding variants are assigned to their closest gene, a strategy that may work well for promoters but is less optimal for other regulatory elements
- It is challenging to obtain large-scale high-quality data for training such statistical/deep learning models

## Evidence

- [readme] The Annovar input/bed format file: "The Annovar input/bed format file"
- [readme] README dependencies list ANNOVAR as the first dependency of iMEGES: "Dependency

Annovar http://annovar.openbioinformatics.org/en/latest/"
- [readme] README describes step 1 integrating five predictor scores plus GNOMAD, brain eQTLs and enhancers: "We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs"
- [results] Article results identify enhancer/promoter regions source: "enhancer/promoter regions from the PsychENOCDE and RoadMap Epigenomics projects"
- [results] Article results report the ncDeepBrain AUC benchmark: "The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1"
- [intro] Article intro frames the deep learning variant score objective: "we used a deep learning approach to build a whole genome variant score for variants which affect brain functions"
