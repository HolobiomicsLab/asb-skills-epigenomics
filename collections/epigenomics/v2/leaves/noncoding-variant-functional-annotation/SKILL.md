---
name: noncoding-variant-functional-annotation
description: Use when you need to prioritize or interpret non-coding GWAS variants
  by their predicted effect on chromatin accessibility, histone marks, or TF binding
  in a tissue- or cell type-specific manner at single-nucleotide resolution.
when_to_use_negative:
- The variant's mechanism depends on long-range interactions such as 3D DNA contact
  — the DeepFun model does not consider these
- Quantitative (continuous) epigenomic signal strength matters, since training binarized
  'peaks' and the model ignores quantitative information
- Coding variants are the target — the feature space and validation focus on non-coding
  regions, where >90% of GWAS variants reside
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3517
tools:
- name: DeepFun
  role: Deep CNN pre-trained on 7879 epigenomic profiles; predicts reference/alternate
    allele activity per profile for SAD scoring and saturated mutagenesis
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Underlying framework providing basset_sad.py, basset_sat_vcf.py, tutorials
    (sad.ipynb, sat_mut.ipynb) and dependency installation
  repo: https://github.com/davek44/Basset
- name: DeepSEA
  role: External comparator; its SAD scores for the same variants are correlated with
    DeepFun's averaged per-assay scores
- name: Torch7
  role: Runtime dependency for executing the pre-trained Torch model files locally
- name: Python
  role: Runtime dependency for preprocessing scripts and downstream SAD/sat-mut analysis
    in the Basset framework
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:73f8423dd3c11b12831870110be34583ba1084b5bf696690ba424710e299cc9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  - DeepSEA
  - Python
  - Torch7
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# noncoding-variant-functional-annotation

## Summary

Annotate non-coding SNPs with tissue- and cell type-specific functional impact scores using DeepFun, a Basset-framework deep CNN pre-trained on 1548 DNase I accessibility, 1536 histone mark, and 4795 transcription factor binding profiles from ENCODE and Roadmap covering 225 tissues/cell types.

## When to use

Use when you need to prioritize or interpret non-coding GWAS variants by their predicted effect on chromatin accessibility, histone marks, or TF binding in a tissue- or cell type-specific manner at single-nucleotide resolution.

## When NOT to use

- The variant's mechanism depends on long-range interactions such as 3D DNA contact — the DeepFun model does not consider these
- Quantitative (continuous) epigenomic signal strength matters, since training binarized 'peaks' and the model ignores quantitative information
- Coding variants are the target — the feature space and validation focus on non-coding regions, where >90% of GWAS variants reside

## Inputs

- Query SNP list / VCF of non-coding variants (reference and alternate alleles), formatted per the Basset SAD tutorial (sad.ipynb)
- Pre-trained DeepFun model file DeepFun_best_A.th.gz (accessibility, histone marks, CTCF) and/or DeepFun_best_B.th.gz (TF binding sites)
- Genomic sequence centered on the query variant for saturated mutagenesis (sat_mut.ipynb input format)
- DeepSEA SAD scores for the same variants and assays (optional, for cross-tool correlation)

## Outputs

- sad_scores.tsv: per-variant, per-feature SAD = P(alt) − P(ref) and log-odds difference values across all chromatin profiles in the model's feature space
- sad_tissue_table.tsv: per-disease, per-tissue aggregated SAD table plus tissue-by-disease heatmap (e.g. Figure 4A)
- sat_mut_rs6426833_NFE2.png: SAD heat maps for reference and alternative alleles over the ±200 bp region from saturated mutagenesis
- deepfun_deepsea_sad_pcc.csv: per-assay Pearson correlation coefficients between DeepFun and DeepSEA averaged SAD scores

## How to apply

