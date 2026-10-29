---
name: snp-activity-difference-computation
description: Use when annotating query non-coding SNPs with per-assay functional impact
  scores across DNase I accessibility, histone mark, and transcription factor binding
  profiles, especially for tissue-specific GWAS variant prioritization.
when_to_use_negative:
- When the goal is to dissect learned motifs around a specific sequence, use the in
  silico saturated mutagenesis tutorial (sat_mut.ipynb) instead of SAD scoring
- When variants need long-range or 3D DNA contact context — the DeepFun model has
  not considered long-range interactions, such as 3D DNA contact
- When quantitative epigenomic signal levels are required — peaks were binarized during
  training, so quantitative information is ignored
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0203
tools:
- name: DeepFun
  role: Provides the pre-trained deep CNN models (DeepFun_best_A.th.gz, DeepFun_best_B.th.gz)
    and web server for predicting allele-specific activity across 7879 epigenomic
    profiles
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Underlying Torch7/Python framework supplying basset_sad.py and the sad.ipynb
    tutorial used to compute SNP Activity Difference profiles
  repo: https://github.com/davek44/Basset
- name: DeepSEA
  role: External baseline whose averaged per-assay SAD scores are correlated with
    DeepFun's (Pearson) for cross-method consistency validation
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:95baf79c4d3df25e306073df54f7a42ccbe7110c91b44d7a8cc264598a64fbe4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  - DeepSEA
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# snp-activity-difference-computation

## Summary

Compute SNP Activity Difference (SAD) profiles with the Basset/DeepFun framework: run a pre-trained deep convolutional neural network over reference and alternate allele sequences for each non-coding variant and score functional impact per chromatin profile via SAD = P(alt) − P(ref) plus a log-odds difference.

## When to use

Use when annotating query non-coding SNPs with per-assay functional impact scores across DNase I accessibility, histone mark, and transcription factor binding profiles, especially for tissue-specific GWAS variant prioritization.

## When NOT to use

- When the goal is to dissect learned motifs around a specific sequence, use the in silico saturated mutagenesis tutorial (sat_mut.ipynb) instead of SAD scoring
- When variants need long-range or 3D DNA contact context — the DeepFun model has not considered long-range interactions, such as 3D DNA contact
- When quantitative epigenomic signal levels are required — peaks were binarized during training, so quantitative information is ignored

## Inputs

- Query SNP input file listing noncoding variants (reference and alternate allele sequences), formatted per the Basset SAD tutorial sad.ipynb
- Pre-trained model file DeepFun_best_A.th.gz (DNA accessibility, histone marks, CTCF binding) and/or DeepFun_best_B.th.gz (transcription factor binding sites)
- Torch7 and Python execution environment with Basset framework dependencies (pysam, bedtools)

## Outputs

- sad_scores.tsv with one row per query variant and per-profile SAD = P(alt) − P(ref) and log-odds difference values across all chromatin profiles
- Tissue-aggregated SAD table (e.g., sad_tissue_table.tsv) for disease-relevant vs unrelated tissue comparison, suitable for a tissue-by-disease heatmap
- Per-assay averaged SAD values for correlation analysis (e.g., deepfun_deepsea_sad_pcc.csv with Pearson correlation coefficients)

## How to apply

Install Torch7 and Python dependencies following the Basset tutorials, then download the DeepFun pre-trained models DeepFun_best_A.th.gz (DNA accessibility sites, histone marks and CTCF binding sites) and/or DeepFun_best_B.th.gz (transcription factor binding sites) from http://bioinfo.uth.edu/deepfun/models/DeepFun_model/. Format the query SNP file according to the Basset SAD tutorial (sad.ipynb), then run basset_sad.py to predict activity probabilities for both alleles across all chromatin features and compute SAD = P(alt) − P(ref) and a log-odds difference for every profile. Model A's feature space spans 1548 DNase I accessibility, 1536 histone mark, and 4795 transcription factor binding profiles covering 225 tissues or cell types, so the output sad_scores.tsv must contain one row per variant with per-profile SAD values. To validate tissue specificity, aggregate variant-level SAD scores by tissue/cell type (e.g., maximum or mean per tissue), compare disease-relevant versus unrelated tissues, and check consistency against DeepSEA by computing per-assay averaged scores and Pearson correlations (e.g., CTCF PCC = 0.883, DNase-seq PCC = 0.751, H3K4me3 PCC = 0.490, H3K9ac PCC = 0.451).

## Related tools

- **DeepFun** (Provides the pre-trained deep CNN models (DeepFun_best_A.th.gz, DeepFun_best_B.th.gz) and web server for predicting allele-specific activity across 7879 epigenomic profiles) — https://github.com/bsml320/DeepFun
- **Basset** (Underlying Torch7/Python framework supplying basset_sad.py and the sad.ipynb tutorial used to compute SNP Activity Difference profiles) — https://github.com/davek44/Basset
- **DeepSEA** (External baseline whose averaged per-assay SAD scores are correlated with DeepFun's (Pearson) for cross-method consistency validation)

## Examples

```
python basset_sad.py -t DeepFun_best_A.th -s sequences.h5 -o sad_scores.tsv
```

## Evaluation signals

- Output table exists, contains one row per query variant, and includes SAD/log-odds values for every chromatin profile in the model's feature space (1548 DNase I, 1536 histone mark, 4795 TF binding for Model A)
- Tissue-specificity check reproduces Figure 4A pattern: immune-related trait loci show higher SAD in intestines and colon; neurodegenerative/neuropsychiatric loci show higher SAD in brain and spinal cord
- Cross-method consistency: Pearson correlation of DeepFun vs DeepSEA per-assay averaged SAD scores ≈ 0.883 for CTCF, ≈ 0.751 for DNase-seq, ≈ 0.490 for H3K4me3, ≈ 0.451 for H3K9ac
- SAD values are defined as P(alt) − P(ref); verify the sign convention so positive scores indicate increased predicted activity for the alternate allele
- All reported profile counts sum to 7879 epigenomic profiles covering 225 tissues or cell types, as stated in the DeepFun feature space

## Limitations

- DeepFun model has not considered long-range interactions, such as 3D DNA contact
- It has ignored the quantitative epigenomic signal information because 'peaks' were treated as binary format during training; no normalization approach yet established to eliminate technical bias for integrating quantitative signals
- The performance of most CNN-based methods may strongly depend on initial convolution filters for captured motif information
- Torch7 is a legacy dependency; local analysis requires installing Basset-framework dependencies and bedtools/pysam

## Evidence

- [methods] SAD definition and computation: [evidence span withheld — anchor status: not_found]
- [abstract] Model feature space: "including 1548 DNase I accessibility, 1536 histone mark, and 4795 transcription factor binding profiles covering 225 tissues or cell types."
- [readme] Local analysis setup: "DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python. If you want to conduct local analysis for large mounts of data, please follow"
- [readme] SAD tutorial pointer: "The most useful capability of DeepFun right now is to annotate noncoding genomic variants with their influence on functional properties like accessibility. More details can follow"
- [discussion] Long-range limitation: "DeepFun model has not considered long-range interactions, such as 3D DNA contact (20,37)"
- [results] Cross-method correlation validation: [evidence span withheld — anchor status: not_found]
