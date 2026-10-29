---
name: cnn-variant-effect-prediction
description: Use when you need to score the reference vs. alternative allele of a
  non-coding GWAS variant against chromatin profiles (DNase, histone marks, TF binding)
  in a tissue-specific manner, or to identify the motif driving a predicted effect
  (e.g., NFE2 at rs6426833).
when_to_use_negative:
- The variant effect depends on long-range interactions such as 3D DNA contact — DeepFun's
  model has not considered long-range interactions
- Quantitative epigenomic signal magnitudes matter, since peaks were binarized during
  training and quantitative information is ignored
- The variant is coding or its functional class is unknown outside DNA accessibility,
  histone marks, and TF binding profiles covered by the two pre-trained models
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: DeepFun
  role: Provides the pre-trained CNN models (DeepFun_best_A.th.gz / DeepFun_best_B.th.gz)
    and web server for scoring SNP effects on 7879 epigenomic profiles
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Framework providing basset_sat_vcf.py, the sat-mut and SAD tutorials, and
    dependency installation for running saturated mutagenesis locally
  repo: https://github.com/davek44/Basset
- name: Torch7
  role: Runtime for executing the pre-trained Basset/DeepFun Torch model (basset_sat_vcf.py
    prediction step)
- name: Python
  role: Runs basset_sat_vcf.py and downstream extraction/plotting of the NFE2 profile
    heat maps
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:112ffe6f163de265f584cf2358e5a27381e776eb1a862a932932179f77bbcea4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  - Python
  - Torch7
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# cnn-variant-effect-prediction

## Summary

Predict the functional impact of non-coding SNPs at single-base resolution with DeepFun, a Basset-based deep convolutional neural network trained on 7879 ENCODE/Roadmap epigenomic profiles (1548 DNase I, 1536 histone mark, 4795 TF binding across 225 tissues/cell types), and dissect the underlying motif with in silico saturated mutagenesis.

## When to use

Use when you need to score the reference vs. alternative allele of a non-coding GWAS variant against chromatin profiles (DNase, histone marks, TF binding) in a tissue-specific manner, or to identify the motif driving a predicted effect (e.g., NFE2 at rs6426833).

## When NOT to use

- The variant effect depends on long-range interactions such as 3D DNA contact — DeepFun's model has not considered long-range interactions
- Quantitative epigenomic signal magnitudes matter, since peaks were binarized during training and quantitative information is ignored
- The variant is coding or its functional class is unknown outside DNA accessibility, histone marks, and TF binding profiles covered by the two pre-trained models

## Inputs

- Genomic sequence centered on the query SNP (e.g., +/-200 bp around rs6426833) formatted for Basset's sat-mut workflow
- VCF or variant record with reference and alternative alleles (e.g., rs6426833 A>G)
- Pre-trained model file: DeepFun_best_B.th.gz (TF binding) or DeepFun_best_A.th.gz (accessibility, histone marks, CTCF)

## Outputs

- Per-allele predicted activity values for each of the 7879 epigenomic profiles (extract the NFE2 TF binding profile as needed)
- SAD heat maps of in silico saturated mutagenesis over the +/-200 bp region (e.g., sat_mut_rs6426833_NFE2.png)
- Motif interpretation for the predicted effect (e.g., NFE2 motif TGACTCAGCA)

## How to apply

Choose the pre-trained model matching the profile class: DeepFun_best_A.th.gz for DNA accessibility/histone marks/CTCF or DeepFun_best_B.th.gz for transcription factor binding profiles (NFE2 is covered by model B). Install Torch7 and Python dependencies per the Basset tutorials, set BASSETDIR, and format a genomic sequence centered on the query variant for Basset's sat-mut workflow (sat_mut.ipynb). Run basset_sat_vcf.py with the chosen model to compute predicted activity for every single-nucleotide substitution, then extract the relevant chromatin profile from the multi-task output. Generate SAD heat maps for reference and alternative alleles over the +/-200 bp window and read the motif signature from the substitution pattern — for rs6426833/NFE2, only the A-to-G mutation increases binding while substitutions within the TGACTCAGCA motif decrease it.

## Related tools

- **DeepFun** (Provides the pre-trained CNN models (DeepFun_best_A.th.gz / DeepFun_best_B.th.gz) and web server for scoring SNP effects on 7879 epigenomic profiles) — https://github.com/bsml320/DeepFun
- **Basset** (Framework providing basset_sat_vcf.py, the sat-mut and SAD tutorials, and dependency installation for running saturated mutagenesis locally) — https://github.com/davek44/Basset
- **Torch7** (Runtime for executing the pre-trained Basset/DeepFun Torch model (basset_sat_vcf.py prediction step))
- **Python** (Runs basset_sat_vcf.py and downstream extraction/plotting of the NFE2 profile heat maps)

## Examples

```
python basset_sat_vcf.py -m DeepFun_best_B.th.gz -s seqs.h5 -v rs6426833.vcf -o sat_out
```

## Evaluation signals

- For rs6426833 on the NFE2 binding profile, only the A-to-G mutation increases predicted binding activity; other nearby single-nucleotide substitutions do not impact SAD
- Substitutions within the NFE2 motif TGACTCAGCA decrease predicted binding activity, consistent with the motif's position in the heat map
- The correct model file is used: DeepFun_best_B.th.gz for TF binding profiles vs. DeepFun_best_A.th.gz for accessibility/histone/CTCF profiles
- Heat maps cover the +/-200 bp window around the variant for both reference and alternative alleles, matching Figure 4B
- Predictions are single-base resolution allele-specific values per epigenomic profile, not aggregate scores

## Limitations

- DeepFun model has not considered long-range interactions, such as 3D DNA contact
- Training treated peaks as binary format, so quantitative epigenomic signal information is ignored
- No normalization approach is yet established to eliminate technical bias for integrating quantitative signals
- CNN-based method performance may strongly depend on initial convolution filters for captured motif information
- Model does not take specific variant information into consideration, predicting effects even for variants never observed

## Evidence

- [readme] DeepFun is a deep learning based approach to predict the functional impacts of sequence alterations at single nucleotide resolution.: "DeepFun is a deep learning based approach to predict the functional impacts of sequence alterations at single nucleotide resolution."
- [methods] or dnacnn_best_B.zip (Transcription factors binding sites).: "or dnacnn_best_B.zip (Transcription factors binding sites)."
- [readme] Saturated mutagenesis is a powerful tool both for dissecting a specific sequence of interest and understanding what the model learned.: "Saturated mutagenesis is a powerful tool both for dissecting a specific sequence of interest and understanding what the model learned."
- [discussion] with the help of in silico saturated mutagenesis analysis, the DeepFun web server will be useful for interpreting potential target gene's motifs around functional variants: "with the help of in silico saturated mutagenesis analysis, the DeepFun web server will be useful for interpreting potential target gene's motifs around functional variants"
- [methods] More details can follow https://github.com/davek44/Basset/blob/master/tutorials/sat_mut.ipynb.: "More details can follow https://github.com/davek44/Basset/blob/master/tutorials/sat_mut.ipynb."
