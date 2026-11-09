---
name: eqtl-dataset-assembly
description: Use when building labeled positive/negative SNP training sets for a non-coding
  variant prioritization model, sourcing from deltaSVM dsQTLs, HGMD regulatory SNPs,
  brain eQTLs, or GRASP/DeepSEA-associated SNPs with matched negatives.
when_to_use_negative:
- Your model input is already a curated feature table or pre-scored variant set —
  this skill is only for assembling the raw labeled training sets
- You need coding variant prioritization — these sets target whole-genome non-coding
  variants; gene prioritization (step 2 of iMEGES) uses different gene-based scores
- You lack access to the upstream sources (deltaSVM, HGMD April 2012, GRASP via DeepSEA,
  1000 Genomes) and cannot reproduce the exact counts or frequency matching
tools:
- name: deltaSVM
  role: Source of the dsQTL training data used to construct training dataset 1 (ncDeepBrain)
- name: GWAVA
  role: Reference predictor whose HGMD-based training set (April 2012 non-coding regulatory
    SNPs) is reused as training dataset 2
  repo: https://www.sanger.ac.uk/sanger/StatGen_Gwava
- name: DeepSEA
  role: Predictor whose associated-SNP calls on GRASP non-coding eQTLs (P-value cutoff
    1 × 10^-10) form the positives of training dataset 4
- name: iMEGES
  role: Downstream deep learning tool whose ncDeepBrain step consumes the assembled
    training datasets
  repo: https://github.com/WGLab/iMEGES
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:9c08108d2d359a1e9c3b8e1601589443c7edfb4e84c19afccc8ae86977f7d14a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSEA
  - GWAVA
  - deltaSVM
  - iMEGES
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# eqtl-dataset-assembly

## Summary

Assembles the four labeled variant-prioritization training datasets used by iMEGES/ncDeepBrain (dsQTL, HGMD non-coding regulatory, brain eQTL, and DeepSEA-derived GRASP sets), each written as a tab-delimited file with variant identifiers and positive/negative labels. Accurate assembly is critical because class sizes (e.g. 574 positives vs 27,735 negatives) directly drive the deep learning variant score.

## When to use

Use when building labeled positive/negative SNP training sets for a non-coding variant prioritization model, sourcing from deltaSVM dsQTLs, HGMD regulatory SNPs, brain eQTLs, or GRASP/DeepSEA-associated SNPs with matched negatives.

## When NOT to use

- Your model input is already a curated feature table or pre-scored variant set — this skill is only for assembling the raw labeled training sets
- You need coding variant prioritization — these sets target whole-genome non-coding variants; gene prioritization (step 2 of iMEGES) uses different gene-based scores
- You lack access to the upstream sources (deltaSVM, HGMD April 2012, GRASP via DeepSEA, 1000 Genomes) and cannot reproduce the exact counts or frequency matching

## Inputs

- dsQTL training data from the deltaSVM resource
- 1614 non-coding regulatory SNPs from the HGMD April 2012 release (as used by GWAVA)
- 31,118 functional eQTLs from the joint test of 7 brain tissues/cell lines across eleven studies
- DeepSEA-associated SNPs (P-value cutoff 1 × 10^-10) from GRASP non-coding eQTLs
- Negative SNVs from the 1000 Genomes Project with allele frequency thresholds (MAF > 5% for dataset 1, MAF > 1% for dataset 2, frequency-matched for datasets 3–4)

## Outputs

- Four tab-delimited training dataset files, each with variant identifiers and positive/negative labels
- Dataset 1 (ncDeepBrain): 574 dsQTL positive SNPs and 27,735 negative SNPs (MAF > 5% in dsQTL regions)
- Dataset 2: 1614 HGMD positive SNPs with 1000 Genomes negative common SNVs (MAF > 1%)
- Dataset 3: 31,118 positive brain eQTLs and 36,540 frequency-matched background SNPs
- Dataset 4: GRASP-associated SNPs and MAF-matched 1000 Genomes non-associated SNPs

## How to apply

Build dataset 1 by downloading the dsQTL training data from the deltaSVM resource and retaining 574 dsQTL positive SNPs plus 27,735 negative SNPs with minor allele frequency (MAF) > 5% in dsQTL regions. Build dataset 2 by taking the 1614 non-coding regulatory SNPs from the HGMD April 2012 release (the same set used by GWAVA) and randomly selecting negative common SNVs with MAF > 1% from the 1000 Genomes Project. Build dataset 3 from the 31,118 functional eQTLs from a joint test of 7 brain tissues/cell lines across eleven studies, with 36,540 frequency-matched background SNPs sampled around the nearest TSS of randomly selected genes. Build dataset 4 from DeepSEA-generated associated SNPs with P-value cutoff 1 × 10^-10 from GRASP non-coding eQTLs, matched to non-associated SNPs from the 1000 Genomes Project matched to the associated SNPs' minor allele frequency distribution. Write each set to a tab-delimited file with variant identifiers and positive/negative labels, then validate row counts against the paper's training dataset table (574/27,735; 1614 positives; 31,118/36,540).

## Related tools

- **deltaSVM** (Source of the dsQTL training data used to construct training dataset 1 (ncDeepBrain))
- **GWAVA** (Reference predictor whose HGMD-based training set (April 2012 non-coding regulatory SNPs) is reused as training dataset 2) — https://www.sanger.ac.uk/sanger/StatGen_Gwava
- **DeepSEA** (Predictor whose associated-SNP calls on GRASP non-coding eQTLs (P-value cutoff 1 × 10^-10) form the positives of training dataset 4)
- **iMEGES** (Downstream deep learning tool whose ncDeepBrain step consumes the assembled training datasets) — https://github.com/WGLab/iMEGES

## Examples

```
python ncDeepBrain.py --help
```

## Evaluation signals

- Row counts of each output file equal the reported counts in the paper's training dataset table: 574/27,735 for dataset 1, 1614 positives for dataset 2, 31,118/36,540 for dataset 3
- Each output file is tab-delimited and contains exactly variant identifiers plus positive/negative labels
- Negative SNPs in dataset 1 all have minor allele frequency (MAF) > 5% in dsQTL regions; dataset 2 negatives have MAF > 1% from the 1000 Genomes Project
- Dataset 4 non-associated SNPs match the associated SNPs' minor allele frequency distribution
- A classifier trained on dataset 1 reaches an AUC in the expected range — the paper reports ncDeepBrain classification AUC of 80% on the deltaSVM training dataset 1

## Limitations

- Negative sets are random SNVs frequency-matched to positives; they may include unannotated true positives and do not reflect specific mental-disorder functional properties
- Only 1614 HGMD non-coding regulatory positives are available, limiting training-set size; the paper notes the challenge of obtaining large-scale high-quality training data
- The four datasets come from different sources with different class definitions (dsQTL, HGMD, brain eQTL, GRASP/DeepSEA), so they are not interchangeable

## Evidence

- [methods] Download the dsQTL training data from the deltaSVM resource, retaining 574 dsQTL positive SNPs and 27,735 negative SNPs with minor allele frequency (MAF) > 5% in dsQTL regions.: [evidence span withheld — anchor status: not_found]
- [results] The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1: "The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1"
- [readme] The first step of the iMEGES take non-coding variants as input and prioritize the brain non-coding variants.: "The first step of the iMEGES take non-coding variants as input and prioritize the brain non-coding variants."
- [discussion] it is challenging to obtain large-scale high quality data for training statistical models.: "First, it is challenging to obtain large-scale high quality data for training statistical models."
