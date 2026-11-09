---
name: multiple-testing-bonferroni-correction
description: Use when calling or powering differential expression / eQTL tests across
  many genes in scRNA-seq pseudobulk data, where a per-gene threshold must be corrected
  for the number of expressed genes tested.
when_to_use_negative:
- The article's own DE power workflow uses MTmethod='FDR' for the pseudobulk negative
  binomial power model — do not force Bonferroni when the analysis design specifies
  FDR control
- Single-hypothesis testing with no family of gene-level tests — no correction basis
  applies
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scPower
  role: Estimates the number of expressed genes that sets the correction basis and
    encodes multiple-testing correction (MTmethod) into its DE/eQTL power model
  repo: https://github.com/heiniglab/scPower
- name: edgeR
  role: Downstream pseudobulk DE testing whose per-gene p-values are thresholded with
    the Bonferroni-corrected cutoff
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:159d0d306b1fc3b26c9da1b400d0ce95d8e89d11f2708f758713f2ad6872c77f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - edgeR
  - muscat
  - powsimR
  - scPower
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# multiple-testing-bonferroni-correction

## Summary

Apply Bonferroni-style multiple-testing correction to gene-level significance testing in single cell pseudobulk DE and eQTL power analysis, using the estimated number of expressed genes as the effective number of tests.

## When to use

Use when calling or powering differential expression / eQTL tests across many genes in scRNA-seq pseudobulk data, where a per-gene threshold must be corrected for the number of expressed genes tested.

## When NOT to use

- The article's own DE power workflow uses MTmethod='FDR' for the pseudobulk negative binomial power model — do not force Bonferroni when the analysis design specifies FDR control
- Single-hypothesis testing with no family of gene-level tests — no correction basis applies

## Inputs

- Per-gene p-values or test statistics from pseudobulk DE testing (edgeR-LRT) or eQTL linear regression on log-transformed pseudobulk counts
- Estimated number of expressed genes from scPower expression probability estimation (min.counts=3, perc.indiv=0.5)

## Outputs

- Corrected significance threshold (sign.threshold / n.genes.expressed) or adjusted p-values
- Set of significant genes per design/grid point
- Empirical DE power and overall detection power per gene and experimental design

## How to apply

Estimate the number of expressed genes expected in the experiment (e.g. scPower's expression probability model with min.counts=3, perc.indiv=0.5) and use that count — not the raw gene count — as the number of tests. For Bonferroni, divide the significance threshold (sign.threshold=0.05) by the estimated number of expressed genes, or in the scPower power model pass the correction choice via the MTmethod argument. In simulation validation (powsimR/muscat vs scPower), apply the corrected threshold to edgeR-LRT pseudobulk DE results; for small-mean eQTL genes, call a gene significant in each of B=100 simulation repeats at the multiple-testing-corrected threshold to obtain empirical power. Rationale: scPower's analytic detection power (expression probability × DE/eQTL power) assumes a controlled false discovery burden, so the correction basis must match the number of genes actually detectable in the design.

## Related tools

- **scPower** (Estimates the number of expressed genes that sets the correction basis and encodes multiple-testing correction (MTmethod) into its DE/eQTL power model) — https://github.com/heiniglab/scPower
- **edgeR** (Downstream pseudobulk DE testing whose per-gene p-values are thresholded with the Bonferroni-corrected cutoff)

## Examples

```
bonf.thresh <- 0.05 / n.genes.expressed; sig.genes <- which(pvalues < bonf.thresh)
```

## Evaluation signals

- The correction basis equals the estimated number of expressed genes, not the total number of genes in the count matrix
- All adjusted thresholds/call rates keep the per-design false positive burden at the nominal 0.05 level
- Power values in output tables lie within [0,1] and simulation-derived power agrees with scPower analytic power within simulation uncertainty
- Comparison table contains expression probability, DE power, and overall detection power for every design (e.g. 4/8/16 samples × 200/1000/3000 cells) with the corrected threshold applied consistently

## Limitations

- Data-driven priors only allow reliable overall power assessment for sample sizes smaller than or roughly equal to the pilot data set; larger samples find new DEGs with lower effect sizes not captured by the corrected threshold computation
- The effective number of tests depends on expression cutoffs; best-practice guidelines remove 19–33% of lowly expressed genes (1–10 reads per million / 1–5 UMI counts), shifting the correction basis

## Evidence

- [other] applying a 0.05 significance threshold with Bonferroni correction based on the estimated number of expressed genes: [evidence span withheld — anchor status: not_found]
- [methods] Apply multiple testing correction with MTmethod='FDR' at sign.threshold=0.05 based on the estimated number of expressed genes: [evidence span withheld — anchor status: not_found]
- [other] fit a linear regression of log-transformed counts on genotype and call a gene significant at the multiple-testing-corrected threshold to obtain empirical power: [evidence span withheld — anchor status: not_found]
