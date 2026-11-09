---
name: logistic-regression-parameter-training
description: Use when mapping epigenome peaks for a model organism other than mouse
  (e.g., rat) with no pre-fit parameters, or when refitting parameters on labeled
  data from a specific tissue/experiment.
when_to_use_negative:
- When mapping mouse query regions with the provided 67 ENCODE epigenome datasets,
  since default LOOCV-trained parameters already perform comparably to refit ones
- When inputs are raw sequencing reads or unaligned peaks — candidate target regions
  and similarity scores must be computed before training
- When no matched human-mouse (or human-model-organism) orthologous epigenome datasets
  exist to provide positive/negative labels
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: AdaLiftOver
  role: Provides the training_module() function that estimates logistic regression
    parameters and the optimal score threshold from labeled candidate target regions
  repo: https://github.com/keleslab/AdaLiftOver
- name: R
  role: Implementation language; the rbindlist/lapply workflow aggregates per-tissue
    training results
- name: rtracklayer
  role: Imports the UCSC reciprocal-best chain file before candidate generation
- name: data.table
  role: Fast data manipulation; rbindlist aggregates training results across tissues
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:fe58748f7928a39cd8f57788b819cc6b57a7827c596b6c531e1dc478b72da63c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/keleslab/AdaLiftOver
  tools:
  - AdaLiftOver
  - GenomicRanges
  - Matrix
  - R
  - data.table
  - motifmatchr
  - rtracklayer
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# logistic-regression-parameter-training

## Summary

AdaLiftOver's training_module() estimates logistic regression parameters and an optimal score threshold from a pair of matched epigenome datasets, so candidate target regions can be scored and filtered without relying on the default LOOCV-derived parameters.

## When to use

Use when mapping epigenome peaks for a model organism other than mouse (e.g., rat) with no pre-fit parameters, or when refitting parameters on labeled data from a specific tissue/experiment.

## When NOT to use

- When mapping mouse query regions with the provided 67 ENCODE epigenome datasets, since default LOOCV-trained parameters already perform comparably to refit ones
- When inputs are raw sequencing reads or unaligned peaks — candidate target regions and similarity scores must be computed before training
- When no matched human-mouse (or human-model-organism) orthologous epigenome datasets exist to provide positive/negative labels

## Inputs

- Scored candidate target regions (GRanges / GRangesList) from adaptive_liftover plus similarity computations
- Matched human epigenome peak sets (e.g., epigenome_hg38_test GRangesList) used as ground-truth labels
- Model-organism epigenome repertoire (e.g., epigenome_rn6_test GRangesList from ChIP-Atlas orthologous ChIP-seq datasets)
- UCSC reciprocal-best chain file (e.g., rn6.hg38.rbest.chain)

## Outputs

- Fitted logistic regression parameters (aggregated across tissues via rbindlist of per-tissue training_module results)
- Suggested optimal score threshold with controlled filtering proportion
- Refit-parameter scored candidate target regions and evaluation metrics (AUROC/AUPR), e.g., islet_liftover_evaluation.csv

## How to apply

First run adaptive_liftover() with the appropriate UCSC reciprocal-best chain (e.g., rn6.hg38.rbest.chain) and compute epigenomic signal and sequence grammar similarities to obtain scored candidate target regions. Label candidate target regions in the human genome as positives if they overlap the corresponding human epigenome peaks and as negatives otherwise. Call training_module(gr_candidate, gr_true) on each matched tissue pair, using max_filter_proportion = 0.5 and interaction = FALSE as in the rat example, to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidates. Average the parameters across tissues to obtain organism-level defaults. Validate by comparing AUROC/AUPR under the refit parameters against the default LOOCV-trained parameters (islet case: AUROC 0.879 vs 0.880, AUPR 0.878 vs 0.879).

## Related tools

- **AdaLiftOver** (Provides the training_module() function that estimates logistic regression parameters and the optimal score threshold from labeled candidate target regions) — https://github.com/keleslab/AdaLiftOver
- **R** (Implementation language; the rbindlist/lapply workflow aggregates per-tissue training results)
- **rtracklayer** (Imports the UCSC reciprocal-best chain file before candidate generation)
- **data.table** (Fast data manipulation; rbindlist aggregates training results across tissues)

## Examples

```
training_result <- training_module(gr_candidate_list[[tissue]], epigenome_hg38_test[[tissue]], max_filter_proportion = 0.5, interaction = FALSE)
```

## Evaluation signals

- Refit parameters on islet labeled data yield AUROC 0.879 and AUPR 0.878 vs 0.880/0.879 for default LOOCV parameters — agreement within ~0.001 validates the approach
- training_module returns a suggested score threshold that does not filter out more than max_filter_proportion (0.5) of candidate target regions
- Positive/negative labels are defined by overlap of candidate target regions with the corresponding human epigenome peaks
- Averaged logistic regression parameters across tissues should be usable as organism-level defaults (e.g., for rat)

## Limitations

- Parameters are organism- and repertoire-specific: for organisms other than mouse the default ENCODE-trained parameters may not apply, requiring a user-defined epigenome repertoire
- Threshold tuning depends on max_filter_proportion; setting it too low risks filtering out too many candidate target regions
- Training requires labeled data (overlapping human epigenome peaks), which may be hard-to-obtain for some model organisms/tissues

## Evidence

- [intro] AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions.: "AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions."
- [readme] we first label the candidate target regions in the human genome as positives if they overlap with the corresponding human epigenome peaks and as negatives otherwise.: "we first label the candidate target regions in the human genome as positives if they overlap with the corresponding human epigenome peaks and as negatives otherwise."
- [readme] max_filter_proportion = 0.5, interaction = FALSE: "training_module(
        gr_candidate_list[[tissue]], 
        epigenome_hg38_test[[tissue]], 
        max_filter_proportion = 0.5, 
        interaction = FALSE
    )"
- [results] Performance with parameters tuned on the islet experiment (AUROC 0.879, AUPR 0.878) agreed well with the default LOOCV-trained parameters (AUROC 0.880, AUPR 0.879): [evidence span withheld — anchor status: not_found]
- [intro] **best_k**: retain at most best k candidate target regions for each query region.: "**best_k**: retain at most best k candidate target regions for each query region."
