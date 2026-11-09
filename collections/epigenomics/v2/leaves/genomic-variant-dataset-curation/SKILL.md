---
name: genomic-variant-dataset-curation
description: Use when building a supervised deep learning training set of non-coding
  variants requiring curated positive (dsQTL, eQTL, HGMD) and frequency-matched negative
  SNPs, as for ncDeepBrain.
when_to_use_negative:
- Input is already an assembled, labeled feature/variant training table — re-curation
  would duplicate the workflow
- The goal is coding-variant prioritization, since these datasets target non-coding
  regulatory SNPs affecting brain functions
- No negative-matching resource (1000 Genomes allele frequencies) is available, since
  frequency matching is central to the design
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3517
tools:
- name: deltaSVM
  role: source of dsQTL positive/negative training SNPs for training dataset 1
- name: GWAVA
  role: source of the HGMD-derived non-coding regulatory SNP positive set (April 2012
    release)
  repo: https://www.sanger.ac.uk/sanger/StatGen_Gwava
- name: DeepSEA
  role: generation of associated SNPs at P-value cutoff 1 × 10^-10 from GRASP non-coding
    eQTLs for training dataset 4
- name: 1000 Genomes Project
  role: source of frequency-matched negative/background SNVs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:2796eee4064b178068b498cdc8bdcd37cfddbffab149714392fd6b53d29d21b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 1000 Genomes Project
  - DeepSEA
  - GWAVA
  - deltaSVM
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# genomic-variant-dataset-curation

## Summary

Assembles labeled positive/negative SNP training datasets for non-coding variant prioritization models (e.g., ncDeepBrain in iMEGES) by downloading disease/functional QTL positives from resources like deltaSVM, HGMD, brain eQTLs and GRASP, and matching them with frequency-matched negatives from 1000 Genomes Project.

## When to use

Use when building a supervised deep learning training set of non-coding variants requiring curated positive (dsQTL, eQTL, HGMD) and frequency-matched negative SNPs, as for ncDeepBrain.

## When NOT to use

- Input is already an assembled, labeled feature/variant training table — re-curation would duplicate the workflow
- The goal is coding-variant prioritization, since these datasets target non-coding regulatory SNPs affecting brain functions
- No negative-matching resource (1000 Genomes allele frequencies) is available, since frequency matching is central to the design

## Inputs

- dsQTL training data from the deltaSVM resource
- 1614 non-coding regulatory SNPs from the HGMD April 2012 release (as used by GWAVA)
- 31,118 functional eQTLs from joint test of 7 brain tissues/cell lines across eleven studies (e.g., CommonMind brain eQTLs)
- GRASP non-coding eQTLs (associated SNPs selected by DeepSEA at P-value cutoff 1 × 10^-10)
- 1000 Genomes Project SNVs for negative/background sampling

## Outputs

- Four tab-delimited variant-identifier files with positive/negative labels (counts: 574/27,735; 1614 positives; 31,118/36,540)
- Frequency-matched negative SNP sets from 1000 Genomes Project

## How to apply

Download the dsQTL training data from the deltaSVM resource, retaining 574 dsQTL positive SNPs and 27,735 negative SNPs with minor allele frequency (MAF) > 5% located in dsQTL regions. For a second dataset, take the 1614 non-coding regulatory SNPs from the HGMD April 2012 release (as used by GWAVA) as positives and randomly select negative common SNVs with MAF > 1% from the 1000 Genomes Project. For eQTL-based sets, download 31,118 functional eQTLs from the joint test of 7 brain tissues/cell lines across eleven studies, paired with 36,540 frequency-matched background SNPs sampled around the nearest TSS of randomly selected genes. A fourth set pairs DeepSEA-generated associated SNPs (P-value cutoff 1 × 10^-10 from GRASP non-coding eQTLs) with non-associated SNPs from 1000 Genomes matched to the associated SNPs' MAF distribution. Write each assembled set to a tab-delimited file of variant identifiers with positive/negative labels; the decision point is matching negatives on allele frequency so class labels reflect function, not frequency differences.

## Related tools

- **deltaSVM** (source of dsQTL positive/negative training SNPs for training dataset 1)
- **GWAVA** (source of the HGMD-derived non-coding regulatory SNP positive set (April 2012 release)) — https://www.sanger.ac.uk/sanger/StatGen_Gwava
- **DeepSEA** (generation of associated SNPs at P-value cutoff 1 × 10^-10 from GRASP non-coding eQTLs for training dataset 4)
- **1000 Genomes Project** (source of frequency-matched negative/background SNVs)

## Examples

```
python ncDeepBrain.py --help
```

## Evaluation signals

- Row counts of each output tab-delimited file equal the reported counts: 574 positives / 27,735 negatives (dataset 1), 1614 positives (dataset 2), 31,118 positives / 36,540 negatives (dataset 3)
- All negative SNPs in dataset 1 have MAF > 5% and lie in dsQTL regions
- Dataset 2 negatives are common SNVs (MAF > 1%) from 1000 Genomes Project
- Background SNPs for eQTL sets are frequency-matched and sampled around the nearest TSS of randomly selected genes
- Each file contains variant identifiers plus explicit positive/negative labels

## Limitations

- It is challenging to obtain large-scale high quality data for training statistical/deep learning models, so negative sampling choices strongly influence results
- Imputed GWAS-derived hits may represent proxy markers rather than true causal variants, making training labels less reliable than ideal

## Evidence

- [methods] Training dataset 1 for variant prioritization in iMEGES consists of 574 dsQTL positive SNPs downloaded via deltaSVM and 27,735 negative SNPs with MAF > 5% in dsQTL regions.: [evidence span withheld — anchor status: not_found]
- [methods] Download the 1614 non-coding regulatory SNPs from the HGMD April 2012 release (as used by GWAVA) and randomly select negative common SNVs with MAF > 1% from the 1000 Genomes Project.: [evidence span withheld — anchor status: not_found]
- [results] The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1: "The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1"
- [methods] Write each of the four assembled datasets to a tab-delimited file with variant identifiers and positive/negative labels.: [evidence span withheld — anchor status: not_found]
- [discussion] First, it is challenging to obtain large-scale high quality data for training statistical models.: "First, it is challenging to obtain large-scale high quality data for training statistical models."
