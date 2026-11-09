---
name: chromatin-profile-score-interpretation
description: Use when a set of query noncoding SNPs must be functionally screened
  for their impact on chromatin accessibility, histone marks, or TF binding across
  many ENCODE/Roadmap epigenomic profiles in a tissue- or cell type-specific manner.
when_to_use_negative:
- The question concerns long-range effects such as 3D DNA contact — the DeepFun model
  has not considered long-range interactions
- Quantitative (continuous) epigenomic signal is required — DeepFun binarizes peaks
  during training and ignores the quantitative information
- The variant is already annotated in a feature table with functional scores, so no
  allele-level re-scoring is needed
edam_operation: http://edamontology.org/operation_0484
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: DeepFun
  role: Provides the pre-trained CNN models and downstream scripts used to score SNP
    activity differences across epigenomic profiles
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Underlying framework; supplies Torch7/Python dependencies, basset_sad.py,
    and the sad.ipynb tutorial for input formatting and execution
  repo: https://github.com/davek44/Basset
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:d7c24d92cb3fef098ca69feef1235ce54f0d1fca2586cd1e55fa919dea70a61b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  edam_operation: http://edamontology.org/operation_0484
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# chromatin-profile-score-interpretation

## Summary

Screens noncoding SNPs with DeepFun pre-trained CNN models (Basset framework, Torch7) to compute per-variant, per-profile SAD = P(alt) − P(ref) and log-odds differences across 7879 epigenomic profiles (1548 DNase I, 1536 histone mark, 4795 transcription factor binding; 225 tissues/cell types).

## When to use

Use when a set of query noncoding SNPs must be functionally screened for their impact on chromatin accessibility, histone marks, or TF binding across many ENCODE/Roadmap epigenomic profiles in a tissue- or cell type-specific manner.

## When NOT to use

- The question concerns long-range effects such as 3D DNA contact — the DeepFun model has not considered long-range interactions
- Quantitative (continuous) epigenomic signal is required — DeepFun binarizes peaks during training and ignores the quantitative information
- The variant is already annotated in a feature table with functional scores, so no allele-level re-scoring is needed

## Inputs

- Query SNP input file listing noncoding variants (Basset SAD tutorial sad.ipynb format)
- Pre-trained DeepFun model file DeepFun_best_A.th.gz (DNA accessibility, histone marks, CTCF binding)
- Pre-trained DeepFun model file DeepFun_best_B.th.gz (transcription factor binding sites)
- Torch7 and Python environment with Basset dependencies

## Outputs

- sad_scores.tsv with one row per query variant and SAD/log-odds values for every chromatin profile in the model's feature space

## How to apply

Install Torch7 and Python dependencies following the Basset tutorials, then download the pre-trained DeepFun model files DeepFun_best_A.th.gz (accessibility, histone marks, CTCF) and/or DeepFun_best_B.th.gz (TF binding) from bioinfo.uth.edu/deepfun/models/DeepFun_model/. Format the query SNP input file per the Basset SAD tutorial (sad.ipynb), then run basset_sad.py with the model to score reference and alternate alleles across all chromatin profiles, computing SAD = P(alt) − P(ref) and the log-odds difference for each profile. Collect the per-variant, per-feature scores (Model A: 1548 DNase I accessibility, 1536 histone mark, and 4795 TF binding profiles spanning 225 tissues/cell types) and write them to sad_scores.tsv. Large variant sets should be processed locally since the web server is intended for quick single-variant assessment.

## Related tools

- **DeepFun** (Provides the pre-trained CNN models and downstream scripts used to score SNP activity differences across epigenomic profiles) — https://github.com/bsml320/DeepFun
- **Basset** (Underlying framework; supplies Torch7/Python dependencies, basset_sad.py, and the sad.ipynb tutorial for input formatting and execution) — https://github.com/davek44/Basset

## Examples

```
basset_sad.py DeepFun_best_A.th query_snps.bed
```

## Evaluation signals

- sad_scores.tsv exists and contains exactly one row per query variant
- Every chromatin profile in the model's feature space (1548 + 1536 + 4795 profiles for Model A) has a SAD and log-odds value per variant, matching the per-variant per-profile structure of the Basset SAD tutorial
- SAD values equal the computed difference P(alt) − P(ref) for each profile
- The correct pre-trained model was used: Model A for accessibility/histone/CTCF profiles, Model B for transcription factor binding profiles

## Limitations

- DeepFun model does not consider long-range interactions such as 3D DNA contact
- Quantitative epigenomic signal information is ignored because peaks are binarized during training
- No normalization approach has yet been established to eliminate technical bias for integrating quantitative signals
- CNN-based performance may strongly depend on initial convolution filters for captured motif information

## Evidence

- [methods] Run basset_sad.py with the pre-trained model(s) to score each variant's reference and alternate alleles across all chromatin profiles: [evidence span withheld — anchor status: not_found]
- [readme] DeepFun is an updated model based on Basset Framework based on Torch7 and Python: "DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python."
- [abstract] Feature space covering 225 tissues or cell types: "including 1548 DNase I accessibility, 1536 histone mark, and 4795 transcription factor binding profiles covering 225 tissues or cell types."
- [readme] Model file locations: "http://bioinfo.uth.edu/deepfun/models/DeepFun_model/DeepFun_best_A.th.gz"
- [readme] Follow the SAD tutorial for input formatting and details: "More details can follow https://github.com/davek44/Basset/blob/master/tutorials/sad.ipynb."
