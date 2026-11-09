---
name: prc2-downregulation-verification
description: Use when validating a predicted transcriptional-regulator/drug-response
  association (e.g. EZH2/PRC2 loss reducing PLX4720 sensitivity) with independent
  expression datasets comparing drug-resistant versus drug-sensitive cells.
when_to_use_negative:
- When the question is inferring drug response-related transcriptional regulators
  de novo from pharmacogenomic and ChIP-seq data (use RePhine itself, not this validation
  skill)
- When only post-treatment or paired resistance-acquisition data are available rather
  than pre-existing resistant-vs-sensitive comparisons, e.g. acquired resistance scenarios
  the article notes CCLE cannot support due to lack of post-treatment data
- When reliable target/regulator annotation is unavailable or ChIP-seq target sets
  are too sparse, a limitation the article flags as misleading for TR identification
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: RePhine
  role: Source method whose EZH2/PRC2 loss-of-function prediction for PLX4720 response
    is being validated; R package available at https://github.com/coexps/RePhine
  repo: https://github.com/coexps/RePhine
- name: R
  role: Analysis environment in which GEOquery data retrieval and limma differential-expression
    fitting are executed
- name: limma
  role: Fits the resistant-vs-sensitive linear model on each expression matrix with
    empirical-Bayes moderation (eBayes) to produce log2FC, p-value, and BH-adjusted
    p-value per gene
- name: GEOquery
  role: Downloads the raw expression series matrices for GSE68840 and GSE68599 from
    GEO
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:0ed5726fc43573f064335d798fb3530260d831e36b8d32ade589b87956109599
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - GEOquery
  - R
  - RePhine
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# prc2-downregulation-verification

## Summary

Verifies the RePhine prediction that loss-of-function of EZH2/PRC2 confers BRAF inhibitor resistance by testing whether EZH2 and genes encoding PRC2 subunits (e.g. EED, SUZ12) are significantly down-regulated in BRAF inhibitor-resistant versus sensitive cells in the independent GEO expression datasets GSE68840 and GSE68599.

## When to use

Use when validating a predicted transcriptional-regulator/drug-response association (e.g. EZH2/PRC2 loss reducing PLX4720 sensitivity) with independent expression datasets comparing drug-resistant versus drug-sensitive cells.

## When NOT to use

- When the question is inferring drug response-related transcriptional regulators de novo from pharmacogenomic and ChIP-seq data (use RePhine itself, not this validation skill)
- When only post-treatment or paired resistance-acquisition data are available rather than pre-existing resistant-vs-sensitive comparisons, e.g. acquired resistance scenarios the article notes CCLE cannot support due to lack of post-treatment data
- When reliable target/regulator annotation is unavailable or ChIP-seq target sets are too sparse, a limitation the article flags as misleading for TR identification

## Inputs

- GEO series matrix files for GSE68840 (BRAF inhibitor-resistant vs sensitive expression data)
- GEO series matrix files for GSE68599 (independent replication expression dataset)
- Series metadata for sample group assignment (resistant vs sensitive)
- List of PRC2 subunit-encoding genes to inspect (EZH2, EED, SUZ12)

## Outputs

- de_results_gse68840.tsv: tab-separated differential-expression table (log2 fold change, raw p-value, BH-adjusted p-value per gene)
- de_results_gse68599.tsv: tab-separated differential-expression table (log2 fold change, raw p-value, BH-adjusted p-value per gene)
- Verification statement of EZH2/PRC2 down-regulation pattern consistent with Figure 5E

## How to apply

Download the raw series matrices for GSE68840 and GSE68599 from GEO with the GEOquery package in R, and assign samples to BRAF inhibitor-resistant versus sensitive groups according to each series' metadata. Fit a limma linear model (resistant vs sensitive) on each expression matrix with empirical-Bayes moderation (eBayes), and extract log2 fold changes, raw p-values, and Benjamini-Hochberg adjusted p-values for every gene. Write one tab-separated differential-expression table per dataset (de_results_gse68840.tsv, de_results_gse68599.tsv). The skill is successful when EZH2 and PRC2 subunit-encoding genes (e.g. EED, SUZ12) show negative log2 fold change with adjusted p-value < 0.05 in resistant cells in BOTH tables, reproducing the down-regulation pattern of Figure 5E and supporting the RePhine prediction that EZH2/PRC2 loss-of-function reduces sensitivity toward the BRAF inhibitor PLX4720.

## Related tools

- **RePhine** (Source method whose EZH2/PRC2 loss-of-function prediction for PLX4720 response is being validated; R package available at https://github.com/coexps/RePhine) — https://github.com/coexps/RePhine
- **R** (Analysis environment in which GEOquery data retrieval and limma differential-expression fitting are executed)
- **limma** (Fits the resistant-vs-sensitive linear model on each expression matrix with empirical-Bayes moderation (eBayes) to produce log2FC, p-value, and BH-adjusted p-value per gene)
- **GEOquery** (Downloads the raw expression series matrices for GSE68840 and GSE68599 from GEO)

## Examples

```
library(GEOquery); library(limma); m <- exprs(getGEO('GSE68840')[[1]]); fit <- eBayes(lmFit(m, model.matrix(~group))); write.table(topTable(fit, number=Inf, adjust='BH'), 'de_results_gse68840.tsv', sep='\t')
```

## Evaluation signals

- de_results_gse68840.tsv and de_results_gse68599.tsv exist and each contains log2FC, raw p-value, and BH-adjusted p-value columns for every gene
- EZH2 has negative log2 fold change in resistant vs sensitive cells in both datasets
- EZH2 and PRC2 subunit genes (EED, SUZ12) have Benjamini-Hochberg adjusted p-value < 0.05 in resistant cells in both tables
- Directionality is concordant between GSE68840 and GSE68599, reproducing the two-dataset down-regulation pattern of Figure 5E
- Sample counts and group assignments match the resistant/sensitive labels in each series' GEO metadata

## Limitations

- Down-regulation in resistant cells is a correlation consistent with, but not proof of, causality; the article notes 'the relationship may not be causal' and that 'TR target analyses and TR knockout or activation experiments combined with drug response examination are also required to validate the causal relationship'
- The verification covers EZH2/PRC2 as predicted for PLX4720 (BRAF inhibitor) response; it does not generalize the RePhine prediction to other drugs or acquired resistance mechanisms, e.g. acquired STAT3 activation in erlotinib resistance could not be detected
- Pharmacological EZH2 inhibition is reported to increase resistance to PLX4720, so EZH2 loss alone does not fully determine response magnitude

## Evidence

- [abstract] RePhine predicted that loss-of-function of EZH2/PRC2 reduces cancer cell sensitivity toward the BRAF inhibitor PLX4720.: "RePhine predicted that loss-of-function of EZH2/PRC2 reduces cancer cell sensitivity toward the BRAF inhibitor PLX4720."
- [results] EZH2 and genes encoding other PRC2 subunits were significantly down-regulated in BRAF inhibitor-resistant cells in two independent expression datasets, GSE68840 and GSE68599.: [evidence span withheld — anchor status: not_found]
- [methods] Fit a limma linear model (resistant vs sensitive) on the expression matrix of each dataset with empirical-Bayes moderation (eBayes).: [evidence span withheld — anchor status: not_found]
- [discussion] TR target analyses and TR knockout or activation experiments combined with drug response examination are also required to validate the causal relationship: "TR target analyses and TR knockout or activation experiments combined with drug response examination are also required to validate the causal relationship"
- [readme] The source code for RePhine is freely available at https://github.com/coexps/RePhine.: "The source code for RePhine is freely available at https://github.com/coexps/RePhine."
