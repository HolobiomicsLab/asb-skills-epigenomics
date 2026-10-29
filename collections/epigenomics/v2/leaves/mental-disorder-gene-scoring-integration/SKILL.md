---
name: mental-disorder-gene-scoring-integration
description: Use when prioritizing mutated genes from whole-genome/exome sequencing
  of a personal genome for mental disorders, after non-coding variants have been scored
  (ncDeepBrain) and need to be linked to genes and integrated with gene-level features.
when_to_use_negative:
- Input is raw variant calls without step-1 ncDeepBrain scores or the required gene-level
  annotations (RVIS, GTEx, haploinsufficiency, Phenolyzer, CNV, de novo) — the framework
  integrates precomputed features, not raw sequence
- The goal is scoring coding variants or protein-impact prediction — this skill targets
  gene-level susceptibility prioritization using non-coding and gene-based scores
- Phenotype/disease context is absent and no disease-specific scores (e.g., PGC GWAS-derived)
  can be assembled — the prioritization is disease-specific by design
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3517
tools:
- name: iMEGES
  role: Pipeline whose step 2 performs the gene-prioritization deep learning integration
    and outputs the iMEGES gene score
  repo: https://github.com/WGLab/iMEGES
- name: TensorFlow
  role: Backend for the Keras deep neural network used in the gene-prioritization
    step
  repo: https://github.com/WGLab/iMEGES
- name: GTEx
  role: Source of tissue-specific eQTL gene scores (44 tissues, q-value ≤ 0.05) used
    as a general gene-level feature
- name: Phenolyzer
  role: Provides the phenotype-based disease-specific gene score integrated as a feature
  repo: http://phenolyzer.wglab.org/
- name: bPCA
  role: Bayesian Principal Component Analysis used to impute missing gene-level feature
    values
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:38e8334d24ffb17de50bddd0bf6d6fade96a632f950b1ef39f5d0c71378f1158
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/iMEGES
  tools:
  - GTEx
  - Phenolyzer
  - TensorFlow
  - bPCA
  - iMEGES
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mental-disorder-gene-scoring-integration

## Summary

Second step of iMEGES: a deep neural network that integrates the ncDeepBrain non-coding variant score with general gene scores (RVIS, GTEx, haploinsufficiency) and disease-specific scores (Phenolyzer, CNV, de novo mutation) to produce an iMEGES gene score — a probability measuring each mutated gene's susceptibility potential for mental disorders.

## When to use

Use when prioritizing mutated genes from whole-genome/exome sequencing of a personal genome for mental disorders, after non-coding variants have been scored (ncDeepBrain) and need to be linked to genes and integrated with gene-level features.

## When NOT to use

- Input is raw variant calls without step-1 ncDeepBrain scores or the required gene-level annotations (RVIS, GTEx, haploinsufficiency, Phenolyzer, CNV, de novo) — the framework integrates precomputed features, not raw sequence
- The goal is scoring coding variants or protein-impact prediction — this skill targets gene-level susceptibility prioritization using non-coding and gene-based scores
- Phenotype/disease context is absent and no disease-specific scores (e.g., PGC GWAS-derived) can be assembled — the prioritization is disease-specific by design

## Inputs

- ncDeepBrain non-coding variant scores from iMEGES step 1 (variant prioritization)
- General gene scores: RVIS, GTEx (44 tissues, q-value ≤ 0.05), haploinsufficiency scores
- Disease-specific scores: Phenolyzer, CNV score, de novo mutation score
- Patient genetic mutations (Annovar input/bed format) and phenotypic information
- Known brain eQTLs from CommonMind (input to step 1 feeding ncDeepBrain scores)

## Outputs

- iMEGES gene score: deep learning probability per mutated gene measuring susceptibility potential for mental disorders
- Ranked list of prioritized disease-specific genes
- ROC curves and ten-fold cross-validation AUC values for disease vs. non-disease gene discrimination

