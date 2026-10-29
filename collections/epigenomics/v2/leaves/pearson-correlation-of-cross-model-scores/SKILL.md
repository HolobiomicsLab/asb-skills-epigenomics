---
name: pearson-correlation-of-cross-model-scores
description: Use when two sequence-based variant-effect models (e.g. DeepFun and DeepSEA)
  have been run on the same SNP set and you must quantify per-assay score agreement,
  such as CTCF, DNase-seq, H3K4me3, or H3K9ac concordance.
when_to_use_negative:
- You need to predict variant effects de novo rather than compare agreement between
  two existing models' outputs — DeepFun's web server or the SAD workflow alone is
  the right choice
- Your variants target only transcription factor binding sites beyond CTCF — the DeepFun_best_A.th.gz
  model used here covers accessibility, histone marks and CTCF; DeepFun_best_B.th.gz
  or dnacnn_best_B.zip covers broader TF binding sites
- Your goal is calibration or accuracy against experimental labels rather than cross-model
  consistency — PCC between two model outputs measures concordance, not correctness
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: DeepFun
  role: Pre-trained Basset-based CNN model whose SAD scores form one side of the correlation;
    model DeepFun_best_A.th.gz is downloaded from bioinfo.uth.edu and run locally
    via Torch7/Python
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Framework supplying the SAD computation workflow (sad.ipynb tutorial) used
    to compute SNP Activity Difference profiles for DeepFun scores
  repo: https://github.com/davek44/Basset
- name: DeepSEA
  role: External deep learning predictor whose SAD scores for the same variants and
    assays form the other side of the per-assay Pearson correlation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:cec2e048f340cee1a4ae09c58f962933614419760814e7dd256111b4f90ae769
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  - DeepSEA
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pearson-correlation-of-cross-model-scores

## Summary

Cross-model concordance analysis: compute Pearson correlation coefficients (PCC) between per-assay averaged SAD scores produced by DeepFun (Basset framework, Torch7) and DeepSEA for the same variant set, to quantify agreement between two deep learning variant-effect predictors.

## When to use

Use when two sequence-based variant-effect models (e.g. DeepFun and DeepSEA) have been run on the same SNP set and you must quantify per-assay score agreement, such as CTCF, DNase-seq, H3K4me3, or H3K9ac concordance.

## When NOT to use

- You need to predict variant effects de novo rather than compare agreement between two existing models' outputs — DeepFun's web server or the SAD workflow alone is the right choice
- Your variants target only transcription factor binding sites beyond CTCF — the DeepFun_best_A.th.gz model used here covers accessibility, histone marks and CTCF; DeepFun_best_B.th.gz or dnacnn_best_B.zip covers broader TF binding sites
- Your goal is calibration or accuracy against experimental labels rather than cross-model consistency — PCC between two model outputs measures concordance, not correctness

## Inputs

- DeepFun per-variant SAD scores from the pre-trained model DeepFun_best_A.th.gz (DNA accessibility sites, histone marks and CTCF binding sites)
- DeepSEA per-variant SAD scores for the same variant set and same assays
- Variant set (SNPs with reference and alternate alleles) shared by both models

## Outputs

- Per-assay Pearson correlation coefficients (CTCF, DNase-seq, H3K4me3, H3K9ac) between DeepFun and DeepSEA averaged SAD scores
- CSV file deepfun_deepsea_sad_pcc.csv containing the four per-assay PCC values

## How to apply

Run the DeepFun pre-trained model DeepFun_best_A.th.gz (covering DNA accessibility, histone marks and CTCF binding) through the Basset-framework SAD tutorial (sad.ipynb) to compute SNP Activity Difference profiles between reference and alternate alleles for each variant. Obtain DeepSEA SAD scores for the same variants and assays. Average SAD scores across all output tracks belonging to each assay (CTCF, DNase-seq, H3K4me3, H3K9ac) separately for DeepFun and DeepSEA. Compute the Pearson correlation coefficient per assay between the two models' averaged scores using scipy.stats.pearsonr, and write the four PCC values to a CSV (deepfun_deepsea_sad_pcc.csv). Correct application is judged by reproducing the reported values: CTCF PCC = 0.883 (P = 2.2 × 10−16), DNase-seq PCC = 0.751 (P = 1.46 × 10−10), H3K4me3 PCC = 0.490, H3K9ac PCC = 0.451.

## Related tools

- **DeepFun** (Pre-trained Basset-based CNN model whose SAD scores form one side of the correlation; model DeepFun_best_A.th.gz is downloaded from bioinfo.uth.edu and run locally via Torch7/Python) — https://github.com/bsml320/DeepFun
- **Basset** (Framework supplying the SAD computation workflow (sad.ipynb tutorial) used to compute SNP Activity Difference profiles for DeepFun scores) — https://github.com/davek44/Basset
- **DeepSEA** (External deep learning predictor whose SAD scores for the same variants and assays form the other side of the per-assay Pearson correlation)

## Examples

```
from scipy.stats import pearsonr; pcc, p = pearsonr(deepfun_avg_sad[assay], deepsea_avg_sad[assay])
```

## Evaluation signals

- Computed CTCF PCC ≈ 0.883 with P = 2.2 × 10−16
- Computed DNase-seq PCC ≈ 0.751 with P = 1.46 × 10−10
- Computed H3K4me3 PCC ≈ 0.490 and H3K9ac PCC ≈ 0.451
- All four PCC values are positive and statistically significant where P values are reported (P < 0.05)
- Output file deepfun_deepsea_sad_pcc.csv contains exactly four per-assay PCC values, one per assay group (CTCF, DNase-seq, H3K4me3, H3K9ac)

## Limitations

- DeepFun model has not considered long-range interactions, such as 3D DNA contact
- Quantitative epigenomic signal information is ignored because peaks are binarized during training
- No normalization approach has yet been established to eliminate technical bias for integrating quantitative signals
- Performance of CNN-based methods may strongly depend on initial convolution filters for captured motif information
- Correlations are moderate (PCC ≈ 0.45–0.49) for histone marks H3K4me3 and H3K9ac, so per-assay agreement is assay-dependent

## Evidence

- [results] DeepFun's averaged SAD scores correlate significantly and positively with DeepSEA's: [evidence span withheld — anchor status: not_found]
- [methods] Following the Basset SAD tutorial, run the Torch7/Python Basset-framework model: [evidence span withheld — anchor status: not_found]
- [readme] More details can follow https://github.com/davek44/Basset/blob/master/tutorials/sad.ipynb.: "The most useful capability of DeepFun right now is to annotate noncoding genomic variants with their influence on functional properties like accessibility. More details can follow"
- [readme] DeepFun is an updated model based on Basset Framework: "DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python."
- [discussion] DeepFun model has not considered long-range interactions, such as 3D DNA contact: "DeepFun model has not considered long-range interactions, such as 3D DNA contact (20,37)"
