---
name: allele-frequency-matching-of-variant-sets
description: Use when building labeled variant training sets for a variant-effect
  or disease-association classifier and you need negative controls frequency-matched
  to positives (MAF thresholds or matched MAF distributions) to avoid allele-frequency
  artifacts.
when_to_use_negative:
- Variants are already labeled as disease/neutral, so no new negative sampling is
  needed (e.g. scoring personal genomes at iMEGES inference time)
- The task is gene prioritization (step 2 of iMEGES), which integrates ncDeepBrain,
  RVIS, GTEx and haploinsufficiency scores rather than constructing variant training
  sets
- Only a small positive set exists without a population frequency resource, so a defensible
  frequency-matched negative set cannot be sampled
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: deltaSVM
  role: Source of the dsQTL positive SNPs and negative SNPs with MAF > 5% in dsQTL
    regions for training dataset 1
- name: GWAVA
  role: Defines the 1614 HGMD-derived non-coding regulatory positives used to build
    a second training set with 1000 Genomes MAF > 1% negatives
- name: DeepSEA
  role: Scores GRASP non-coding eQTLs to define associated positives at P-value cutoff
    1 x 10^-10, matched to 1000 Genomes SNPs by minor allele frequency distribution
- name: iMEGES
  role: 'Downstream consumer: the four assembled frequency-matched datasets train
    the ncDeepBrain variant-prioritization score'
  repo: https://github.com/WGLab/iMEGES
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:e5157c025cb4a6dd20eb5647602cf086ebcbeb9c8806de94bb66cabc9b8dca14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/iMEGES
  tools:
  - DeepSEA
  - GWAVA
  - deltaSVM
  - iMEGES
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# allele-frequency-matching-of-variant-sets

## Summary

Constructs positive/negative training sets for non-coding variant classifiers (ncDeepBrain in iMEGES) by sampling negative SNVs from population resources (1000 Genomes, GNOMAD) so their minor allele frequency distribution matches the positive disease/regulatory variants, preventing allele-frequency confounding in deep-learning variant prioritization.

## When to use

Use when building labeled variant training sets for a variant-effect or disease-association classifier and you need negative controls frequency-matched to positives (MAF thresholds or matched MAF distributions) to avoid allele-frequency artifacts.

## When NOT to use

- Variants are already labeled as disease/neutral, so no new negative sampling is needed (e.g. scoring personal genomes at iMEGES inference time)
- The task is gene prioritization (step 2 of iMEGES), which integrates ncDeepBrain, RVIS, GTEx and haploinsufficiency scores rather than constructing variant training sets
- Only a small positive set exists without a population frequency resource, so a defensible frequency-matched negative set cannot be sampled

## Inputs

- dsQTL training data from the deltaSVM resource (574 positives)
- 1614 non-coding regulatory SNPs from HGMD April 2012 release (as used by GWAVA)
- 1000 Genomes Project variant frequencies for drawing matched negatives
- 31,118 functional eQTLs from joint test of 7 brain tissues/cell lines across eleven studies
- GRASP non-coding eQTLs scored by DeepSEA (P-value cutoff 1 x 10^-10)

## Outputs

- Four tab-delimited training dataset files with variant identifiers and positive/negative labels (574/27,735; 1614 positives + 1000G negatives; 31,118/36,540; GRASP eQTL positives + MAF-matched 1000G negatives)
- Row counts matching the paper's training dataset table for validation

## How to apply

Assemble four paired positive/negative sets as in iMEGES: (1) retain 574 dsQTL positive SNPs from the deltaSVM resource and sample 27,735 negatives with MAF > 5% inside dsQTL regions; (2) take the 1614 non-coding regulatory SNPs from HGMD April 2012 (GWAVA's positives) and randomly select negative common SNVs with MAF > 1% from the 1000 Genomes Project; (3) pair the 31,118 functional eQTLs from a joint test of 7 brain tissues/cell lines across eleven studies with 36,540 frequency-matched background SNPs sampled around the nearest TSS of randomly selected genes; (4) take DeepSEA-associated SNPs at P-value cutoff 1 x 10^-10 from GRASP non-coding eQTLs and draw non-associated 1000 Genomes SNPs matched to the associated SNPs' minor allele frequency distribution. Write each set as a tab-delimited file with variant identifiers and positive/negative labels, then verify row counts against the paper's training dataset table before training ncDeepBrain.

## Related tools

- **deltaSVM** (Source of the dsQTL positive SNPs and negative SNPs with MAF > 5% in dsQTL regions for training dataset 1)
- **GWAVA** (Defines the 1614 HGMD-derived non-coding regulatory positives used to build a second training set with 1000 Genomes MAF > 1% negatives)
- **DeepSEA** (Scores GRASP non-coding eQTLs to define associated positives at P-value cutoff 1 x 10^-10, matched to 1000 Genomes SNPs by minor allele frequency distribution)
- **iMEGES** (Downstream consumer: the four assembled frequency-matched datasets train the ncDeepBrain variant-prioritization score) — https://github.com/WGLab/iMEGES

## Evaluation signals

- Row counts of each output file equal the reported counts: 574 positives/27,735 negatives (dataset 1), 1614 positives (dataset 2), 31,118 eQTLs/36,540 background SNPs (dataset 3)
- Negatives satisfy their frequency criteria: MAF > 5% in dsQTL regions, MAF > 1% (common) from 1000 Genomes, and frequency-matched to positives' MAF distribution
- Each tab-delimited file contains both variant identifiers and explicit positive/negative labels
- Downstream sanity check: ncDeepBrain trained on dataset 1 (deltaSVM data) reaches classification AUC of 80%

## Limitations

- It is challenging to obtain large-scale high quality data for training statistical models
- Some procedures rely on imputed GWAS data, so hits may be proxy markers rather than true causal variants, making the model less reliable than ideal
- Non-coding variants are assigned to their closest gene, which may be suboptimal for non-promoter regulatory elements

## Evidence

- [results] Random SNVs with allele frequency >= 1% in the 1000 Genomes Project used as negatives in variant prioritization training: "random SNVs with allele frequency ≥ 1% in the 1000 Genomes Project"
- [results] ncDeepBrain classification performance on the deltaSVM training dataset validates the assembled frequency-matched set: "The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1"
- [readme] README lists the frequency resource and score inputs integrated for variant prioritization: "We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from CommonMind"
