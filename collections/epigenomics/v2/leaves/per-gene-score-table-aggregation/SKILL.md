---
name: per-gene-score-table-aggregation
description: Use when interval-level ClinGen HI/TS dosage-sensitivity annotations
  (GRCh37/hg19 BED) must be mapped onto individual genes and reduced to binary per-gene
  dosage-sensitivity calls for LOF/GOF contexts, or when POSTRE per-SV results need
  a per-gene aggregated view.
when_to_use_negative:
- Interpreting SVs implicated in recessive phenotypes — POSTRE is oriented towards
  dominant disorders and its capacity for recessive phenotypes has not been specifically
  evaluated
- Genes or intervals lacking ClinGen HI/TS dosage-sensitivity annotation — the 0.85
  threshold and HI-as-TS proxy cannot be applied without scores
- Analysis requires only per-SV pathogenicity calls — the first POSTRE output table
  ('Results per SV and phenotype') already provides this without gene-level aggregation
edam_operation: http://edamontology.org/operation_2409
tools:
- name: POSTRE
  role: Shiny/R tool whose dosage-sensitivity scoring module (HI/TS tables + ClinGen
    BED) is reconstructed and whose second output table performs the per-gene, per-phenotype
    aggregation
  repo: https://github.com/vicsanga/Postre
- name: POSTRE_multiSV()
  role: R function used to run the multiple-SV analysis and obtain the aggregated
    per-gene/per-phenotype output tables
  repo: https://github.com/vicsanga/Postre
- name: R (>=3.5.0)
  role: Runtime environment required to source POSTRE_multiSV.R and perform the per-gene
    table assembly and thresholding
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:3905d4175e028405c95e696784765af936008cecc00551b7d7181df43fcca274
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
  - POSTRE
  - POSTRE_multiSV()
  - R
  - R (>=3.5.0)
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# per-gene-score-table-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Collapses POSTRE's per-SV and per-interval predictions into per-gene score tables: it maps ClinGen haploinsufficiency (HI) interval annotations (ClinGen_haploinsufficiency_gene_GRCh37.bed, GRCh37/hg19) onto genes, derives final_HI_score/final_TS_score, applies a 0.85 dosage-sensitivity threshold, and writes a gene-level TSV (dosage_sensitivity_scores.tsv) used by POSTRE_multiSV()'s 'Results per gene and phenotype' aggregation.

## When to use

Use when interval-level ClinGen HI/TS dosage-sensitivity annotations (GRCh37/hg19 BED) must be mapped onto individual genes and reduced to binary per-gene dosage-sensitivity calls for LOF/GOF contexts, or when POSTRE per-SV results need a per-gene aggregated view.

## When NOT to use

- Interpreting SVs implicated in recessive phenotypes — POSTRE is oriented towards dominant disorders and its capacity for recessive phenotypes has not been specifically evaluated
- Genes or intervals lacking ClinGen HI/TS dosage-sensitivity annotation — the 0.85 threshold and HI-as-TS proxy cannot be applied without scores
- Analysis requires only per-SV pathogenicity calls — the first POSTRE output table ('Results per SV and phenotype') already provides this without gene-level aggregation

## Inputs

- ClinGen HI gene score table (per-gene haploinsufficiency scores)
- ClinGen TS gene score table (per-gene triplosensitivity scores)
- ClinGen_haploinsufficiency_gene_GRCh37.bed interval file (hg19/GRCh37 coordinates) from the Postre_app folder
- SV data frame with 7 tab-separated columns (chromosome 1, coordinates 1, chromosome 2, coordinates 2, SV type, phenotypes, SV id) in hg19 coordinates

## Outputs

- dosage_sensitivity_scores.tsv with one row per gene and columns final_HI_score, final_TS_score, dosageSensitivityScore_LOF, dosageSensitivityScore_GOF
- POSTRE 'Results per gene and phenotype' table: pathogenic predictions aggregated per gene, phenotype and pathogenic mechanism (coding, long-range)

## How to apply

In R (>=3.5.0), load the ClinGen HI and TS gene score tables together with the ClinGen_haploinsufficiency_gene_GRCh37.bed interval file from the Postre_app folder, and map each interval-level GRCh37 (hg19) BED annotation onto the gene(s) it overlaps to assemble a per-gene HI/TS score table. Apply the 0.85 thresholding rule to classify genes as dosage-sensitive for the LOF context (final_HI_score >= 0.85) and the GOF context (final_TS_score >= 0.85); for X-chromosome (non-autosomal) genes lacking a TS score, substitute final_HI_score as the TS proxy (final_TS_score = final_HI_score) before thresholding. From the thresholded scores, derive binary dosageSensitivityScore_LOF and dosageSensitivityScore_GOF values per gene, and write the table (final_HI_score, final_TS_score, dosageSensitivityScore_LOF, dosageSensitivityScore_GOF) to dosage_sensitivity_scores.tsv. The same aggregation logic underlies POSTRE's second output table, which aggregates pathogenic predictions per gene, phenotype and pathogenic mechanism (coding, long-range). Run POSTRE_multiSV() without arguments to inspect the documented parameters and output schema before constructing the aggregation.

## Related tools

- **POSTRE** (Shiny/R tool whose dosage-sensitivity scoring module (HI/TS tables + ClinGen BED) is reconstructed and whose second output table performs the per-gene, per-phenotype aggregation) — https://github.com/vicsanga/Postre
- **POSTRE_multiSV()** (R function used to run the multiple-SV analysis and obtain the aggregated per-gene/per-phenotype output tables) — https://github.com/vicsanga/Postre
- **R (>=3.5.0)** (Runtime environment required to source POSTRE_multiSV.R and perform the per-gene table assembly and thresholding)

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); POSTRE_multiSV(SVs, pathTo_Postre_app_Folder)
```

## Evaluation signals

- dosage_sensitivity_scores.tsv contains exactly one row per gene with all four score columns (final_HI_score, final_TS_score, dosageSensitivityScore_LOF, dosageSensitivityScore_GOF) populated
- X-chromosome genes that have an HI score but lack a TS score carry the HI value as their TS proxy (final_TS_score == final_HI_score) before thresholding
- Binary dosageSensitivityScore_LOF/GOF equal 1 exactly when the corresponding final score is >= 0.85 and 0 otherwise
- Per-gene aggregated predictions match the counts implied by the per-SV/per-phenotype table when grouped by gene, phenotype and mechanism (coding, long-range)
- Gene-to-interval mapping respects hg19/GRCh37 coordinates as supplied in ClinGen_haploinsufficiency_gene_GRCh37.bed (no cross-build mismatches)

## Limitations

- Non-autosomal (sex) chromosome genes lack TS scores from the published dosage sensitivity maps, so the HI score is only a proxy for triplosensitivity in the GOF context
- Databases used to train SV classifiers are biased towards coding deleterious effects due to the scarcity of reported SVs with long-range pathogenic effects
- POSTRE's capacity to identify SVs implicated in recessive phenotypes has not been specifically evaluated; bulk whole-tissue genomic data may miss relevant enhancers (e.g. ZRS in whole limb buds)

## Evidence

- [other] Sex-chromosome HI-as-TS proxy (task_007 finding card): [evidence span withheld — anchor status: not_found]
- [methods] SV input format for POSTRE_multiSV(): [evidence span withheld — anchor status: not_found]
