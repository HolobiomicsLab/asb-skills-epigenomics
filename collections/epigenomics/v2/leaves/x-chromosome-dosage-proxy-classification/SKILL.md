---
name: x-chromosome-dosage-proxy-classification
description: Use when building or reimplementing per-gene dosage-sensitivity classifications
  from ClinGen HI/TS scores and you must score X/Y chromosome genes that have no TS
  entry in the published dosage sensitivity maps.
when_to_use_negative:
- Genes on autosomes already carry canonical ClinGen TS scores — do not apply the
  HI-as-TS proxy there, as it would override the true triplosensitivity score.
- The analysis goal is SV-level pathogenicity prediction rather than per-gene dosage-sensitivity
  annotation — use POSTRE_multiSV() results directly instead.
- The dosage sensitivity maps have been updated to include TS scores for sex chromosomes,
  making the proxy substitution obsolete.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: POSTRE_multiSV()
  role: POSTRE R function whose embedded dosageSensitivityScore module is being reconstructed;
    provides the HI/TS score logic and 0.85 thresholding for LOF/GOF contexts
  repo: https://github.com/vicsanga/Postre
- name: R
  role: Runtime environment (version >=3.5.0) in which the ClinGen score tables are
    loaded and the per-gene dosage-sensitivity table is assembled and thresholded
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:46bb3583c0b94f35406d894d4b4681a3f81c191bcc949741caaf9a5bb5fdad00
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# x-chromosome-dosage-proxy-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs POSTRE's dosageSensitivityScore module by assigning ClinGen haploinsufficiency (HI) and triplosensitivity (TS) scores per gene, thresholding at 0.85 for LOF/GOF contexts, and substituting final_HI_score as the TS proxy for non-autosomal (sex-chromosome) genes that lack published TS scores.

## When to use

Use when building or reimplementing per-gene dosage-sensitivity classifications from ClinGen HI/TS scores and you must score X/Y chromosome genes that have no TS entry in the published dosage sensitivity maps.

## When NOT to use

- Genes on autosomes already carry canonical ClinGen TS scores — do not apply the HI-as-TS proxy there, as it would override the true triplosensitivity score.
- The analysis goal is SV-level pathogenicity prediction rather than per-gene dosage-sensitivity annotation — use POSTRE_multiSV() results directly instead.
- The dosage sensitivity maps have been updated to include TS scores for sex chromosomes, making the proxy substitution obsolete.

## Inputs

- ClinGen haploinsufficiency (HI) gene score table
- ClinGen triplosensitivity (TS) gene score table
- ClinGen_haploinsufficiency_gene_GRCh37.bed interval file (GRCh37/hg19) from the Postre_app folder
- Gene coordinates/identifiers to map interval-level BED annotations onto individual genes

## Outputs

- dosage_sensitivity_scores.tsv with one row per gene and columns: final_HI_score, final_TS_score, dosageSensitivityScore_LOF, dosageSensitivityScore_GOF

## How to apply

In R (>=3.5.0), load the ClinGen HI and TS gene score tables together with the ClinGen_haploinsufficiency_gene_GRCh37.bed interval file from the Postre_app folder, then map the interval-level GRCh37 (hg19) BED annotations onto individual genes to assemble a per-gene HI/TS table. Apply the 0.85 thresholding rule: a gene is dosage-sensitive in the LOF context when final_HI_score >= 0.85 and in the GOF context when final_TS_score >= 0.85. Before thresholding, for genes on non-autosomal (sex) chromosomes that lack a TS score, substitute the HI score as the TS proxy, i.e. set final_TS_score = final_HI_score, so that X-chromosome HI genes receive a meaningful GOF classification. Derive the binary dosageSensitivityScore_LOF and dosageSensitivityScore_GOF values per gene from the thresholded scores and write the four-column per-gene table to dosage_sensitivity_scores.tsv. This mirrors the logic embedded in POSTRE_multiSV() so that downstream SV pathogenicity predictions handle sex-chromosome genes consistently.

## Related tools

- **POSTRE_multiSV()** (POSTRE R function whose embedded dosageSensitivityScore module is being reconstructed; provides the HI/TS score logic and 0.85 thresholding for LOF/GOF contexts) — https://github.com/vicsanga/Postre
- **R** (Runtime environment (version >=3.5.0) in which the ClinGen score tables are loaded and the per-gene dosage-sensitivity table is assembled and thresholded)

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); scores <- assemble_dosage_scores(ClinGen_HI, ClinGen_TS, "ClinGen_haploinsufficiency_gene_GRCh37.bed"); scores$final_TS_score[is.na(scores$final_TS_score)] <- scores$final_HI_score[is.na(scores$final_TS_score])
```

## Evaluation signals

- dosage_sensitivity_scores.tsv contains exactly one row per gene with all four columns (final_HI_score, final_TS_score, dosageSensitivityScore_LOF, dosageSensitivityScore_GOF) populated
- X-chromosome HI genes receive the HI-as-TS proxy value, i.e. final_TS_score equals final_HI_score for sex-chromosome genes lacking published TS scores
- dosageSensitivityScore_LOF is binary and equals 1 exactly when final_HI_score >= 0.85
- dosageSensitivityScore_GOF is binary and equals 1 exactly when final_TS_score >= 0.85
- For autosomal genes with canonical ClinGen TS scores, the reconstructed final_TS_score matches the published ClinGen TS value

## Limitations

- The HI-as-TS proxy is an approximation: for sex-chromosome genes the GOF classification inherits the HI (loss-of-function) evidence, which may not reflect true gain-of-function dosage sensitivity.
- Scores depend on GRCh37 (hg19) interval annotations from ClinGen_haploinsufficiency_gene_GRCh37.bed; interval-to-gene mapping errors propagate into the per-gene scores.
- POSTRE is oriented towards dominant disorders, and its dosage-sensitivity scoring was designed for that context, not for recessive phenotype prediction.

## Evidence

- [methods] sex chromosome proxy rule: [evidence span withheld — anchor status: not_found]
- [methods] 0.85 threshold rule: [evidence span withheld — anchor status: not_found]
- [readme] R runtime requirement: [evidence span withheld — anchor status: not_found]
