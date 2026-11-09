---
name: cross-tool-benchmark-comparison
description: Use when two sequence-based variant-effect tools (e.g. DeepFun/Basset
  and DeepSEA) have scored the same SNP set across matched assays (CTCF, DNase-seq,
  H3K4me3, H3K9ac) and you need a quantitative agreement metric.
when_to_use_negative:
- You need track-level (not assay-level) agreement — the reference workflow averages
  SAD scores across all tracks belonging to each assay before correlating
- The two tools were run on different variant sets or different assay tracks — correlation
  would be confounded by input mismatch rather than tool disagreement
- You need causal-effect or phenotypic prioritization (DeepFun's web-server GWAS use
  case) rather than inter-tool consistency benchmarking
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: DeepFun
  role: Pre-trained CNN model (e.g. DeepFun_best_A.th.gz) used to compute SAD scores
    between reference and alternate alleles for the query variants
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Framework providing the Torch7/Python SAD pipeline (sad.ipynb tutorial) used
    to run DeepFun's model and compute SNP Activity Difference profiles
  repo: https://github.com/davek44/Basset
- name: DeepSEA
  role: Second variant-effect predictor whose SAD scores for the same variants and
    assays are compared against DeepFun's via Pearson correlation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:af614bbff3f6355186ddb2ec4377d747552ec9f788158b5e6e1646f89e1e4727
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  - DeepSEA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cross-tool-benchmark-comparison

## Summary

Benchmark agreement between two deep learning variant-effect predictors by computing per-assay averaged SNP Activity Difference (SAD) scores and Pearson correlation coefficients, as done to compare DeepFun and DeepSEA on the same variant set.

## When to use

Use when two sequence-based variant-effect tools (e.g. DeepFun/Basset and DeepSEA) have scored the same SNP set across matched assays (CTCF, DNase-seq, H3K4me3, H3K9ac) and you need a quantitative agreement metric.

## When NOT to use

- You need track-level (not assay-level) agreement — the reference workflow averages SAD scores across all tracks belonging to each assay before correlating
- The two tools were run on different variant sets or different assay tracks — correlation would be confounded by input mismatch rather than tool disagreement
- You need causal-effect or phenotypic prioritization (DeepFun's web-server GWAS use case) rather than inter-tool consistency benchmarking

## Inputs

- VCF/BED file of query SNPs scored identically by both tools
- DeepFun SAD scores from the Basset-framework model (Torch7/Python, e.g. DeepFun_best_A.th.gz)
- DeepSEA SAD scores for the same variants and assays
- Track-to-assay mapping (which output tracks belong to CTCF, DNase-seq, H3K4me3, H3K9ac)

## Outputs

- Per-assay averaged SAD score vectors for each tool
- Pearson correlation coefficients with P values per assay (deepfun_deepsea_sad_pcc.csv)

## How to apply

First ensure both tools score the identical variant set for the same assays: run DeepFun's pre-trained model (e.g. DeepFun_best_A.th.gz, covering DNA accessibility, histone marks and CTCF binding) through the Basset SAD tutorial (sad.ipynb) on Torch7/Python to compute SNP Activity Difference profiles between reference and alternate alleles. Obtain DeepSEA SAD scores for the same variants and assays. Average the SAD scores across all output tracks belonging to each assay (CTCF, DNase-seq, H3K4me3, H3K9ac) separately per tool, so the comparison is at assay level rather than track level. Compute the Pearson correlation coefficient between the two tools' averaged per-assay scores using numpy/scipy (scipy.stats.pearsonr). Write the four PCC values to a CSV (e.g. deepfun_deepsea_sad_pcc.csv) and report the P values. Validity is judged by comparing the computed PCCs to reference values from the paper: CTCF 0.883, DNase-seq 0.751, H3K4me3 0.490, H3K9ac 0.451.

## Related tools

- **DeepFun** (Pre-trained CNN model (e.g. DeepFun_best_A.th.gz) used to compute SAD scores between reference and alternate alleles for the query variants) — https://github.com/bsml320/DeepFun
- **Basset** (Framework providing the Torch7/Python SAD pipeline (sad.ipynb tutorial) used to run DeepFun's model and compute SNP Activity Difference profiles) — https://github.com/davek44/Basset
- **DeepSEA** (Second variant-effect predictor whose SAD scores for the same variants and assays are compared against DeepFun's via Pearson correlation)

## Examples

```
from scipy.stats import pearsonr; pcc, p = pearsonr(deepfun_avg_sad, deepsea_avg_sad)
```

## Evaluation signals

- Computed CTCF PCC ≈ 0.883 with P = 2.2 × 10−16
- Computed DNase-seq PCC ≈ 0.751 with P = 1.46 × 10−10
- Computed H3K4me3 PCC ≈ 0.490 and H3K9ac PCC ≈ 0.451 (moderate agreement, indicating assay-type sensitivity of the benchmark)
- All reported correlations are positive and statistically significant (P ≤ 2.2 × 10−16 for the strongest)
- Output CSV (deepfun_deepsea_sad_pcc.csv) contains exactly four per-assay PCC rows matching the assay list

## Limitations

- Agreement is only moderate for histone marks (H3K4me3 PCC = 0.490, H3K9ac PCC = 0.451) compared with CTCF/DNase-seq, so high correlation should not be assumed across all assay types
- DeepFun models binarize peaks, ignoring quantitative epigenomic signal information, which may limit comparability with tools using quantitative signals
- DeepFun does not consider long-range interactions such as 3D DNA contact
- DeepFun pre-trained models are split into A (DNA accessibility, histone marks, CTCF) and B (transcription factor binding), so the assay must match the model downloaded

## Evidence

- [results] CTCF PCC = 0.883 (P = 2.2 × 10−16), DNase-seq PCC = 0.751 (P = 1.46 × 10−10), and moderately for H3K4me3 (PCC = 0.490) and H3K9ac (PCC = 0.451): [evidence span withheld — anchor status: not_found]
- [methods] Following the Basset SAD tutorial (sad.ipynb): "More details can follow https://github.com/davek44/Basset/blob/master/tutorials/sad.ipynb."
- [readme] DeepFun is an updated model based on Basset Framework based on Torch7 and Python: "DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python."
- [readme] download the pre-trained model from github here: dnacnn_best_A.zip (DNA accessibility sites, histone marks and Transcription factor CTCF binding sites): "you'll need to download the pre-trained model from github here: dnacnn_best_A.zip (DNA accessibility sites, histone marks and Transcription factor CTCF binding sites)"
- [methods] Compute SNP Activity Difference profiles: "The most useful capability of DeepFun right now is to annotate noncoding genomic variants with their influence on functional properties like accessibility."