Set up the Torch7/Python environment following the Basset tutorials, then download the pre-trained models DeepFun_best_A.th.gz (DNA accessibility, histone marks, CTCF) and/or DeepFun_best_B.th.gz (transcription factor binding sites). Format the query SNPs per the Basset SAD tutorial (sad.ipynb) and run basset_sad.py, which predicts activity probabilities for both alleles and computes SAD = P(alt) − P(ref) plus a log-odds difference for every chromatin profile, writing per-variant per-profile scores to sad_scores.tsv. To validate tissue specificity, aggregate variant-level SAD scores by tissue/cell type (e.g. maximum or mean SAD per tissue) and compare disease-relevant versus unrelated tissues. For motif interpretation, run in silico saturated mutagenesis (basset_sat_vcf.py with sat_mut.ipynb) to compute predicted activity for every single-nucleotide substitution within ±200 bp of the variant and generate SAD heat maps. Cross-method consistency can be checked by averaging SAD scores across tracks per assay and computing Pearson correlation against DeepSEA (expected PCCs: CTCF 0.883, DNase-seq 0.751, H3K4me3 0.490, H3K9ac 0.451).

## Related tools

- **DeepFun** (Deep CNN pre-trained on 7879 epigenomic profiles; predicts reference/alternate allele activity per profile for SAD scoring and saturated mutagenesis) — https://github.com/bsml320/DeepFun
- **Basset** (Underlying framework providing basset_sad.py, basset_sat_vcf.py, tutorials (sad.ipynb, sat_mut.ipynb) and dependency installation) — https://github.com/davek44/Basset
- **DeepSEA** (External comparator; its SAD scores for the same variants are correlated with DeepFun's averaged per-assay scores)
- **Torch7** (Runtime dependency for executing the pre-trained Torch model files locally)
- **Python** (Runtime dependency for preprocessing scripts and downstream SAD/sat-mut analysis in the Basset framework)

## Examples

```
python basset_sad.py -t DeepFun_best_A.th.gz -o sad_scores.tsv query_snps.vcf  # per Basset SAD tutorial (sad.ipynb)
```

## Evaluation signals

- sad_scores.tsv exists with one row per query variant and SAD/log-odds columns for every chromatin profile (1548 DNase I + 1536 histone mark + 4795 TF binding for Model A's feature space spanning 225 tissues/cell types)
- Tissue-specificity check reproduces Figure 4A: immune-related trait loci show higher SAD in intestines/colon; neurodegenerative/neuropsychiatric loci show higher SAD in brain/spinal cord
- Sat-mut heat map for rs6426833 on the NFE2 profile shows only the A-to-G mutation increasing predicted binding, consistent with the NFE2 motif TGACTCAGCA (Figure 4B)
- DeepFun–DeepSEA per-assay PCCs match reported values: CTCF ≈ 0.883 (P = 2.2×10−16), DNase-seq ≈ 0.751 (P = 1.46×10−10), H3K4me3 ≈ 0.490, H3K9ac ≈ 0.451
- SAD values are signed per allele: P(alt) − P(ref), so positive values indicate increased activity of the alternate allele for a given profile

## Limitations

- DeepFun model has not considered long-range interactions, such as 3D DNA contact
- Training treated peaks as binary format, ignoring quantitative epigenomic signal information
- No normalization approach yet established to eliminate technical bias for integrating quantitative signals
- Performance of most CNN-based methods may strongly depend on initial convolution filters for captured motif information
- Local analysis requires Torch7 and Python dependencies per Basset tutorials; no software version is stated for DeepFun

## Evidence

- [abstract] feature space: "including 1548 DNase I accessibility, 1536 histone mark, and 4795 transcription factor binding profiles covering 225 tissues or cell types."
- [abstract] web server availability: "Here, we present a user-friendly web server, DeepFun (https://bioinfo.uth.edu/deepfun/), to assess the functional activity of non-coding genetic variants."
- [methods] Basset framework dependency: "DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python."
- [methods] SAD tutorial reference: "More details can follow https://github.com/davek44/Basset/blob/master/tutorials/sad.ipynb."
- [readme] model download files: "you'll need to download the pre-trained model from github here: dnacnn_best_A.zip (DNA accessibility sites, histone marks and Transcription factor CTCF binding sites) or dnacnn_best_B.zip"
- [readme] SAD capability description: "The most useful capability of DeepFun right now is to annotate noncoding genomic variants with their influence on functional properties like accessibility."
- [results] tissue-specific validation finding: [evidence span withheld — anchor status: not_found]
- [results] DeepSEA correlation finding: [evidence span withheld — anchor status: not_found]
- [results] sat-mut motif finding: [evidence span withheld — anchor status: not_found]
- [discussion] long-range limitation: "DeepFun model has not considered long-range interactions, such as 3D DNA contact"
