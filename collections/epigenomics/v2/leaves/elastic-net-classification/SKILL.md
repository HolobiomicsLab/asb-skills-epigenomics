---
name: elastic-net-classification
description: Use when a labeled set of genomic intervals (e.g. HOT vs COLD regions)
  must be classified from sequence-derived features such as k-mer frequencies, GC
  content, and CpG observed/expected ratio, with feature importance ranking.
when_to_use_negative:
- When the goal is to assess whether HOT region signal reflects true TF binding vs
  ChIP-seq artifact (e.g. KO TF ChIP-seq controls) — sequence classification alone
  cannot resolve that
- When regions lack paired class labels or the input is already a curated feature
  table rather than raw interval sets plus sequences
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: glmnet
  role: Fits the binomial elastic net classifiers with 10-fold cross-validation to
    tune lambda, and provides the coefficients used as feature importances
- name: HOT-region-main.R
  role: Main analysis script in the HOT regions manuscript repository containing the
    analysis pipeline
  repo: BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:179f695307e97529d13bb247dcac831dc1c5e9636971a9e333cc29b8f670a453
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
  tools:
  - HOT-region-main.R
  - glmnet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# elastic-net-classification

## Summary

Trains species-specific elastic net classifiers (glmnet, binomial family, 10-fold CV) to discriminate HOT from COLD genomic regions using sequence-derived features (k-mer frequencies for k=2–4, GC content, GC skew, CpG O/E ratio), yielding cross-validation AUCs and ranked feature importances.

## When to use

Use when a labeled set of genomic intervals (e.g. HOT vs COLD regions) must be classified from sequence-derived features such as k-mer frequencies, GC content, and CpG observed/expected ratio, with feature importance ranking.

## When NOT to use

- When the goal is to assess whether HOT region signal reflects true TF binding vs ChIP-seq artifact (e.g. KO TF ChIP-seq controls) — sequence classification alone cannot resolve that
- When regions lack paired class labels or the input is already a curated feature table rather than raw interval sets plus sequences

## Inputs

- HOT region set (BED/interval list) from the upstream HOT-region detection step
- COLD (non-HOT) region set (BED/interval list)
- Reference genome sequences corresponding to the region coordinates

## Outputs

- Feature-importance table of top-ranked elastic net coefficients per species-specific model
- Cross-validated AUC value for each of the four elastic net classifiers (expected range 0.82–0.94)

## How to apply

Load the HOT and COLD region sets produced in the upstream step and extract their genomic sequences. Compute k-mer frequency vectors for k=2, 3, and 4 over each region, plus per-region GC content, GC skew, and CpG observed/expected (O/E) ratio, and assemble these into a feature matrix with HOT/COLD class labels. Train four elastic net classifiers with glmnet (binomial family), each tuned by 10-fold cross-validation to select the regularization parameter lambda. Extract the top-ranked coefficients from each fitted model into a feature-importance table and record the cross-validated AUC per classifier. The rationale is that GC/CpG-rich k-mers and CpG O/E were found to be discriminatory of HOT regions, so the model both predicts class and highlights the sequence features shared across species.

## Related tools

- **glmnet** (Fits the binomial elastic net classifiers with 10-fold cross-validation to tune lambda, and provides the coefficients used as feature importances)
- **HOT-region-main.R** (Main analysis script in the HOT regions manuscript repository containing the analysis pipeline) — BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions

## Examples

```
cvfit <- cv.glmnet(x, y, family="binomial", type.measure="auc", nfolds=10); coef(cvfit, s="lambda.min")
```

## Evaluation signals

- Cross-validated AUCs of the four species-specific classifiers fall within the reported 0.82–0.94 range
- CpG/GC-rich k-mers together with the CpG observed/expected (O/E) ratio rank among the top features in every fitted model
- Regularization parameter lambda is selected via 10-fold cross-validation, not fixed arbitrarily
- Feature matrix contains k-mer frequencies for k=2, 3, 4 plus GC content, GC skew, and CpG O/E ratio for every region with a HOT/COLD label
- Top coefficients are extracted from each fitted glmnet model into a comparable per-species importance table

## Limitations

- HOT regions are likely to be ChIP-seq artifacts similar to 'hyper-ChIPable' regions, so the classifier may learn features of artifact-prone sequence (GC/CpG-rich, R-loops, G-quadruplex DNA) rather than true TF binding
- Technical conditions such as low number of reads, low library complexity, or unsuccessful IP can confound labels derived from ChIP-seq-based region definitions
- With currently available data it is not possible to estimate the proportion of antibody-specific error from pull-down of non-target proteins vs. direct binding to polynucleotide epitopes, limiting biological interpretation of learned features

## Evidence

- [abstract] discriminatory sequence features: "We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures"
- [intro] cross-species sequence comparison motivation: "To date, there has been no cross-species comparison of HOT regions in terms of sequence features."
- [readme] analysis script: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
