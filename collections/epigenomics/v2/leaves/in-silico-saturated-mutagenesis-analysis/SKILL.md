---
name: in-silico-saturated-mutagenesis-analysis
description: Use when you need to dissect which bases and sequence motifs around a
  non-coding SNP drive a predicted chromatin activity change (e.g., transcription
  factor binding) for a single variant, at single-base resolution.
when_to_use_negative:
- The goal is to annotate many variants genome-scale rather than dissect one sequence/variant
  — use DeepFun's SNP Activity Difference (SAD) profiles via tutorials/sad.ipynb instead
- The variant's effect depends on long-range interactions such as 3D DNA contact,
  which the DeepFun model does not consider
- You need quantitative (non-binarized) epigenomic signal predictions — DeepFun trained
  with peaks treated as binary format and ignores quantitative information
edam_operation: http://edamontology.org/operation_0440
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3474
tools:
- name: DeepFun
  role: Deep learning model (Basset-based CNN) whose multi-task predictions over 7879
    epigenomic profiles supply the activity scores used in saturated mutagenesis;
    pre-trained model DeepFun_best_B.th.gz is used for transcription factor binding
    profiles
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Framework providing basset_sat_vcf.py and the sat_mut tutorial for executing
    in silico saturated mutagenesis and generating heat maps
  repo: https://github.com/davek44/Basset
- name: Torch7
  role: Runtime dependency for executing the Basset/DeepFun Torch model locally
- name: Python
  role: Runtime dependency for Basset's preprocessing and visualization scripts (basset_sat_vcf.py)
    and dependency installation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:24c50a7a804b96516aa8a7863743d99bfed6e1cda735a55fe5d921177f224832
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  - Python
  - Torch7
  edam_operation: http://edamontology.org/operation_0440
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# in-silico-saturated-mutagenesis-analysis

## Summary

Runs DeepFun/Basset in silico saturated mutagenesis to predict the effect of every single-nucleotide substitution within a sequence window (e.g., ±200 bp around a query SNP) on a specific chromatin profile, and renders SAD-style heat maps that reveal informative motifs (e.g., the NFE2 motif TGACTCAGCA at rs6426833).

## When to use

Use when you need to dissect which bases and sequence motifs around a non-coding SNP drive a predicted chromatin activity change (e.g., transcription factor binding) for a single variant, at single-base resolution.

## When NOT to use

- The goal is to annotate many variants genome-scale rather than dissect one sequence/variant — use DeepFun's SNP Activity Difference (SAD) profiles via tutorials/sad.ipynb instead
- The variant's effect depends on long-range interactions such as 3D DNA contact, which the DeepFun model does not consider
- You need quantitative (non-binarized) epigenomic signal predictions — DeepFun trained with peaks treated as binary format and ignores quantitative information

## Inputs

- Pre-trained DeepFun model file (DeepFun_best_A.th.gz or DeepFun_best_B.th.gz; model B covers transcription factor binding profiles such as NFE2)
- Genomic sequence centered on the query variant (e.g., sequence around rs6426833)
- Variant allele information (reference and alternative alleles) formatted for Basset's sat-mut workflow

## Outputs

- Saturated mutagenesis heat map of predicted activity for every single-nucleotide substitution across the sequence window (e.g., sat_mut_rs6426833_NFE2.png)
- Per-substitution predicted activity values for the profile of interest (e.g., NFE2 transcription factor binding profile task)
- Identified informative sequence motif (e.g., TGACTCAGCA for NFE2)

## How to apply

Download the pre-trained DeepFun model matching the profile of interest — DeepFun_best_B.th.gz for transcription factor binding profiles such as NFE2, DeepFun_best_A.th.gz for DNA accessibility, histone marks and CTCF. Install Torch7 and Python dependencies following the Basset tutorials, and set BASSETDIR/PATH/PYTHONPATH/LUA_PATH per the Basset README. Retrieve the genomic sequence centered on the query SNP (rs6426833 for the NFE2 example) and run the saturated mutagenesis script basset_sat_vcf.py, following https://github.com/davek44/Basset/blob/master/tutorials/sat_mut.ipynb, to compute predicted activity for every single-nucleotide substitution across the window. Extract the task corresponding to the chromatin profile of interest from the multi-task output and generate heat maps for reference and alternative alleles over the ±200 bp region. Judge success by comparing to the paper's expected pattern: for rs6426833 on the NFE2 profile, only the A-to-G mutation increases predicted binding activity, while substitutions within the TGACTCAGCA motif decrease it (Figure 4B).

## Related tools

- **DeepFun** (Deep learning model (Basset-based CNN) whose multi-task predictions over 7879 epigenomic profiles supply the activity scores used in saturated mutagenesis; pre-trained model DeepFun_best_B.th.gz is used for transcription factor binding profiles) — https://github.com/bsml320/DeepFun
- **Basset** (Framework providing basset_sat_vcf.py and the sat_mut tutorial for executing in silico saturated mutagenesis and generating heat maps) — https://github.com/davek44/Basset
- **Torch7** (Runtime dependency for executing the Basset/DeepFun Torch model locally)
- **Python** (Runtime dependency for Basset's preprocessing and visualization scripts (basset_sat_vcf.py) and dependency installation)

## Examples

```
python basset_sat_vcf.py -t DeepFun_best_B.th -o sat_mut_rs6426833_NFE2 -s rs6426833_seq.bed -v rs6426833.vcf
```

## Evaluation signals

- For rs6426833 on the NFE2 binding profile, only the A-to-G substitution increases predicted binding activity; all other nearby mutations do not impact SAD
- Substitutions within the predicted NFE2 motif TGACTCAGCA decrease predicted binding activity, matching Figure 4B
- The heat map spans the ±200 bp region surrounding the query SNP for both reference and alternative alleles
- The correct pre-trained model was used (model B for TF binding profiles such as NFE2; model A for DNase I accessibility, histone marks and CTCF)
- The task for the specific chromatin profile of interest (e.g., NFE2) was correctly extracted from the multi-task output before plotting

## Limitations

- DeepFun model has not considered long-range interactions, such as 3D DNA contact
- Training treated peaks as binary format, so quantitative epigenomic signal information is ignored; no normalization approach is yet established to integrate quantitative signals
- Performance of CNN-based methods may strongly depend on initial convolution filters for captured motif information
- No software version is stated for DeepFun; planned additions such as training with ENCODE 3 maps are not yet implemented

## Evidence

- [readme] Saturated mutagenesis is a powerful tool both for dissecting a specific sequence of interest and understanding what the model learned.: "Saturated mutagenesis is a powerful tool both for dissecting a specific sequence of interest and understanding what the model learned."
- [readme] DeepFun is an updated model based on Basset Framework, based on Torch7 and Python.: "DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python."
- [methods] Execute an in silico saturated mutagenesis: "Execute an in silico saturated mutagenesis"
- [methods] Install dependencies packages following Basset tutorials.: "please follow Basset tutorials install dependencies packages at first."
- [discussion] with the help of in silico saturated mutagenesis analysis, the DeepFun web server will be useful for interpreting potential target gene's motifs around functional variants: "with the help of in silico saturated mutagenesis analysis, the DeepFun web server will be useful for interpreting potential target gene's motifs around functional variants"
- [discussion] during the training process, we treated the 'peaks' as the binary format: [evidence span withheld — anchor status: not_found]
