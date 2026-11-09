---
name: gene-dosage-sensitivity-scoring
description: Use when annotating genes with binary LOF/GOF dosage-sensitivity scores
  for SV pathogenicity prediction, especially when sex-chromosome genes lack ClinGen
  TS scores and need an HI-based proxy.
when_to_use_negative:
- When the goal is predicting long-range (enhancer-driven) pathomechanisms — dosage
  sensitivity scoring only informs coding (gene deletion/duplication) events; non-coding
  mechanisms like enhancer disconnection rely on TAD/enhancer data instead
- When recessive-phenotype SV interpretation is the target — POSTRE is oriented towards
  dominant disorders and its capacity for recessive phenotypes has not been specifically
  evaluated
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: POSTRE_multiSV()
  role: R function whose dosageSensitivityScore module is being reconstructed; consumes
    the per-gene HI/TS table for LOF/GOF coding-effect prediction
  repo: https://github.com/vicsanga/Postre
- name: R (>=3.5.0)
  role: Runtime environment in which the ClinGen HI/TS tables and GRCh37 BED intervals
    are loaded and thresholded
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:ccb1cbe838ea92f6014519dc8a65746d37e82067836688db31725b4711abae4b
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# gene-dosage-sensitivity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct POSTRE's dosageSensitivityScore module by mapping ClinGen haploinsufficiency (HI) and triplosensitivity (TS) gene scores onto genes via GRCh37 BED intervals, applying the 0.85 dosage-sensitivity threshold, and substituting the HI score as a TS proxy for sex-chromosome genes that lack TS scores.

## When to use

Use when annotating genes with binary LOF/GOF dosage-sensitivity scores for SV pathogenicity prediction, especially when sex-chromosome genes lack ClinGen TS scores and need an HI-based proxy.

## When NOT to use

- When the goal is predicting long-range (enhancer-driven) pathomechanisms — dosage sensitivity scoring only informs coding (gene deletion/duplication) events; non-coding mechanisms like enhancer disconnection rely on TAD/enhancer data instead
- When recessive-phenotype SV interpretation is the target — POSTRE is oriented towards dominant disorders and its capacity for recessive phenotypes has not been specifically evaluated

## Inputs

- ClinGen HI gene score table
- ClinGen TS gene score table
- ClinGen_haploinsufficiency_gene_GRCh37.bed interval file (hg19) from the Postre_app folder
- Per-gene annotation (gene symbols/coordinates) to map BED intervals onto genes

## Outputs

- dosage_sensitivity_scores.tsv with one row per gene and four columns: final_HI_score, final_TS_score, dosageSensitivityScore_LOF, dosageSensitivityScore_GOF

## How to apply

In R (>=3.5.0), load the ClinGen HI and TS gene score tables together with the ClinGen_haploinsufficiency_gene_GRCh37.bed interval file from the Postre_app folder, and map interval-level GRCh37 (hg19) BED annotations onto individual genes to assemble a per-gene HI/TS score table. Apply the 0.85 thresholding rule to classify dosage-sensitive genes for the LOF context (final_HI_score >= 0.85) and the GOF context (final_TS_score >= 0.85). For X-chromosome (non-autosomal) genes lacking a TS score, substitute final_HI_score as the TS proxy (final_TS_score = final_HI_score) before thresholding, since published dosage sensitivity maps do not cover sex chromosomes. Derive the binary dosageSensitivityScore_LOF and dosageSensitivityScore_GOF per gene from the thresholded final_HI_score and final_TS_score. Write the per-gene table (final_HI_score, final_TS_score, dosageSensitivityScore_LOF, dosageSensitivityScore_GOF) to dosage_sensitivity_scores.tsv; this module feeds POSTRE_multiSV()'s coding pathomechanism predictions (gene deletion vs. duplication contexts).

## Related tools

- **POSTRE_multiSV()** (R function whose dosageSensitivityScore module is being reconstructed; consumes the per-gene HI/TS table for LOF/GOF coding-effect prediction) — https://github.com/vicsanga/Postre
- **R (>=3.5.0)** (Runtime environment in which the ClinGen HI/TS tables and GRCh37 BED intervals are loaded and thresholded)

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R")
```

## Evaluation signals

- dosage_sensitivity_scores.tsv contains exactly one row per gene with all four columns (final_HI_score, final_TS_score, dosageSensitivityScore_LOF, dosageSensitivityScore_GOF) populated
- Every gene with final_HI_score >= 0.85 has dosageSensitivityScore_LOF set to the dosage-sensitive (binary positive) value; genes below 0.85 are negative
- Every gene with final_TS_score >= 0.85 has dosageSensitivityScore_GOF set to the dosage-sensitive value
- X-chromosome HI genes receive the HI-as-TS proxy: their final_TS_score equals their final_HI_score
- HI/TS scores match the ClinGen source tables after interval-to-gene mapping from the GRCh37 (hg19) BED file

## Limitations

- Sex chromosomes lack TS scores in the published dosage sensitivity maps, so the HI score is used as a TS proxy for non-autosomal genes — an approximation, not a measured triplosensitivity value
- Only genes with ClinGen HI/TS annotations (recoverable via the GRCh37 BED intervals) receive dosage sensitivity scores; unannotated genes cannot be classified
- The 0.85 cutoff is a fixed thresholding rule inherited from the module; it does not capture partial dosage sensitivity below the cutoff

## Evidence

- [methods] For genes on non-autosomal (sex) chromosomes, which lack TS scores, POSTRE uses the final_HI_score as a proxy, i.e. final_TS_score = final_HI_score.: [evidence span withheld — anchor status: not_found]
