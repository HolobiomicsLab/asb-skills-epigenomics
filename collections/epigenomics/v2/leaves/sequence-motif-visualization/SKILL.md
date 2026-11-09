---
name: sequence-motif-visualization
description: 'Use when a single non-coding SNP needs motif-level interpretation: predicting
  how every base substitution within ~200 bp of the variant alters a specific chromatin
  profile (e.g. NFE2 binding) and visualizing informative motifs.'
when_to_use_negative:
- The question is variant-level prioritization across many SNPs rather than motif
  dissection of one locus — use DeepFun's SNP Activity Difference (sad.ipynb) workflow
  instead of saturated mutagenesis
- The effect of interest involves long-range interactions such as 3D DNA contact,
  which the DeepFun model does not consider
- Quantitative (non-binarized) epigenomic signal magnitude matters, since DeepFun
  treats peaks as binary during training
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: DeepFun
  role: Deep CNN pre-trained on ENCODE/Roadmap chromatin profiles whose weights are
    used to score every mutation's effect on a specific profile
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Provides the saturated mutagenesis framework, basset_sat_vcf.py script, and
    sat_mut.ipynb tutorial used to compute and visualize per-substitution activity
    differences
  repo: https://github.com/davek44/Basset
- name: Torch7
  role: Runtime dependency for loading and running the pre-trained DeepFun/Basset
    Torch model
  repo: https://github.com/davek44/Basset
- name: Python
  role: Runs the downstream saturated mutagenesis scripts and heat map generation
    (Basset's Python dependencies)
  repo: https://github.com/davek44/Basset
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:a8df137d2d56898558070172a9282c496f6d7a65f7ae3e0a6bf1d98abd7d0079
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  - Python
  - Torch7
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# sequence-motif-visualization

## Summary

Visualizes the sequence features (motifs) a deep learning chromatin model learned around a query variant by running in silico saturated mutagenesis with DeepFun/Basset and rendering SAD heat maps for reference and alternative alleles.

## When to use

Use when a single non-coding SNP needs motif-level interpretation: predicting how every base substitution within ~200 bp of the variant alters a specific chromatin profile (e.g. NFE2 binding) and visualizing informative motifs.

## When NOT to use

- The question is variant-level prioritization across many SNPs rather than motif dissection of one locus — use DeepFun's SNP Activity Difference (sad.ipynb) workflow instead of saturated mutagenesis
- The effect of interest involves long-range interactions such as 3D DNA contact, which the DeepFun model does not consider
- Quantitative (non-binarized) epigenomic signal magnitude matters, since DeepFun treats peaks as binary during training

## Inputs

- Genomic sequence centered on the query variant (e.g. rs6426833) formatted for Basset's sat-mut workflow
- Pre-trained DeepFun model file (DeepFun_best_B.th.gz for transcription factor binding profiles; dnacnn_best_A/DeepFun_best_A weights for accessibility, histone marks and CTCF)
- VCF or SNP specification giving the reference and alternative alleles
- Index/identifier of the chromatin profile of interest among DeepFun's 7879 ENCODE/Roadmap epigenomic tasks

## Outputs

- SAD (SNP Activity Difference) heat maps for reference and alternative alleles over the +/-200 bp region surrounding the variant
- Predicted binding/activity values for every single-nucleotide substitution across the sequence window for the selected profile
- Figure file (e.g. sat_mut_rs6426833_NFE2.png) and identification of the learned motif (e.g. NFE2 motif TGACTCAGCA)

## How to apply

Download the pre-trained model appropriate to the profile — DeepFun_best_B.th.gz from http://bioinfo.uth.edu/deepfun/models/DeepFun_model/ for transcription factor binding profiles like NFE2, or the model A weights for accessibility/histone/CTCF profiles — then install Torch7 and Python dependencies following the Basset tutorials. Retrieve the genomic sequence centered on the query variant and run Basset's saturated mutagenesis workflow (basset_sat_vcf.py, per the sat_mut.ipynb tutorial) to compute predicted activity for every single-nucleotide substitution across the window. Extract the target chromatin profile from the multi-task output (DeepFun integrates 1548 DNase I accessibility, 1536 histone mark, and 4795 transcription factor binding profiles) and generate SAD heat maps for the reference and alternative alleles over the +/-200 bp region. Judge correctness by checking whether substitutions overlapping the learned motif (e.g. TGACTCAGCA for NFE2) decrease predicted binding while the disease-associated allele change alters activity, matching the expected figure (e.g. Figure 4B for rs6426833).

## Related tools

- **DeepFun** (Deep CNN pre-trained on ENCODE/Roadmap chromatin profiles whose weights are used to score every mutation's effect on a specific profile) — https://github.com/bsml320/DeepFun
- **Basset** (Provides the saturated mutagenesis framework, basset_sat_vcf.py script, and sat_mut.ipynb tutorial used to compute and visualize per-substitution activity differences) — https://github.com/davek44/Basset
- **Torch7** (Runtime dependency for loading and running the pre-trained DeepFun/Basset Torch model) — https://github.com/davek44/Basset
- **Python** (Runs the downstream saturated mutagenesis scripts and heat map generation (Basset's Python dependencies)) — https://github.com/davek44/Basset

## Examples

```
python basset_sat_vcf.py -t cuda -o sat_mut_out/ DeepFun_best_B.th rs6426833.vcf
```

## Evaluation signals

- The heat map shows that only the disease-associated allele change (e.g. A-to-G at rs6426833) increases predicted NFE2 binding activity while other nearby substitutions do not impact SAD
- Substitutions across the predicted motif positions (TGACTCAGCA for NFE2) decrease predicted binding activity, consistent with the motif being learned by the model
- The extracted profile corresponds to the correct task among DeepFun's 7879 epigenomic profiles (TF binding profile from model B)
- The generated figure reproduces the article's expected result (e.g. Figure 4B) and is saved with the variant/profile naming convention

## Limitations

- DeepFun's model has not considered long-range interactions, such as 3D DNA contact
- Training treats peaks as binary, ignoring quantitative epigenomic signal information, and no normalization approach has yet been established to integrate quantitative signals
- CNN-based method performance may strongly depend on initial convolution filters for captured motif information
- No software version is stated for DeepFun

## Evidence

- [readme] Saturated mutagenesis is a powerful tool both for dissecting a specific sequence of interest and understanding what the model learned.: "Saturated mutagenesis is a powerful tool both for dissecting a specific sequence of interest and understanding what the model learned."
- [methods] DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python.: "DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python."
- [results] the in silico saturated mutagenesis predicted binding activity of the A allele is consistent with the NFE2 motif TGACTCAGCA: "with the help of in silico saturated mutagenesis analysis, the DeepFun web server will be useful for interpreting potential target gene's motifs around functional variants"
- [readme] dnacnn_best_A.zip (DNA accessibility sites, histone marks and Transcription factor CTCF binding sites): "you'll need to download the pre-trained model from github here: dnacnn_best_A.zip (DNA accessibility sites, histone marks and Transcription factor CTCF binding sites)"
