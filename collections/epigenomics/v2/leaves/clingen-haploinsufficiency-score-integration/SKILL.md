---
name: clingen-haploinsufficiency-score-integration
description: Use when replicating POSTRE's dosage-sensitivity scoring for structural
  variant pathogenicity prediction, especially when genes on sex chromosomes lack
  triplosensitivity (TS) scores and a proxy must be substituted before thresholding.
when_to_use_negative:
- The SVs or annotations are not on GRCh37/hg19 coordinates — POSTRE_multiSV() explicitly
  requires hg19 and remapping would invalidate the BED interval-to-gene mapping
- The goal is predicting SVs implicated in recessive phenotypes — the article states
  POSTRE is oriented towards dominant disorders and its recessive capacity has not
  been specifically evaluated
- A gene already has a published TS score — the HI-as-TS proxy substitution is only
  for genes on sex chromosomes lacking TS scores and must not override existing scores
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: POSTRE_multiSV()
  role: R function into which the reconstructed dosageSensitivityScore module feeds;
    consumes the per-gene HI/TS thresholded scores when predicting SV pathogenicity
  repo: https://github.com/vicsanga/Postre
- name: R (>=3.5.0)
  role: Runtime environment in which the ClinGen HI/TS tables and GRCh37 BED intervals
    are loaded, mapped, and thresholded
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:6380dcd3557832be2a826e8066e4c4be7bc64dcda34e4a7149d279cb0c8a2240
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/vicsanga/Postre
  repo_url: https://github.com/vicsanga/Postre
  tools:
  - POSTRE_multiSV()
  - R
  - R (>=3.5.0)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# clingen-haploinsufficiency-score-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs POSTRE's per-gene dosageSensitivityScore module by integrating ClinGen haploinsufficiency (HI) and triplosensitivity (TS) scores with the ClinGen_haploinsufficiency_gene_GRCh37.bed interval annotations, applying a 0.85 threshold to derive binary LOF/GOF dosage-sensitivity calls. Sex-chromosome genes lacking published TS scores receive the HI score as a TS proxy (final_TS_score = final_HI_score).

## When to use

Use when replicating POSTRE's dosage-sensitivity scoring for structural variant pathogenicity prediction, especially when genes on sex chromosomes lack triplosensitivity (TS) scores and a proxy must be substituted before thresholding.

## When NOT to use

- The SVs or annotations are not on GRCh37/hg19 coordinates — POSTRE_multiSV() explicitly requires hg19 and remapping would invalidate the BED interval-to-gene mapping
- The goal is predicting SVs implicated in recessive phenotypes — the article states POSTRE is oriented towards dominant disorders and its recessive capacity has not been specifically evaluated
- A gene already has a published TS score — the HI-as-TS proxy substitution is only for genes on sex chromosomes lacking TS scores and must not override existing scores

## Inputs

- ClinGen haploinsufficiency (HI) gene score table
- ClinGen triplosensitivity (TS) gene score table
- ClinGen_haploinsufficiency_gene_GRCh37.bed interval file (hg19) from the Postre_app folder
- Optionally, a 7-column tab-separated SV file (chromosome 1, coordinates 1, chromosome 2, coordinates 2, SV type, phenotypes, SV id) in hg19 coordinates for downstream POSTRE_multiSV() analysis

## Outputs

- dosage_sensitivity_scores.tsv — one row per gene with columns final_HI_score, final_TS_score, dosageSensitivityScore_LOF, dosageSensitivityScore_GOF
- Per-gene binary dosageSensitivityScore_LOF and dosageSensitivityScore_GOF values derived via the 0.85 threshold

## How to apply

In R (>=3.5.0), load the ClinGen HI and TS gene score tables together with the ClinGen_haploinsufficiency_gene_GRCh37.bed interval file from the Postre_app folder, and map the interval-level GRCh37 (hg19) BED annotations onto individual genes to assemble a per-gene HI/TS score table. Apply the 0.85 thresholding rule to classify genes as dosage-sensitive: final_HI_score >= 0.85 for the LOF context and final_TS_score >= 0.85 for the GOF context. For X-chromosome (and other non-autosomal) genes lacking a TS score, substitute the HI score as the TS proxy before thresholding, since the published dosage sensitivity maps do not cover sex chromosomes. Derive the binary dosageSensitivityScore_LOF and dosageSensitivityScore_GOF values per gene from the thresholded final_HI_score and final_TS_score, and write the four-column per-gene table to dosage_sensitivity_scores.tsv. This module feeds POSTRE_multiSV(), which expects SVs as a data frame with 7 columns of hg19 coordinates and integrates the dosage sensitivity calls into its pathogenicity predictions.

## Related tools

- **POSTRE_multiSV()** (R function into which the reconstructed dosageSensitivityScore module feeds; consumes the per-gene HI/TS thresholded scores when predicting SV pathogenicity) — https://github.com/vicsanga/Postre
- **R (>=3.5.0)** (Runtime environment in which the ClinGen HI/TS tables and GRCh37 BED intervals are loaded, mapped, and thresholded)

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); POSTRE_multiSV(SVs, pathTo_Postre_app_Folder)
```

## Evaluation signals

- dosage_sensitivity_scores.tsv contains exactly one row per gene with all four score columns (final_HI_score, final_TS_score, dosageSensitivityScore_LOF, dosageSensitivityScore_GOF) populated and non-missing
- Every X-chromosome gene classified as HI (final_HI_score >= 0.85) carries final_TS_score identical to final_HI_score, confirming the HI-as-TS proxy was applied
- All dosageSensitivityScore_LOF and dosageSensitivityScore_GOF values are binary and consistent with the 0.85 threshold applied to final_HI_score and final_TS_score respectively
- No gene with a published ClinGen TS score has its final_TS_score overwritten by the HI proxy
- The BED interval-to-gene mapping covers all genes present in the ClinGen score tables, with no gene silently dropped during GRCh37 interval mapping

## Limitations

- Genes on sex chromosomes lack TS scores from the published dosage sensitivity maps, so the HI score is used as a TS proxy — an approximation rather than a measured triplosensitivity value
- Bulk or context-inappropriate data can affect gene-level calls, as POSTRE's broader predictions can fail when cell type or developmental stage data are missing
- The ClinGen dosage sensitivity maps are themselves incomplete, so thresholding at 0.85 may misclassify genes with insufficient evidence

## Evidence

- [other] For genes on non-autosomal (sex) chromosomes, which lack TS scores, POSTRE uses the final_HI_score as a proxy, i.e. final_TS_score = final_HI_score.: [evidence span withheld — anchor status: not_found]
- [other] Apply the 0.85 thresholding rule to classify genes as dosage-sensitive for the LOF context (final_HI_score >= 0.85) and the GOF context (final_TS_score >= 0.85).: [evidence span withheld — anchor status: not_found]
- [readme] Thus, to run POSTRE you only require R (version >=3.5.0).: "Thus, to run POSTRE you only require R (version >=3.5.0)."
