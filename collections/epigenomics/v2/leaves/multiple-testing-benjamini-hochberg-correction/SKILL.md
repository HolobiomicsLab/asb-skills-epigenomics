---
name: multiple-testing-benjamini-hochberg-correction
description: Use when a large family of one-sided Fisher's exact tests (e.g., clusterProfiler::enrichGO
  GO term enrichment of BIT top-ranked TRs) must be corrected for multiple testing,
  with terms ranked by BH-adjusted p values at a q-value cutoff of 0.05.
when_to_use_negative:
- The quantity of interest is a BIT score with a Bayesian 95% credible interval from
  posterior sampling — uncertainty is already quantified by MCMC posterior draws,
  so frequentist BH adjustment is not the appropriate mechanism for TR ranking itself
- Only a single pre-specified hypothesis test is performed (no multiple-testing family),
  where an unadjusted p value suffices
- Input is raw ATAC-seq data or peak calls (e.g., MACS2 -q 0.01 peaks) rather than
  ranked TR lists requiring post-hoc GO enrichment correction
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: clusterProfiler
  role: Runs over-representation analysis (enrichGO) with pAdjustMethod = 'BH' and
    qvalueCutoff = 0.05 to produce BH-adjusted p values for GO terms
- name: BIT
  role: Produces the TR rank table whose top-ranked TRs are the gene sets tested;
    BH correction is applied downstream of BIT scores
  repo: https://github.com/ZeyuL01/BIT
- name: R
  role: Language in which clusterProfiler::enrichGO and the org.Mm.eg.db / org.Hs.eg.db
    annotation workflow are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:59dfdfff51c644c274e61bcc6935952e1c5bac13abb00f557a305ec6992efeea
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
  - BIT
  - R
  - clusterProfiler
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# multiple-testing-benjamini-hochberg-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies Benjamini-Hochberg (BH) adjustment to control the false discovery rate when ranking statistically significant results from many simultaneous one-sided Fisher's exact tests, as done in the BIT paper's GO enrichment analysis of top-ranked transcriptional regulators (TRs).

## When to use

Use when a large family of one-sided Fisher's exact tests (e.g., clusterProfiler::enrichGO GO term enrichment of BIT top-ranked TRs) must be corrected for multiple testing, with terms ranked by BH-adjusted p values at a q-value cutoff of 0.05.

## When NOT to use

- The quantity of interest is a BIT score with a Bayesian 95% credible interval from posterior sampling — uncertainty is already quantified by MCMC posterior draws, so frequentist BH adjustment is not the appropriate mechanism for TR ranking itself
- Only a single pre-specified hypothesis test is performed (no multiple-testing family), where an unadjusted p value suffices
- Input is raw ATAC-seq data or peak calls (e.g., MACS2 -q 0.01 peaks) rather than ranked TR lists requiring post-hoc GO enrichment correction

## Inputs

- BIT output rank table (e.g., mm10_RUNX1_rank_table.csv) with TR gene symbols and BIT scores
- Top-ranked TR gene symbol list (top 10 for perturbation experiments, top 20 for cancer-type/cell-type cases)
- Entrez gene identifier mappings for TR symbols
- Organism annotation background: org.Mm.eg.db (mm10) or org.Hs.eg.db (hg38)

## Outputs

- GO_enrichment_results.csv containing GO term IDs with BH-adjusted p-value and q-value columns, ranked by adjusted p value
- Summary bar/dot plot of the most significant GO terms for each analyzed TR set

## How to apply

In the BIT workflow, the top-ranked TR gene symbols (top 10 for perturbation experiments such as RUNX1 knockout and K562 CRISPR perturbations; top 20 for cancer-type and cell-type-specific cases) are mapped to Entrez IDs and tested for over-representation with clusterProfiler::enrichGO against the organism-specific background (org.Mm.eg.db for mm10, org.Hs.eg.db for hg38). The BH procedure is invoked by setting pAdjustMethod = 'BH' and qvalueCutoff = 0.05 in enrichGO, which converts the per-term one-sided Fisher's exact p values into FDR-controlled adjusted p values. Enriched GO terms are then ranked by their BH-adjusted p values, and only terms with adjusted q-values <= 0.05 are retained. The rationale is that GO enrichment tests hundreds of terms simultaneously, so unadjusted p values would inflate false positives; the article contrasts this frequentist BH ranking with BIT's own Bayesian 95% credible intervals as the complementary uncertainty framework for TR scores themselves.

## Related tools

- **clusterProfiler** (Runs over-representation analysis (enrichGO) with pAdjustMethod = 'BH' and qvalueCutoff = 0.05 to produce BH-adjusted p values for GO terms)
- **BIT** (Produces the TR rank table whose top-ranked TRs are the gene sets tested; BH correction is applied downstream of BIT scores) — https://github.com/ZeyuL01/BIT
- **R** (Language in which clusterProfiler::enrichGO and the org.Mm.eg.db / org.Hs.eg.db annotation workflow are executed)

## Examples

```
ego <- clusterProfiler::enrichGO(gene = tr_entrez_ids, OrgDb = org.Mm.eg.db, pAdjustMethod = "BH", qvalueCutoff = 0.05)
```

## Evaluation signals

- The output table exists and contains GO term IDs with adjusted p-value and q-value columns
- All reported GO terms have BH-adjusted q-values <= 0.05, matching the analysis cutoff
- Enriched GO terms are ordered by ascending Benjamini-Hochberg adjusted p value (e.g., sister chromatid cohesion formation top-ranked for CTCF depletion; erythrocyte differentiation and homeostasis for ZBTB7A knockout)
- The enrichGO call sets pAdjustMethod = 'BH' and the correct organism background (org.Mm.eg.db for mm10, org.Hs.eg.db for hg38)
- The number of input gene symbols matches the protocol (top 10 for perturbation experiments, top 20 for cancer-type/cell-type-specific analyses)

## Limitations

- BH-adjusted p values from the one-sided Fisher's exact test provide no coherent probabilistic interpretation or uncertainty estimation, unlike BIT's Bayesian credible intervals — the two frameworks serve complementary roles and should not be conflated
- Enrichment results depend on the correctness of the organism background database; mixing up org.Mm.eg.db and org.Hs.eg.db for a genome build silently invalidates the adjusted p values
- The qvalueCutoff = 0.05 threshold can exclude all terms for weakly enriched TR sets, and BH control addresses only FDR across GO terms, not biases in the upstream peak-calling or TR ranking

## Evidence
