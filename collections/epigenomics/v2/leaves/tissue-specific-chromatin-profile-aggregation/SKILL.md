---
name: tissue-specific-chromatin-profile-aggregation
description: 'Use when prioritizing non-coding GWAS variants in a tissue- or cell
  type-specific manner: you have per-variant SAD scores from DeepFun pre-trained models
  and need tissue-level aggregation plus a disease-by-tissue comparison such as a
  heatmap.'
when_to_use_negative:
- The variant is coding or you need long-range (3D DNA contact) effects — DeepFun's
  model does not consider long-range interactions such as 3D DNA contact
- You require quantitative epigenomic signal magnitudes — DeepFun training binarized
  peaks and ignored quantitative information
- Input is already aggregated tissue-level enrichment data rather than variant-level
  SAD scores requiring the Basset sad.ipynb pipeline
edam_operation: http://edamontology.org/operation_0484
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3517
tools:
- name: DeepFun
  role: Pre-trained CNN models (DeepFun_best_A.th.gz, DeepFun_best_B.th.gz) used to
    predict reference/alternate allele effects across 7879 epigenomic profiles from
    ENCODE and Roadmap
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Framework whose SAD tutorial pipeline (sad.ipynb, basset_sad.py) computes
    SNP Activity Difference profiles from the DeepFun pre-trained models
  repo: https://github.com/davek44/Basset
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:85a815af5ba9f1abd8893c5bc08300b52721eb23566ace5435a3ac2bd71b96ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  edam_operation: http://edamontology.org/operation_0484
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# tissue-specific-chromatin-profile-aggregation

## Summary

Aggregate per-variant SNP Activity Difference (SAD) scores from DeepFun/Basset across 1548 DNase I accessibility, 1536 histone mark and 4795 TF binding profiles (225 tissues/cell types) to test whether disease-associated non-coding variants show higher predicted chromatin impact in disease-relevant tissues.

## When to use

Use when prioritizing non-coding GWAS variants in a tissue- or cell type-specific manner: you have per-variant SAD scores from DeepFun pre-trained models and need tissue-level aggregation plus a disease-by-tissue comparison such as a heatmap.

## When NOT to use

- The variant is coding or you need long-range (3D DNA contact) effects — DeepFun's model does not consider long-range interactions such as 3D DNA contact
- You require quantitative epigenomic signal magnitudes — DeepFun training binarized peaks and ignored quantitative information
- Input is already aggregated tissue-level enrichment data rather than variant-level SAD scores requiring the Basset sad.ipynb pipeline

## Inputs

- Disease-associated noncoding variant list (e.g., GWAS SNPs for celiac disease, multiple sclerosis, type 1 diabetes, ulcerative colitis, Alzheimer's disease, autism, schizophrenia, CAD, type 2 diabetes) with reference and alternate alleles
- Pre-trained DeepFun models: DeepFun_best_A.th.gz and DeepFun_best_B.th.gz (http://bioinfo.uth.edu/deepfun/models/DeepFun_model/)
- Basset framework (Torch7 + Python) with dependencies installed per Basset tutorials
- Tissue/cell type mapping for the 7879 epigenomic profiles (1548 DNase I, 1536 histone mark, 4795 TF binding) covering 225 tissues or cell types

## Outputs

- Per-variant, per-profile SNP Activity Difference (SAD) scores from the Basset SAD pipeline
- Per-disease, per-tissue aggregated SAD table (sad_tissue_table.tsv)
- Tissue-by-disease SAD heatmap reproducing Figure 4A

## How to apply

Set up the Torch7/Python environment per the Basset tutorials and download the pre-trained DeepFun models DeepFun_best_A.th.gz (DNA accessibility, histone marks, CTF/CTCF binding) and DeepFun_best_B.th.gz (transcription factor binding) from bioinfo.uth.edu/deepfun/models. Encode reference and alternate allele sequences for each disease-associated noncoding variant (e.g., celiac disease, multiple sclerosis, type 1 diabetes, ulcerative colitis, Alzheimer's disease, autism, schizophrenia, coronary artery disease, type 2 diabetes) and run the Basset SAD tutorial pipeline (sad.ipynb) with each pre-trained model to compute SNP Activity Difference scores across all chromatin profiles. Aggregate variant-level SAD scores by tissue/cell type for each disease (e.g., maximum or mean SAD per tissue), then compare aggregated SAD between disease-relevant and unrelated tissues. Render a tissue-by-disease heatmap (reproducing Figure 4A) and export the per-disease, per-tissue SAD table as sad_tissue_table.tsv. Correct application is confirmed when immune-related trait loci show higher aggregated SAD in intestines/colon and neurodegenerative/neuropsychiatric loci show higher SAD in brain/spinal cord, matching the article's validation finding.

## Related tools

- **DeepFun** (Pre-trained CNN models (DeepFun_best_A.th.gz, DeepFun_best_B.th.gz) used to predict reference/alternate allele effects across 7879 epigenomic profiles from ENCODE and Roadmap) — https://github.com/bsml320/DeepFun
- **Basset** (Framework whose SAD tutorial pipeline (sad.ipynb, basset_sad.py) computes SNP Activity Difference profiles from the DeepFun pre-trained models) — https://github.com/davek44/Basset

## Examples

```
basset_sad.lua -t 1 model_th gpu 1 input.bed out.txt  # compute SAD profiles per Basset sad.ipynb with DeepFun_best_A.th.gz, then aggregate per tissue in Python
```

## Evaluation signals

- SAD output covers all 7879 profiles (1548 DNase I + 1536 histone mark + 4795 TF binding) spanning 225 tissues or cell types
- Aggregated SAD is higher in disease-relevant tissues than unrelated tissues per disease (immune traits: intestines/colon; neurodegenerative/neuropsychiatric: brain/spinal cord)
- Tissue-by-disease heatmap is reproducible against Figure 4A
- sad_tissue_table.tsv contains one aggregated SAD value per disease x tissue combination
- Reference and alternate allele sequences were both encoded for every variant before SAD computation

## Limitations

- DeepFun model has not considered long-range interactions, such as 3D DNA contact
- Quantitative epigenomic signal information is ignored because peaks are binarized during training
- CNN-based method performance may strongly depend on initial convolution filters for captured motif information
- No normalization approach yet established to eliminate technical bias for integrating quantitative signals

## Evidence

- [abstract] DeepFun's feature space includes 1548 DNase I accessibility, 1536 histone mark, and 4795 transcription factor binding profiles covering 225 tissues or cell types: "including 1548 DNase I accessibility, 1536 histone mark, and 4795 transcription factor binding profiles covering 225 tissues or cell types."
- [methods] Compute SNP Activity Difference profiles following the Basset sad tutorial: "More details can follow https://github.com/davek44/Basset/blob/master/tutorials/sad.ipynb."
- [results] Immune trait loci show higher SAD in intestines and colon; neurodegenerative/neuropsychiatric loci in brain and spinal cord: [evidence span withheld — anchor status: not_found]
- [readme] DeepFun local analysis is based on Basset in Torch7/Python and requires the pre-trained model: "DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python."
- [discussion] Model limitation: no long-range interactions considered: "DeepFun model has not considered long-range interactions, such as 3D DNA contact"
