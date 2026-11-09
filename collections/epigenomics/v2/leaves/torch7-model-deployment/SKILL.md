---
name: torch7-model-deployment
description: Use when annotating a batch of non-coding SNPs with per-profile functional
  impact scores using DeepFun pre-trained models locally (e.g. large variant sets
  beyond the web server), following the Basset SAD tutorial.
when_to_use_negative:
- Variants are few enough to annotate interactively via the DeepFun web server (https://bioinfo.uth.edu/deepfun/)
  rather than local deployment
- The effect of interest depends on long-range interactions such as 3D DNA contact,
  which the DeepFun model does not consider
- Quantitative (non-binarized) epigenomic signal strength is required, since DeepFun
  treats peaks as binary during training
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3474
tools:
- name: DeepFun
  role: Provides the pre-trained Torch7 CNN models (DeepFun_best_A.th.gz, DeepFun_best_B.th.gz)
    deployed locally to score SNP alleles across epigenomic profiles
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Underlying Torch7/Python framework supplying basset_sad.py, the SAD tutorial
    (sad.ipynb), and dependency installation scripts used to run the DeepFun models
  repo: https://github.com/davek44/Basset
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:30d865bd7f89caa538a878c0cea156ae6c1cd611a4d17e5b32d84cf692ac4e3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# torch7-model-deployment

## Summary

Deploy DeepFun's pre-trained Torch7 models (DeepFun_best_A.th.gz / DeepFun_best_B.th.gz) locally on the Basset framework to score non-coding SNPs, computing SAD = P(alt) − P(ref) and log-odds differences across 7879 epigenomic profiles (1548 DNase I, 1536 histone mark, 4795 TF binding) covering 225 tissues/cell types.

## When to use

Use when annotating a batch of non-coding SNPs with per-profile functional impact scores using DeepFun pre-trained models locally (e.g. large variant sets beyond the web server), following the Basset SAD tutorial.

## When NOT to use

- Variants are few enough to annotate interactively via the DeepFun web server (https://bioinfo.uth.edu/deepfun/) rather than local deployment
- The effect of interest depends on long-range interactions such as 3D DNA contact, which the DeepFun model does not consider
- Quantitative (non-binarized) epigenomic signal strength is required, since DeepFun treats peaks as binary during training

## Inputs

- Query SNP input file listing noncoding variants to annotate (Basset SAD tutorial format)
- Pre-trained model file DeepFun_best_A.th.gz (DNA accessibility, histone marks, CTCF binding)
- Pre-trained model file DeepFun_best_B.th.gz (transcription factor binding sites)
- Basset framework installation with Torch7 and Python dependencies (BASSETDIR configured)

## Outputs

- sad_scores.tsv with one row per query variant and SAD = P(alt) − P(ref) plus log-odds difference for every chromatin profile in the model's feature space

## How to apply

First install the dependency stack — Torch7 and Python (plus pysam and bedtools for Basset preprocessing) — following the Basset tutorials, and set BASSETDIR/PATH/LUA_PATH environment variables. Download the pre-trained DeepFun model files: DeepFun_best_A.th.gz (DNA accessibility, histone marks, and CTCF binding) and/or DeepFun_best_B.th.gz (transcription factor binding sites) from http://bioinfo.uth.edu/deepfun/models/DeepFun_model/. Prepare the query SNP input file per the Basset SAD tutorial (sad.ipynb) formatting, then run basset_sad.py with the chosen model(s) so each variant's reference and alternate alleles are scored across all chromatin profiles, yielding SAD = P(alt) − P(ref) and log-odds difference values. Decide which model to use based on the feature space needed: Model A for accessibility/histone/CTCF profiles, Model B for broader TF binding sites. Write per-variant, per-profile scores to sad_scores.tsv and validate its structure against the Basset SAD tutorial output.

## Related tools

- **DeepFun** (Provides the pre-trained Torch7 CNN models (DeepFun_best_A.th.gz, DeepFun_best_B.th.gz) deployed locally to score SNP alleles across epigenomic profiles) — https://github.com/bsml320/DeepFun
- **Basset** (Underlying Torch7/Python framework supplying basset_sad.py, the SAD tutorial (sad.ipynb), and dependency installation scripts used to run the DeepFun models) — https://github.com/davek44/Basset

## Examples

```
python basset_sad.py -t DeepFun_best_A.th.gz -o sad_out variants.bed
```

## Evaluation signals

- sad_scores.tsv exists and contains exactly one row per query variant
- Output table includes SAD and log-odds values for every chromatin profile in the model's feature space (Model A: 1548 DNase I + 1536 histone mark + 4795 TF binding profiles spanning 225 tissues/cell types)
- SAD values are consistent with SAD = P(alt) − P(ref) recomputed from the predicted allele probabilities
- Output structure matches the per-variant per-profile table shown in the Basset SAD tutorial (sad.ipynb)
- Model file choice matches intended feature space (Model A for accessibility/histone/CTCF, Model B for TF binding sites)

## Limitations

- DeepFun's model has not considered long-range interactions, such as 3D DNA contact
- Quantitative epigenomic signal information is ignored because peaks are binarized during training
- No normalization approach is yet established to eliminate technical bias for integrating quantitative epigenomic signals
- CNN-based method performance may strongly depend on initial convolution filters for capturing motif information
- Local deployment requires the legacy Torch7 framework and Basset dependency stack (pysam, bedtools)

## Evidence

- [methods] DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python.: "DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python."
- [abstract] including 1548 DNase I accessibility, 1536 histone mark, and 4795 transcription factor binding profiles covering 225 tissues or cell types.: "including 1548 DNase I accessibility, 1536 histone mark, and 4795 transcription factor binding profiles covering 225 tissues or cell types."
- [readme] you'll need to download the pre-trained model from github here: dnacnn_best_A.zip (DNA accessibility sites, histone marks and Transcription factor CTCF binding sites): "you'll need to download the pre-trained model from github here: dnacnn_best_A.zip (DNA accessibility sites, histone marks and Transcription factor CTCF binding sites)"
- [methods] More details can follow https://github.com/davek44/Basset/blob/master/tutorials/sad.ipynb.: "More details can follow https://github.com/davek44/Basset/blob/master/tutorials/sad.ipynb."
- [methods] please follow Basset tutorials install dependencies packages at first.: "please follow Basset tutorials install dependencies packages at first."
