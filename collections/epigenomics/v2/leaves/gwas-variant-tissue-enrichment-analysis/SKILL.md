---
name: gwas-variant-tissue-enrichment-analysis
description: Use when you have a set of disease-associated non-coding SNPs for one
  or more traits and need to determine whether their predicted chromatin impact concentrates
  in the biologically relevant tissue (e.g. immune variants in intestine/colon, neurodegenerative
  variants in brain/spinal cord).
when_to_use_negative:
- Variants are coding or you need protein-level consequence prediction — DeepFun is
  designed for non-coding variant effects on chromatin profiles
- You need long-range (3D DNA contact) regulatory effects — DeepFun does not consider
  long-range interactions
- You need quantitative epigenomic signal magnitudes — DeepFun treats peaks as binary,
  ignoring quantitative information
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3173
tools:
- name: DeepFun
  role: Pre-trained dense epigenomic-mapped CNN used to compute reference/alternate
    allele activity values and SAD scores across 7879 epigenomic profiles covering
    225 tissues or cell types
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Underlying Torch7/Python framework whose SAD tutorial pipeline (sad.ipynb,
    basset_sad.py) computes SNP Activity Difference profiles for the encoded variant
    sequences
  repo: https://github.com/davek44/Basset
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:bb08c9ae0b6cd8d1ef04ec5eda232996c7597a55d36057aca54c7f540e57e805
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gwas-variant-tissue-enrichment-analysis

## Summary

Aggregate DeepFun/Basset SNP Activity Difference (SAD) scores for disease-associated non-coding GWAS variants by tissue/cell type and test whether impact is enriched in the disease-relevant tissue, enabling tissue-specific prioritization of GWAS loci.

## When to use

Use when you have a set of disease-associated non-coding SNPs for one or more traits and need to determine whether their predicted chromatin impact concentrates in the biologically relevant tissue (e.g. immune variants in intestine/colon, neurodegenerative variants in brain/spinal cord).

## When NOT to use

- Variants are coding or you need protein-level consequence prediction — DeepFun is designed for non-coding variant effects on chromatin profiles
- You need long-range (3D DNA contact) regulatory effects — DeepFun does not consider long-range interactions
- You need quantitative epigenomic signal magnitudes — DeepFun treats peaks as binary, ignoring quantitative information

## Inputs

- List of disease-associated non-coding GWAS SNPs with reference and alternate alleles (e.g. VCF or tab-separated variant list)
- Pre-trained DeepFun model DeepFun_best_A.th.gz (DNA accessibility, histone marks, CTCF binding profiles)
- Pre-trained DeepFun model DeepFun_best_B.th.gz (transcription factor binding profiles)
- Mapping of tissues/cell types to disease relevance (e.g. immune traits → intestines/colon; neurodegenerative/neuropsychiatric traits → brain/spinal cord)

## Outputs

- Per-variant SAD scores across all chromatin profiles from the Basset SAD pipeline
- Per-disease, per-tissue aggregated SAD table (e.g. sad_tissue_table.tsv, using maximum or mean SAD per tissue)
- Tissue-by-disease heatmap of aggregated SAD values (reproducing Figure 4A)

## How to apply

Set up the Torch7/Python environment by installing dependency packages following the Basset tutorials, then download the pre-trained DeepFun models DeepFun_best_A.th.gz (DNA accessibility, histone marks, CTCF binding) and DeepFun_best_B.th.gz (transcription factor binding sites). Encode reference and alternate allele sequences for each disease-associated noncoding variant (e.g. celiac disease, multiple sclerosis, type 1 diabetes, ulcerative colitis, Alzheimer's disease, autism, schizophrenia, coronary artery disease, type 2 diabetes) and run the Basset SAD tutorial pipeline (sad.ipynb) with each pre-trained DeepFun model to compute SNP Activity Difference scores across all chromatin profiles — 1548 DNase I accessibility, 1536 histone mark and 4795 TF binding profiles covering 225 tissues or cell types. Aggregate variant-level SAD scores by tissue/cell type for each disease (e.g. maximum or mean SAD per tissue), then compare aggregated SAD between disease-relevant and unrelated tissues. Render a tissue-by-disease heatmap (e.g. reproducing Figure 4A) and export the per-disease, per-tissue SAD table (e.g. sad_tissue_table.tsv). The analysis is judged correct when the disease-relevant tissues show visibly/quantitatively higher aggregated SAD than unrelated tissues, matching the expected trait–tissue associations.

## Related tools

- **DeepFun** (Pre-trained dense epigenomic-mapped CNN used to compute reference/alternate allele activity values and SAD scores across 7879 epigenomic profiles covering 225 tissues or cell types) — https://github.com/bsml320/DeepFun
- **Basset** (Underlying Torch7/Python framework whose SAD tutorial pipeline (sad.ipynb, basset_sad.py) computes SNP Activity Difference profiles for the encoded variant sequences) — https://github.com/davek44/Basset

## Examples

```
basset_sad.py -t DeepFun_best_A.th -s sad_out -n 300 variants.bed ref_seq.hdf5
```

## Evaluation signals

- SAD scores are produced for both alleles of every input SNP across all 7879 chromatin profiles (1548 DNase, 1536 histone mark, 4795 TF binding)
- Aggregated per-tissue SAD table has one row per tissue/cell type (225 total) per disease with no missing tissue entries
- Immune-related trait loci show higher aggregated SAD in intestines/colon; neurodegenerative and neuropsychiatric loci show higher SAD in brain/spinal cord (expected trait–tissue association from the paper)
- Heatmap reproduces the tissue-by-disease pattern of Figure 4A with disease-relevant tissues as high-SAD outliers
- Reference and alternate allele sequences were correctly encoded/centered on the SNP position before prediction (SAD = difference of predicted activities)

## Limitations

- DeepFun model does not consider long-range interactions such as 3D DNA contact
- Quantitative epigenomic signal information is ignored because peaks are binarized during training
- Performance of CNN-based methods may strongly depend on initial convolution filters for captured motif information
- No normalization approach is yet established to eliminate technical bias when integrating quantitative signals

## Evidence

- [methods] SAD workflow grounding: "More details can follow https://github.com/davek44/Basset/blob/master/tutorials/sad.ipynb."
- [abstract] Feature space: "including 1548 DNase I accessibility, 1536 histone mark, and 4795 transcription factor binding profiles covering 225 tissues or cell types."
- [abstract] Tissue-specific validation finding: "By using the datasets from various GWAS studies, we conducted independent validations and demonstrated the functions of the DeepFun web server in predicting the effect of a non-coding variant in a"
- [readme] README local analysis instructions: "DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python."
- [readme] Model download URLs: "http://bioinfo.uth.edu/deepfun/models/DeepFun_model/DeepFun_best_A.th.gz"
- [discussion] Long-range interaction limitation: "DeepFun model has not considered long-range interactions, such as 3D DNA contact"
