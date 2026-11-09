---
name: positive-negative-variant-labeling
description: Use when constructing supervised training data for non-coding variant
  prioritization, i.e. pairing known functional/eQTL/disease SNPs (positives) with
  frequency-matched common background SNVs (negatives) in tab-delimited variant files.
when_to_use_negative:
- Input is already a labeled training table or a pre-trained score output (e.g. ncDeepBrain
  scores ready for gene prioritization) rather than raw variant source files
- The task is coding-variant impact prediction, since ncDeepBrain and these training
  sets target whole-genome non-coding variants for brain functions
- You cannot frequency-match negatives (MAF thresholds/matching to the associated
  SNPs' allele frequency distribution), since unbalanced label sets degrade the classifier
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: deltaSVM
  role: Source resource for training dataset 1 (dsQTL positive and negative SNPs)
- name: GWAVA
  role: Source of the HGMD-based non-coding regulatory SNP positive set used in training
  repo: https://www.sanger.ac.uk/sanger/StatGen_Gwava
- name: DeepSEA
  role: Used to generate associated SNPs (P-value cutoff 1 × 10^-10) from GRASP non-coding
    eQTLs for the fourth training dataset
- name: iMEGES
  role: Consumer of the labeled datasets; ncDeepBrain deep neural network is trained
    on them
  repo: https://github.com/WGLab/iMEGES
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:bc1d8cbfdc56518378d8e3f4e0d0b511cb4960c36d0f745077d5ddb260e6ce06
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSEA
  - GWAVA
  - deltaSVM
  - iMEGES
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# positive-negative-variant-labeling

## Summary

Assemble class-labeled training sets of positive (disease-associated or functional) and negative (frequency-matched common) non-coding SNPs for training the ncDeepBrain deep neural network in iMEGES, e.g. 574 dsQTL positives vs 27,735 MAF>5% negatives from the deltaSVM resource.

## When to use

Use when constructing supervised training data for non-coding variant prioritization, i.e. pairing known functional/eQTL/disease SNPs (positives) with frequency-matched common background SNVs (negatives) in tab-delimited variant files.

## When NOT to use

- Input is already a labeled training table or a pre-trained score output (e.g. ncDeepBrain scores ready for gene prioritization) rather than raw variant source files
- The task is coding-variant impact prediction, since ncDeepBrain and these training sets target whole-genome non-coding variants for brain functions
- You cannot frequency-match negatives (MAF thresholds/matching to the associated SNPs' allele frequency distribution), since unbalanced label sets degrade the classifier

## Inputs

- dsQTL positive SNPs and negative SNPs from the deltaSVM resource
- 1614 non-coding regulatory SNPs from the HGMD April 2012 release (as used by GWAVA)
- 31,118 functional eQTLs from a joint test of 7 brain tissues/cell lines across eleven studies
- DeepSEA-associated SNPs from GRASP non-coding eQTLs (P-value cutoff 1 × 10^-10)
- 1000 Genomes Project SNVs for negative/background sampling
- BED/Annovar-format variant files accepted by ncDeepBrain

## Outputs

- Four tab-delimited training datasets with variant identifiers and positive/negative labels (expected counts: 574/27,735; 1614 positives; 31,118/36,540)
- Labeled variant training data feeding ncDeepBrain (reported classification AUC of 80% on training dataset 1)

## How to apply

Download training data from each source resource and attach binary positive/negative labels. For training dataset 1 (deltaSVM/dsQTL), retain 574 dsQTL positive SNPs and 27,735 negative SNPs with minor allele frequency (MAF) > 5% located in dsQTL regions. For other datasets, pair 1614 non-coding regulatory HGMD SNPs (April 2012 release, as used by GWAVA) with random 1000 Genomes negatives of MAF > 1%; pair 31,118 functional brain eQTLs with 36,540 frequency-matched background SNPs sampled around the nearest TSS of randomly selected genes; and pair DeepSEA-associated GRASP non-coding eQTLs (P-value cutoff 1 × 10^-10) with 1000 Genomes SNPs matched to the associated SNPs' minor allele frequency distribution. Write each assembled dataset as a tab-delimited file containing variant identifiers and positive/negative labels, then validate row counts against the reported values before training ncDeepBrain.

## Related tools

- **deltaSVM** (Source resource for training dataset 1 (dsQTL positive and negative SNPs))
- **GWAVA** (Source of the HGMD-based non-coding regulatory SNP positive set used in training) — https://www.sanger.ac.uk/sanger/StatGen_Gwava
- **DeepSEA** (Used to generate associated SNPs (P-value cutoff 1 × 10^-10) from GRASP non-coding eQTLs for the fourth training dataset)
- **iMEGES** (Consumer of the labeled datasets; ncDeepBrain deep neural network is trained on them) — https://github.com/WGLab/iMEGES

## Examples

```
python ncDeepBrain.py -f1 variants.bed -E brain_eqtls.bed -o ncDeepBrain_scores.bed -buildver hg19
```

## Evaluation signals

- Row counts of each tab-delimited output file match the reported counts: 574 positives / 27,735 negatives (dataset 1), 1614 positives, 31,118 positives / 36,540 negatives
- Negative SNPs in dataset 1 all satisfy MAF > 5% and lie in dsQTL regions
- ncDeepBrain trained on the assembled labels reproduces the reported classification AUC of 80% on the deltaSVM training dataset 1
- Every output row carries a valid positive/negative label plus a variant identifier, in tab-delimited format
- Fourth dataset's 1000 Genomes negatives match the minor allele frequency distribution of the DeepSEA-associated GRASP eQTL positives

## Limitations

- It is challenging to obtain large-scale high quality data for training statistical models
- Some procedures rely on imputed GWAS data, and many hits represent proxy markers rather than true causal variants, making the model less reliable than ideal
- Negative set sizes and matching depend on the source resource snapshots (e.g. HGMD April 2012 release; 1000 Genomes allele frequencies) and may not generalize to other releases

## Evidence

- [other] Training dataset 1 for variant prioritization in iMEGES consists of 574 dsQTL positive SNPs downloaded via deltaSVM and 27,735 negative SNPs with MAF > 5% in dsQTL regions.: [evidence span withheld — anchor status: not_found]
- [results] ncDeepBrain achieves 80% AUC on the labeled training dataset 1.: "The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1"
- [abstract] Large labeled training data matter for iMEGES performance.: "it achieved improved performance than competing approaches when large training dataset is available"
- [readme] The labeled variant sets feed the deep learning variant prioritization step.: "known brain eQTLs from CommonMind and enhancer/promoters from PsychENCODE and Roadmap Epigenomics projects to prioritize the non-coding variants based on deep learning"
