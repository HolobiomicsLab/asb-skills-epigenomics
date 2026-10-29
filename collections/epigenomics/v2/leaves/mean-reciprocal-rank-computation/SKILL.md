---
name: mean-reciprocal-rank-computation
description: Use when comparing ranked TR output from two or more TR-identification
  methods against a known ground-truth set of perturbed TRs (e.g., CTCF, ZBTB7A, FOXA2,
  RUNX1, BCL11B) and a single ranking-based metric such as MRR is needed, as in reproducing
  Fig. 3f of the BIT paper.
when_to_use_negative:
- When there is no ground-truth perturbed TR to recover — MRR is undefined without
  a known target TR in the ranked list
- When the perturbed TR is absent from a method's ranked output (rank cannot be extracted,
  biasing the comparison)
- When the comparison goal is per-TR statistical significance or uncertainty (e.g.,
  BIT's 95% credible intervals) rather than aggregate ranking quality
tools:
- name: BIT
  role: Produces ranked TR tables (BIT_score, Rank columns) from DAR BED inputs; the
    method expected to achieve highest MRR
  repo: https://github.com/ZeyuL01/BIT
- name: BART
  role: ChIP-seq binding-based comparison method run on the same DAR inputs; its ranked
    TR output supplies ranks for MRR
  repo: https://github.com/zanglab/bart2
- name: ChIP-Atlas
  role: ChIP-seq binding-based comparison method ranking TRs by Fisher's exact test
    p-values
- name: i-cisTarget
  role: Motif-based comparison method using ENCODE references and recovery analysis
- name: WhichTF
  role: Motif-based comparison method using JASPAR/TRANSFAC references and hypergeometric
    test
- name: HOMER
  role: Motif-based comparison method using JASPAR/TRANSFAC references
- name: R
  role: Used to compute reciprocal ranks, average them into per-method MRR, write
    mrr_benchmark.csv, and plot the Fig. 3f-style bar chart
  repo: https://github.com/ZeyuL01/BIT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:b60396ce802b204c5ecc90dbac7dff2e9ab5265d39447e6c6846f469a49578a2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZeyuL01/BIT
  repo_url: https://github.com/ZeyuL01/BIT
  tools:
  - BART
  - BIT
  - ChIP-Atlas
  - HOMER
  - I-cisTarget
  - R
  - WhichTF
  - i-cisTarget
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mean-reciprocal-rank-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes mean reciprocal rank (MRR) to benchmark transcriptional regulator (TR) identification methods — BIT vs. BART, ChIP-Atlas, i-cisTarget, WhichTF, and HOMER — on perturbed-TR DAR query sets, quantifying how highly each method ranks the known perturbed TR.

## When to use

Use when comparing ranked TR output from two or more TR-identification methods against a known ground-truth set of perturbed TRs (e.g., CTCF, ZBTB7A, FOXA2, RUNX1, BCL11B) and a single ranking-based metric such as MRR is needed, as in reproducing Fig. 3f of the BIT paper.

## When NOT to use

- When there is no ground-truth perturbed TR to recover — MRR is undefined without a known target TR in the ranked list
- When the perturbed TR is absent from a method's ranked output (rank cannot be extracted, biasing the comparison)
- When the comparison goal is per-TR statistical significance or uncertainty (e.g., BIT's 95% credible intervals) rather than aggregate ranking quality

## Inputs

- Differentially accessible region BED files for CTCF, ZBTB7A, FOXA2, RUNX1, and BCL11B perturbation datasets (bulk)
- Ranked TR output tables per method containing BIT_score and Rank columns (BIT) or equivalent ranked TR lists (BART, ChIP-Atlas, i-cisTarget, WhichTF, HOMER)
- List of ground-truth perturbed TR names per dataset

## Outputs

- mrr_benchmark.csv with one MRR value per method (mean of 1/rank over five perturbed TRs)
- Bar chart of per-method MRR in the style of Fig. 3f

## How to apply

For each method, run it on the same differentially accessible region BED inputs (derived from bulk TR perturbation datasets via DiffBind report generation and export.bed, e.g., the RUNX1 cKO workflow with dba.blacklist MM10 removal and MACS2 q=0.01 peaks) and extract the rank of the perturbed TR from the method's ranked TR output (BIT tables contain BIT_score and Rank columns). Compute the reciprocal rank 1/rank for each of the five perturbed TRs per method, then average over the five TRs to obtain the per-method MRR. Write per-method MRR values to mrr_benchmark.csv and visualize as a bar chart in the style of Fig. 3f. Validation: the computed MRRs should reproduce Fig. 3f, where BIT attains the highest MRR among the six methods.

## Related tools

- **BIT** (Produces ranked TR tables (BIT_score, Rank columns) from DAR BED inputs; the method expected to achieve highest MRR) — https://github.com/ZeyuL01/BIT
- **BART** (ChIP-seq binding-based comparison method run on the same DAR inputs; its ranked TR output supplies ranks for MRR) — https://github.com/zanglab/bart2
- **ChIP-Atlas** (ChIP-seq binding-based comparison method ranking TRs by Fisher's exact test p-values)
- **i-cisTarget** (Motif-based comparison method using ENCODE references and recovery analysis)
- **WhichTF** (Motif-based comparison method using JASPAR/TRANSFAC references and hypergeometric test)
- **HOMER** (Motif-based comparison method using JASPAR/TRANSFAC references)
- **R** (Used to compute reciprocal ranks, average them into per-method MRR, write mrr_benchmark.csv, and plot the Fig. 3f-style bar chart) — https://github.com/ZeyuL01/BIT

## Examples

```
mrr <- mean(1 / subset(rank_table, TR %in% c('CTCF','ZBTB7A','FOXA2','RUNX1','BCL11B'))$Rank)
```

## Evaluation signals

- Each per-method MRR equals the mean of five reciprocal ranks and therefore lies in (0, 1]
- The computed MRR values reproduce the reported Fig. 3f result in which BIT attains the highest MRR among the six methods
- Per-method MRR values are individually checkable against the published figure
- The perturbed TR's rank is successfully extracted from every method's ranked TR output (no missing ranks)
- mrr_benchmark.csv contains exactly one MRR row per method (six methods: BIT, BART, ChIP-Atlas, i-cisTarget, WhichTF, HOMER)

## Limitations

- Validation depends on visual comparison against Fig. 3f; supplementary comparison figures (e.g., Supplementary Figs. 9, 10a-c referenced in the discussion) are not present in the provided text, limiting cross-checks
- MRR rewards only the rank of the single known perturbed TR and does not capture uncertainty; BIT's Bayesian 95% credible intervals provide complementary information not reflected in MRR
- BIT's performance can be reduced by biological complexities of TR regulation, sequencing biases (GC content, PCR amplification), and the growing parameter count with expanding ChIP-seq reference libraries

## Evidence

- [readme] BART region-mode invocation on a BED query: [evidence span withheld — anchor status: not_found]
