---
name: multiple-testing-correction
description: Use when a differential-expression or regression analysis produces thousands
  of raw p-values (one per gene or TR) and hits must be called with a controlled false-discovery
  rate, e.g. adjusted p-value < 0.05.
when_to_use_negative:
- Only one or a small pre-specified number of hypotheses is tested, where the BH adjustment
  provides little benefit and raw p-values are interpretable
- Adjusted p-values are being computed per gene on subsets of features after filtering
  on effect size, which can distort the FDR calculation relative to the full test
  set
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Language in which limma/eBayes and p.adjust(BH) correction are run on the
    expression matrices
- name: limma
  role: Fits the resistant-vs-sensitive linear model with empirical-Bayes moderation
    whose raw p-values are then BH-adjusted
- name: RePhine
  role: Produces per-TR significance statistics (likelihood-ratio test uniP/multiP)
    that require multiple-testing awareness when calling drug response-related TRs
  repo: https://github.com/coexps/RePhine
- name: GEOquery
  role: Downloads the raw expression series matrices from GEO whose per-gene test
    statistics are corrected
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:eba87718acd51a0621d5257a24467807cb5a911ca04cbb2eaa8ca5a3ba0ce6c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - GEOquery
  - R
  - RePhine
  - limma
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# multiple-testing-correction

## Summary

Apply Benjamini-Hochberg false-discovery-rate correction to raw per-gene p-values from differential-expression analyses so that significance calls (e.g. EZH2/PRC2 down-regulation in BRAF inhibitor-resistant cells) control the expected proportion of false positives across thousands of simultaneous tests.

## When to use

Use when a differential-expression or regression analysis produces thousands of raw p-values (one per gene or TR) and hits must be called with a controlled false-discovery rate, e.g. adjusted p-value < 0.05.

## When NOT to use

- Only one or a small pre-specified number of hypotheses is tested, where the BH adjustment provides little benefit and raw p-values are interpretable
- Adjusted p-values are being computed per gene on subsets of features after filtering on effect size, which can distort the FDR calculation relative to the full test set

## Inputs

- Raw per-gene p-values from a limma/eBayes differential-expression table (resistant vs sensitive)
- Raw per-TR significance p-values from RePhine uniP likelihood-ratio tests
- Expression series matrices (GSE68840, GSE68599) with sample group assignments

## Outputs

- Differential-expression tables (de_results_gse68840.tsv, de_results_gse68599.tsv) with columns for log2 fold change, raw p-value, and Benjamini-Hochberg adjusted p-value
- Set of significant features meeting adjusted p-value < 0.05 with the expected effect direction

## How to apply

Fit the per-gene model first — e.g. a limma linear model of resistant vs sensitive samples with empirical-Bayes moderation (eBayes) for GSE68840 and GSE68599, or the likelihood-ratio-test uniP/multiP statistics in RePhine that score transcriptional regulators against drug response. Then convert raw p-values to Benjamini-Hochberg adjusted p-values across all tested features in the table. Report both the raw p-value and the BH-adjusted value for every gene/TR so downstream readers can re-threshold. Call a feature significant only when its adjusted p-value passes the chosen cutoff (0.05 in the EZH2/PRC2 validation) together with the effect-size sign, e.g. negative log2 fold change for down-regulation. Rationale: raw p-values at genome scale yield many false positives, so adjusted values are the basis for validating findings such as significant down-regulation of EZH2, EED, and SUZ12 in both independent datasets.

## Related tools

- **R** (Language in which limma/eBayes and p.adjust(BH) correction are run on the expression matrices)
- **limma** (Fits the resistant-vs-sensitive linear model with empirical-Bayes moderation whose raw p-values are then BH-adjusted)
- **RePhine** (Produces per-TR significance statistics (likelihood-ratio test uniP/multiP) that require multiple-testing awareness when calling drug response-related TRs) — https://github.com/coexps/RePhine
- **GEOquery** (Downloads the raw expression series matrices from GEO whose per-gene test statistics are corrected)

## Examples

```
fit <- eBayes(lmFit(expr, design)); tt <- topTable(fit, number=Inf); tt$adj.P.Val <- p.adjust(tt$P.Value, method="BH"); hits <- subset(tt, adj.P.Val < 0.05)
```

## Evaluation signals

- Each output table contains one adjusted p-value per gene equal to BH-corrected values computed over the full set of tested genes
- Adjusted p-values are monotonically consistent with raw p-values (larger raw p never yields smaller adjusted p within the same table)
- EZH2 and PRC2 subunit genes (e.g. EED, SUZ12) show negative log2 fold change with adjusted p-value < 0.05 in resistant cells in both de_results_gse68840.tsv and de_results_gse68599.tsv, matching Figure 5E
- No significant-call is made on raw p-values alone; every reported hit cites the adjusted value
- All adjusted p-values fall in [0,1] and the table schema (log2FC, p-value, adjusted p-value) is complete for every gene

## Limitations

- BH-adjusted significance does not establish causality between a transcriptional regulator and drug response; experimental validation (TR knockout/activation with drug response testing) is still required
- Correction cannot rescue tests with low power: noisy expression data or few samples per group can mask genuine EZH2/PRC2 down-regulation even after adjustment
- Adjusted p-values depend on the full feature set tested; changing the tested gene list changes the correction and thus the significance calls

## Evidence

- [methods] Extract log2 fold changes, raw p-values, and Benjamini-Hochberg adjusted p-values for every gene in each dataset.: [evidence span withheld — anchor status: not_found]
- [results] confirm that EZH2 and genes encoding PRC2 subunits (e.g. EED, SUZ12) show negative log2 fold change with adjusted p-value < 0.05 in resistant cells in both tables, reproducing the down-regulation pattern plotted in Figure 5E.: [evidence span withheld — anchor status: not_found]
- [methods] Fit a limma linear model (resistant vs sensitive) on the expression matrix of each dataset with empirical-Bayes moderation (eBayes).: [evidence span withheld — anchor status: not_found]
- [readme] Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations with drug response: "Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations with drug response"
- [discussion] TR target analyses and TR knockout or activation experiments combined with drug response examination are also required to validate the causal relationship: "TR target analyses and TR knockout or activation experiments combined with drug response examination are also required to validate the causal relationship"
