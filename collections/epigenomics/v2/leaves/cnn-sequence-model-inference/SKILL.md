---
name: cnn-sequence-model-inference
description: Use when you need single-base-resolution functional impact scores (SAD
  = P(alt) − P(ref)) for non-coding SNPs across chromatin accessibility, histone mark,
  and TF binding profiles in a tissue-specific manner.
when_to_use_negative:
- Variants whose effects act through long-range (3D DNA contact) mechanisms — the
  DeepFun model has not considered long-range interactions
- Analyses requiring quantitative epigenomic signal intensities — peaks are binarized
  during training, so quantitative information is ignored
- Coding variant interpretation — DeepFun targets non-coding genetic variants
edam_operation: http://edamontology.org/operation_0446
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3517
tools:
- name: DeepFun
  role: Pre-trained dense epigenomic CNN used to predict per-allele activity probabilities
    and derive SAD scores for query non-coding SNPs
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Framework on which DeepFun is built; provides basset_sad.py and the sad.ipynb
    / sat_mut.ipynb tutorials used to compute SNP Activity Difference profiles and
    run in silico saturated mutagenesis
  repo: https://github.com/davek44/Basset
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:499037772c4573b754bff1b05c5b096bf66aa79f396ae59da33d9484cd886e9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  edam_operation: http://edamontology.org/operation_0446
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cnn-sequence-model-inference

## Summary

Score query non-coding variants with a pre-trained deep convolutional neural network (DeepFun, built on the Basset Torch7 framework) that predicts per-allele activity across 7879 epigenomic profiles (1548 DNase I accessibility, 1536 histone mark, 4795 TF binding profiles spanning 225 tissues/cell types), enabling tissue-specific functional prioritization of GWAS variants.

## When to use

Use when you need single-base-resolution functional impact scores (SAD = P(alt) − P(ref)) for non-coding SNPs across chromatin accessibility, histone mark, and TF binding profiles in a tissue-specific manner.

## When NOT to use

- Variants whose effects act through long-range (3D DNA contact) mechanisms — the DeepFun model has not considered long-range interactions
- Analyses requiring quantitative epigenomic signal intensities — peaks are binarized during training, so quantitative information is ignored
- Coding variant interpretation — DeepFun targets non-coding genetic variants

## Inputs

- Query SNP list of non-coding variants (reference and alternate alleles), formatted per the Basset SAD tutorial (sad.ipynb)
- Pre-trained model file DeepFun_best_A.th.gz (DNA accessibility, histone marks, CTCF) and/or DeepFun_best_B.th.gz (TF binding sites)
- Torch7 and Python environment with Basset dependencies (pysam, bedtools)

## Outputs

- Per-variant, per-profile SAD scores (P(alt) − P(ref)) across 1548 DNase I, 1536 histone mark, and 4795 TF binding profiles (Model A feature space)
- Log-odds difference values for each variant/profile pair
- Tissue-aggregated SAD table (e.g. sad_tissue_table.tsv) for tissue-by-disease comparison
- In silico saturated mutagenesis output for motif dissection around variants

## How to apply

Set up the Torch7/Python environment following the Basset tutorials, then download the pre-trained models DeepFun_best_A.th.gz (DNA accessibility, histone marks, CTCF) and/or DeepFun_best_B.th.gz (transcription factor binding) from the DeepFun model repository. Encode reference and alternate allele sequences for each query SNP following the Basset SAD tutorial input format, and run basset_sad.py with each pre-trained model to compute SAD = P(alt) − P(ref) plus a log-odds difference across all chromatin profiles. The model uses a deep CNN trained on ENCODE and Roadmap peaks treated as binary labels, so it can predict accessibility effects even for variants never observed previously. Aggregate per-variant, per-feature scores by tissue/cell type (e.g. max or mean SAD per tissue) when the goal is tissue-level prioritization, and optionally run in silico saturated mutagenesis (basset_sat.py / sat_mut.ipynb) to dissect motifs around a variant.

## Related tools

- **DeepFun** (Pre-trained dense epigenomic CNN used to predict per-allele activity probabilities and derive SAD scores for query non-coding SNPs) — https://github.com/bsml320/DeepFun
- **Basset** (Framework on which DeepFun is built; provides basset_sad.py and the sad.ipynb / sat_mut.ipynb tutorials used to compute SNP Activity Difference profiles and run in silico saturated mutagenesis) — https://github.com/davek44/Basset

## Examples

```
basset_sad.py DeepFun_best_A.th query_snps.vcf out_prefix
```

## Evaluation signals

- Output table contains one row per query variant with SAD/log-odds values for every chromatin profile in the model's feature space (7879 profiles for Model A)
- SAD values equal P(alt) − P(ref) for both alleles predicted by the model at single-base resolution
- Tissue aggregation reproduces expected trait–tissue concordance, e.g. immune-related trait loci show higher SAD in intestines/colon and neurodegenerative/neuropsychiatric loci show higher SAD in brain/spinal cord
- Model files load correctly under Torch7 (DeepFun_best_A.th.gz / DeepFun_best_B.th.gz) and profile count matches the 1548+1536+4795 feature space

## Limitations

- DeepFun model has not considered long-range interactions, such as 3D DNA contact
- Peaks were treated as binary format during training, so quantitative epigenomic signal information is ignored
- No normalization approach has yet been established to eliminate technical bias for integrating quantitative signals
- CNN-based method performance may strongly depend on initial convolution filters for captured motif information

## Evidence

- [readme] DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python.: "DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python."
- [abstract] feature space of 1548 DNase I, 1536 histone mark, 4795 TF binding profiles: "including 1548 DNase I accessibility, 1536 histone mark, and 4795 transcription factor binding profiles covering 225 tissues or cell types."
- [methods] SAD computation via Basset SAD tutorial: "Compute SNP Activity Difference profiles."
- [intro] model does not take specific variant information into consideration, enabling prediction for unobserved variants: "This feature in DeepFun enables the prediction of accessibility effects even for those that have never been observed previously"
- [discussion] peaks treated as binary during training: [evidence span withheld — anchor status: not_found]