## How to apply

Attach each non-coding variant's ncDeepBrain score to its closest gene within 100 kb genomic distance, and assemble a gene-level feature vector of ncDeepBrain, RVIS, GTEx, haploinsufficiency, Phenolyzer, CNV, and de novo mutation scores; a gene harboring multiple mutations aggregates all of its variants' annotations. Before modeling, examine summary statistics and pairwise correlations of feature variables to detect outliers and collinearity, and impute missing values with bPCA (applying default fills such as 0.5 for missing CNV q-values and 0 for missing RVIS). Train a multilayer Keras network with a TensorFlow backend (four hidden layers with dropout, sigmoid output scaled 0–1, negative log-likelihood loss minimized by stochastic gradient descent with momentum) on ROSE-balanced training data, then predict a probability per mutated gene. Evaluate with ten-fold cross-validation, reporting ROC curves and AUC values for discriminating disease genes from non-disease genes (~57% AUC for schizophrenia, ~58% for ASD).

## Related tools

- **iMEGES** (Pipeline whose step 2 performs the gene-prioritization deep learning integration and outputs the iMEGES gene score) — https://github.com/WGLab/iMEGES
- **TensorFlow** (Backend for the Keras deep neural network used in the gene-prioritization step) — https://github.com/WGLab/iMEGES
- **GTEx** (Source of tissue-specific eQTL gene scores (44 tissues, q-value ≤ 0.05) used as a general gene-level feature)
- **Phenolyzer** (Provides the phenotype-based disease-specific gene score integrated as a feature) — http://phenolyzer.wglab.org/
- **bPCA** (Bayesian Principal Component Analysis used to impute missing gene-level feature values)

## Examples

```
python ncDeepBrain.py -f1 input.bed -E brain_eqtls.bed -B bPCA.R -o imeges_gene_scores.tsv -buildver hg19
```

## Evaluation signals

- Every non-coding variant carrying an ncDeepBrain score is mapped to a gene at genomic distance ≤ 100 kb; variants beyond 100 kb from any gene are not linked
- Feature table completeness: missing values are imputed via bPCA with documented defaults (e.g., 0.5 for missing CNV q-values, 0 for missing RVIS) and pairwise correlations show no severe collinearity
- Ten-fold cross-validation ROC AUC for disease vs. non-disease gene discrimination lands near reported values (~57% schizophrenia, ~58% ASD); large deviations indicate pipeline errors
- Output gene scores are sigmoid probabilities in the 0–1 range, one per mutated gene, rankable as iMEGES gene scores

## Limitations

- Non-coding variants are associated to their closest gene, which works well for promoters but may be suboptimal for other types of regulatory elements
- Some procedures rely on imputed GWAS data, where hits may be proxy markers rather than true causal variants, making the model less reliable than ideal
- Lack of large-scale high-quality data for training deep learning models constrains performance

## Evidence

- [results] Gene prioritization: integrate ncDeepBrain score, general gene scores (GTEx, RVIS, haploinsufficiency) and disease-specific scores via deep learning framework to generate iMEGES score: [evidence span withheld — anchor status: not_found]
- [readme] Closest-gene 100 kb linking rule: "To link the non-coding variants to a gene, we assigned each of these variants to it closet gene in term of genome distance, i.e., distance to gene is 100 KB or less."
- [readme] Output definition of the gene score: "the output of the second Step is the deep learning probability for each of the mutated gene and call it the iMEGES gene score, which measure the susceptibility potential"
- [intro] Feature set listed in intro: "we used another deep learning framework to integrate the ncDeepBrain score, general gene scores (such as GTEx), and disease-specific scores to prioritize mutated genes for mental disorders"
- [discussion] Closest-gene limitation: [evidence span withheld — anchor status: not_found]
- [results] GTEx feature derivation: "GTEx score of the variants with q-value less than or equal to 0.05 [32] for each of the 44 available tissues from the GTEx database"
