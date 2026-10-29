---
name: transcription-factor-binding-profile-interpretation
description: Use when a non-coding SNP needs functional interpretation against a transcription
  factor binding profile, and you want saturated mutagenesis SAD heat maps that reveal
  which motif and which allele-specific substitutions drive predicted binding activity.
when_to_use_negative:
- The question concerns DNA accessibility, histone marks, or CTCF binding rather than
  transcription factor binding profiles — model A (dnacnn_best_A.zip / DeepFun_best_A.th.gz)
  is the appropriate pre-trained model instead
- You need to assess long-range regulatory effects such as 3D DNA contact, which the
  DeepFun model does not consider
- You only need a variant's predicted effect without motif dissection — the SNP Activity
  Difference (SAD) workflow is simpler than saturated mutagenesis
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0749
tools:
- name: DeepFun
  role: Deep CNN that predicts reference and alternative allele activity for 4795
    transcription factor binding profiles; supplies the pre-trained model B used for
    saturated mutagenesis
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Framework providing basset_sat_vcf.py and the sat_mut/sad tutorials that DeepFun's
    local analysis is built on
  repo: https://github.com/davek44/Basset
- name: Torch7
  role: Runtime dependency for loading and running the .th model file
- name: Python
  role: Runtime for basset_sat_vcf.py and downstream heat-map generation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:670dea470e1e2adf8e08750447b2c9c78b3cc1e8ce5bd8cbeefa2c3e3ea5b98e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  - Python
  - Torch7
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# transcription-factor-binding-profile-interpretation

## Summary

Runs DeepFun's in silico saturated mutagenesis (via Basset's sat-mut workflow) to predict the effect of every single-nucleotide substitution within ~200 bp of a query SNP on transcription factor binding profiles, and reads out the motif (e.g. NFE2 TGACTCAGCA) that explains the predicted binding change.

## When to use

Use when a non-coding SNP needs functional interpretation against a transcription factor binding profile, and you want saturated mutagenesis SAD heat maps that reveal which motif and which allele-specific substitutions drive predicted binding activity.

## When NOT to use

- The question concerns DNA accessibility, histone marks, or CTCF binding rather than transcription factor binding profiles — model A (dnacnn_best_A.zip / DeepFun_best_A.th.gz) is the appropriate pre-trained model instead
- You need to assess long-range regulatory effects such as 3D DNA contact, which the DeepFun model does not consider
- You only need a variant's predicted effect without motif dissection — the SNP Activity Difference (SAD) workflow is simpler than saturated mutagenesis

## Inputs

- Pre-trained DeepFun model file for transcription factor binding profiles (DeepFun_best_B.th.gz)
- Genomic sequence centered on the query SNP (e.g. rs6426833), formatted for Basset's sat-mut workflow
- VCF entry for the query SNP (reference and alternative alleles)

## Outputs

- In silico saturated mutagenesis SAD heat maps for reference and alternative alleles over +/-200 bp around the SNP
- Extracted transcription factor binding profile predictions (e.g. NFE2) from the multi-task output
- Figure identifying the causal allele substitution and the informative motif (e.g. sat_mut_rs6426833_NFE2.png)

## How to apply

Download the transcription-factor pre-trained model DeepFun_best_B.th.gz (model B covers transcription factor binding sites; model A covers DNA accessibility, histone marks and CTCF). Retrieve the genomic sequence centered on the query SNP (e.g. rs6426833) and format it for Basset's sat-mut workflow, installing Torch7 and Python dependencies per the Basset tutorials. Run basset_sat_vcf.py with the DeepFun_best_B model to compute predicted activity for every single-nucleotide substitution across the window, then extract the transcription factor binding profile of interest (e.g. NFE2) from the multi-task output. Generate SAD heat maps for reference and alternative alleles over the +/-200 bp region, and interpret them by checking that substitutions within the identified motif (e.g. TGACTCAGCA) decrease predicted binding while the causal allele substitution (e.g. A-to-G at rs6426833) increases it, matching Figure 4B. The rationale is that saturated mutagenesis dissects a specific sequence of interest and reveals what the CNN model learned about motif dependency.

## Related tools

- **DeepFun** (Deep CNN that predicts reference and alternative allele activity for 4795 transcription factor binding profiles; supplies the pre-trained model B used for saturated mutagenesis) — https://github.com/bsml320/DeepFun
- **Basset** (Framework providing basset_sat_vcf.py and the sat_mut/sad tutorials that DeepFun's local analysis is built on) — https://github.com/davek44/Basset
- **Torch7** (Runtime dependency for loading and running the .th model file)
- **Python** (Runtime for basset_sat_vcf.py and downstream heat-map generation)

## Examples

```
python basset_sat_vcf.py DeepFun_best_B.th.gz rs6426833.vcf && python plot_sat_heatmap.py --task NFE2 --out sat_mut_rs6426833_NFE2.png
```

## Evaluation signals

- Only the causal allele substitution (rs6426833 A-to-G) increases predicted NFE2 binding activity while all other nearby substitutions do not impact SAD
- Substitutions within the identified NFE2 motif TGACTCAGCA decrease predicted binding activity in the heat map
- Predicted binding of the A allele is consistent with the NFE2 motif TGACTCAGCA, reproducing Figure 4B
- The multi-task output successfully contains the specific transcription factor binding profile extracted from model B's 4795 transcription factor tasks

## Limitations

- DeepFun has not considered long-range interactions, such as 3D DNA contact
- Quantitative epigenomic signal information is ignored because peaks were binarized during training
- Performance of CNN-based methods may strongly depend on initial convolution filters for captured motif information

## Evidence

- [readme] Saturated mutagenesis is a powerful tool both for dissecting a specific sequence of interest and understanding what the model learned.: "Saturated mutagenesis is a powerful tool both for dissecting a specific sequence of interest and understanding what the model learned."
- [discussion] with the help of in silico saturated mutagenesis analysis, the DeepFun web server will be useful for interpreting potential target gene's motifs around functional variants: "with the help of in silico saturated mutagenesis analysis, the DeepFun web server will be useful for interpreting potential target gene's motifs around functional variants"
- [methods] or dnacnn_best_B.zip (Transcription factors binding sites).: "or dnacnn_best_B.zip (Transcription factors binding sites)."
- [methods] More details can follow https://github.com/davek44/Basset/blob/master/tutorials/sat_mut.ipynb.: "More details can follow https://github.com/davek44/Basset/blob/master/tutorials/sat_mut.ipynb."
- [abstract] including 1548 DNase I accessibility, 1536 histone mark, and 4795 transcription factor binding profiles covering 225 tissues or cell types.: "including 1548 DNase I accessibility, 1536 histone mark, and 4795 transcription factor binding profiles covering 225 tissues or cell types."
