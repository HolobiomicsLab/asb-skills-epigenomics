---
name: fisher-exact-driver-enrichment-testing
description: Use when validating a CITRUS attention-weight output (attn_wt) by asking
  if highly attended genes are enriched for known cancer drivers (e.g., IntOGen),
  or when quantifying overlap/enrichment of two gene/TF sets via Fisher's exact test.
when_to_use_negative:
- The model output has not been trained/run — there is no 'attn_wt' entry in the output
  dictionary yet
- The question is about TF activities (hid_tmr) associated with a specific somatic
  mutation rather than enrichment of driver genes among highly attended genes
- One of the compared sets is too small for a 2x2 contingency table (expected counts
  near zero) where exact-test assumptions on marginals are invalid
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: CITRUS
  role: Source of the trained model's 'attn_wt' attention weights and gene list from
    the output pickle dictionary
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: SciPy
  role: scipy.stats.fisher_exact computes the odds ratio and p-value on the 2x2 driver-by-attention
    contingency table
- name: Pandas
  role: Loads and manipulates the pickle output and writes attention_driver_enrichment.csv
- name: Numpy
  role: Array operations for the log(attention + 1) transform and threshold classification
    of genes
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:bbf2639d9015ca1648e5211cf24dab3c938137640db2027cab609737857dcfda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/osmanbeyoglulab/CITRUS
  tools:
  - CITRUS
  - Numpy
  - Pandas
  - Python
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# fisher-exact-driver-enrichment-testing

## Summary

Tests whether genes receiving high attention weights in a trained CITRUS model are enriched for known cancer drivers from the IntOGen database by building a 2x2 contingency table (driver vs non-driver x high vs low attention) and running Fisher's exact test in SciPy. This validates that the model's self-attention layer concentrates on biologically relevant somatic alterations.

## When to use

Use when validating a CITRUS attention-weight output (attn_wt) by asking if highly attended genes are enriched for known cancer drivers (e.g., IntOGen), or when quantifying overlap/enrichment of two gene/TF sets via Fisher's exact test.

## When NOT to use

- The model output has not been trained/run — there is no 'attn_wt' entry in the output dictionary yet
- The question is about TF activities (hid_tmr) associated with a specific somatic mutation rather than enrichment of driver genes among highly attended genes
- One of the compared sets is too small for a 2x2 contingency table (expected counts near zero) where exact-test assumptions on marginals are invalid

## Inputs

- CITRUS output pickle file (output_dataset_CITRUS.pkl) containing the 'attn_wt' attention-weight array and gene list
- IntOGen known cancer driver gene list (pan-cancer)
- SGA binary somatic alteration matrix (11998 genes, 1 = mutation or copy number alteration) for ranking alteration frequency

## Outputs

- attention_driver_enrichment.csv containing the Fisher's exact test odds ratio, p-value, contingency table, and high/low-attention membership of the top 100 frequently mutated genes
- 2x2 contingency table of driver status by attention group

## How to apply

Load the CITRUS output pickle file (output_dataset_CITRUS.pkl) with pickle/pandas and extract the 'attn_wt' attention-weight array plus the gene list from the output dictionary. Transform each gene's attention weight as log(attention + 1) and classify genes with log(attention + 1) >= 2 into the high-attention group, with the remainder forming the low-attention group. Build a 2x2 contingency table of IntOGen known cancer driver status (driver vs non-driver) by attention group (high vs low) and run scipy.stats.fisher_exact; in the pan-cancer analysis of 5803 TCGA samples across 17 tumor types this yields P = 4.48e-41, confirming known drivers are enriched in the high-attention group. Optionally rank genes by somatic alteration frequency from the SGA binary matrix, take the top 100 most frequently mutated genes, label each by the same log(attention + 1) >= 2 threshold, and write the odds ratio, p-value, contingency table, and high/low memberships to attention_driver_enrichment.csv.

## Related tools

- **CITRUS** (Source of the trained model's 'attn_wt' attention weights and gene list from the output pickle dictionary) — https://github.com/osmanbeyoglulab/CITRUS
- **SciPy** (scipy.stats.fisher_exact computes the odds ratio and p-value on the 2x2 driver-by-attention contingency table)
- **Pandas** (Loads and manipulates the pickle output and writes attention_driver_enrichment.csv)
- **Numpy** (Array operations for the log(attention + 1) transform and threshold classification of genes)

## Examples

```
import pickle; from scipy.stats import fisher_exact; out = pickle.load(open('./data/output_dataset_CITRUS.pkl','rb')); attn = out['attn_wt']; odds, p = fisher_exact([[a,b],[c,d]])
```

## Evaluation signals

- The reproduced Fisher's exact test p-value matches the reported pan-cancer enrichment value of P = 4.48e-41
- Contingency table row sums equal the number of high-attention genes (log(attn+1) >= 2) and low-attention genes, and column sums equal IntOGen driver and non-driver gene counts
- Exactly 100 genes are labeled in the top-100 most frequently mutated list from the SGA binary matrix, each tagged high- or low-attention
- attention_driver_enrichment.csv contains all four fields: odds ratio, p-value, contingency table, and high/low memberships
- Threshold check: a gene with attention weight a belongs to the high group iff log(a + 1) >= 2 (i.e., a >= e^2 - 1)

## Limitations

- The log(attention + 1) >= 2 cutoff defining the high-attention group is a fixed threshold; results are sensitive to this choice
- Ranking by the top 100 most frequently mutated SGA genes is a heuristic subset and does not test enrichment over all 11998 SGA genes
- CITRUS attention weights are model-dependent (trained across 5803 TCGA samples, 17 tumor types, 10-run ensemble), so enrichment results depend on the training run and dataset preprocessing

## Evidence

- [readme] attention weight extraction from output dictionary: ""attn_wt": attn_wt,       # attention weight"
- [readme] loading the CITRUS output pickle: "output_data = pickle.load(open("./data/output_dataset_CITRUS.pkl", "rb"))"
- [readme] SGA binary matrix definition used for alteration-frequency ranking: "SGA is originally a binary matrix containing 11998 genes with 1 as having a distinct mutation or copy number alteration and 0 as none"
- [methods] SciPy availability for statistical testing: "some other packages such as PyTorch, Pandas, Numpy, scikit-learn, Scipy are used"
- [methods] pan-cancer cohort scale grounding the analysis: "They included 5803 samples with 17 different tumor types from TCGA."
