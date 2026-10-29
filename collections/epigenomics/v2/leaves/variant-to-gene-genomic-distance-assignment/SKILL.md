---
name: variant-to-gene-genomic-distance-assignment
description: Use when you have a prioritized list of non-coding variant scores (e.g.,
  ncDeepBrain output on a VCF/Annovar bed input) and need them attached to genes to
  build gene-level feature vectors for disease-gene prioritization.
when_to_use_negative:
- When you have direct regulatory evidence (eQTL or enhancer-promoter links) linking
  variants to target genes — the paper notes closest-gene assignment 'may be less
  optimal' for non-promoter regulatory elements
- When variants are already coding and directly gene-assigned — no distance-based
  mapping is needed
- When variants lie more than 100 kb from any annotated gene — the assignment rule
  will not link them to a gene
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: iMEGES
  role: Implements the two-step pipeline; step 2 links non-coding variants to genes
    and produces the iMEGES gene score
  repo: https://github.com/WGLab/iMEGES
- name: TensorFlow
  role: Backend for the Keras deep neural network that consumes the gene-level features
    built from distance-assigned variants
  repo: https://www.tensorflow.org/
- name: GTEx
  role: General gene score used as a feature in the gene-level vector after variant-to-gene
    assignment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:da6d1d661316ec5f1d962a13e751af38b669b1d28bfc6ce5d60470778ff44239
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/iMEGES
  tools:
  - GTEx
  - TensorFlow
  - iMEGES
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# variant-to-gene-genomic-distance-assignment

## Summary

Assigns non-coding variants to genes by nearest-genomic-distance (≤100 kb) so that variant-level scores (e.g., the ncDeepBrain brain-variant score) can be aggregated into gene-level features for iMEGES gene prioritization. It is the bridging step between whole-genome variant prioritization and deep-learning gene scoring.

## When to use

Use when you have a prioritized list of non-coding variant scores (e.g., ncDeepBrain output on a VCF/Annovar bed input) and need them attached to genes to build gene-level feature vectors for disease-gene prioritization.

## When NOT to use

- When you have direct regulatory evidence (eQTL or enhancer-promoter links) linking variants to target genes — the paper notes closest-gene assignment 'may be less optimal' for non-promoter regulatory elements
- When variants are already coding and directly gene-assigned — no distance-based mapping is needed
- When variants lie more than 100 kb from any annotated gene — the assignment rule will not link them to a gene

## Inputs

- Non-coding variant coordinates with ncDeepBrain scores (Annovar/bed format output of iMEGES step 1)
- Gene annotation coordinates for the reference genome build (e.g., hg)
- Gene-level feature values: RVIS, GTEx, haploinsufficiency, Phenolyzer, CNV q-values, de novo mutation scores

## Outputs

- Variant-to-gene assignments (each non-coding variant mapped to its closest gene within 100 kb)
- Gene-level feature matrix with aggregated ncDeepBrain scores and bPCA-imputed missing values
- iMEGES gene scores (deep learning susceptibility probability per mutated gene), written as imeges_gene_scores.tsv

## How to apply

Annotate each non-coding variant position (from the Annovar/bed-format input processed by iMEGES step 1) against gene coordinates, and assign each variant to its closest gene in genome distance, accepting assignments only when the distance to the gene is 100 kb or less. Because a gene may harbor more than one mutation, retain all mutations mapping to the same gene rather than collapsing to a single variant. Compose the gene-level feature vector F = (ncDeepBrain, RVIS, GTEx, haploinsufficiency, Phenolyzer, CNV, de novo mutation), attaching each mapped variant's ncDeepBrain score, then impute missing values with bPCA and apply default fills (0.5 for missing CNV q-values, 0 for missing RVIS). The downstream second deep learning framework (Keras with TensorFlow backend, four hidden layers with dropout, sigmoid output) consumes these gene features and outputs the iMEGES gene score, a 0–1 susceptibility probability per mutated gene.

## Related tools

- **iMEGES** (Implements the two-step pipeline; step 2 links non-coding variants to genes and produces the iMEGES gene score) — https://github.com/WGLab/iMEGES
- **TensorFlow** (Backend for the Keras deep neural network that consumes the gene-level features built from distance-assigned variants) — https://www.tensorflow.org/
- **GTEx** (General gene score used as a feature in the gene-level vector after variant-to-gene assignment)

## Examples

```
python ncDeepBrain.py --bed1 variants.bed -E brain_eqtls.bed -B bPCA.R -o imeges_output -buildver hg38
```

## Evaluation signals

- Every assigned variant maps to exactly one nearest gene, and no assignment exceeds 100 kb genomic distance
- Genes harboring multiple mutations retain all mapped variants when building the gene feature vector
- Missing feature values after mapping are imputed (bPCA) with the specified defaults (0.5 CNV q-value, 0 RVIS)
- Final gene scores are sigmoid outputs bounded in 0–1; ten-fold cross-validation ROC/AUC on schizophrenia (~57%) and ASD (~58%) datasets is reproducible

## Limitations

- Closest-gene assignment is acknowledged as suboptimal for non-promoter regulatory elements; a variant may be functionally linked to a distal gene
- Some disease-specific scores rely on imputed GWAS data, where hits may be proxy markers rather than true causal variants
- Genes with no variant within 100 kb cannot receive an ncDeepBrain score through this mapping

## Evidence

- [readme] closest-gene-within-100kb rule: "To link the non-coding variants to a gene, we assigned each of these variants to it closet gene in term of genome distance, i.e., distance to gene is 100 KB or less."
- [discussion] closest-gene limitation: [evidence span withheld — anchor status: not_found]
- [intro] gene prioritization integration step: "we used another deep learning framework to integrate the ncDeepBrain score, general gene scores (such as GTEx), and disease-specific scores to prioritize mutated genes for mental disorders"
- [readme] multiple mutations per gene: "Considering the fact that maybe some genes harbor more than one mutations, we consider all the mutations and prioritize each of the variants for the specific mental disorders genes."
- [results] gene score definition: "the output of the second Step is the deep learning probability for each of the mutated gene and call it the iMEGES gene score, which measure the susceptibility potential for this gene to be involved"
