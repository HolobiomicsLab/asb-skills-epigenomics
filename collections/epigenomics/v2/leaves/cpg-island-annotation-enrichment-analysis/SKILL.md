---
name: cpg-island-annotation-enrichment-analysis
description: Use when interpreting top-value-method-selected CpG probes from an Illumina
  450K methylation matrix (e.g. cola SD vs ATC top probes) by CpG-island association
  category, to judge whether features capture local or global methylation patterns.
when_to_use_negative:
- The input is expression or single-cell RNA-Seq data (e.g. Golub leukemia, HSMM,
  recount2 datasets) where rows are genes rather than CpG probes — CpG-island categories
  do not apply
- Probes have not been filtered and preprocessed first (sex-chromosome/SNP-overlapping
  probes removed, low-variance rows dropped), since unfiltered probe sets confound
  the enrichment comparison
- You need subtype classification itself rather than interpretation of which features
  were selected — use cola's run_all_consensus_partitions() instead
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3311
tools:
- name: cola
  role: Provides the top-value methods (SD, ATC) whose top-5000 probe sets are annotated
    with CpG-island categories, and generates the HTML report containing the comparison
  repo: https://github.com/jokergoo/cola_manuscript
- name: R
  role: Language in which the 450K annotation lookup, probe-set intersection and fraction
    computation are performed
- name: Bioconductor
  role: Source of Illumina 450K probe annotation resources for mapping probes to CpG-island
    association categories
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:03ac698f996238c66a5fe3510e1b5f4a8729757c05ff6b8f1df06bd5fc77e59f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jokergoo/cola_manuscript
  repo_url: https://github.com/jokergoo/cola_manuscript
  tools:
  - Bioconductor
  - R
  - cola
  - impute
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3311
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cpg-island-annotation-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates top-ranked CpG probes from methylation array feature selection (e.g. cola SD- vs ATC-selected probes on the Illumina 450K GBM dataset GSE36278) with CpG-island association categories and compares enrichment fractions to interpret whether selected features reflect local CpG-island structure or global methylation changes.

## When to use

Use when interpreting top-value-method-selected CpG probes from an Illumina 450K methylation matrix (e.g. cola SD vs ATC top probes) by CpG-island association category, to judge whether features capture local or global methylation patterns.

## When NOT to use

- The input is expression or single-cell RNA-Seq data (e.g. Golub leukemia, HSMM, recount2 datasets) where rows are genes rather than CpG probes — CpG-island categories do not apply
- Probes have not been filtered and preprocessed first (sex-chromosome/SNP-overlapping probes removed, low-variance rows dropped), since unfiltered probe sets confound the enrichment comparison
- You need subtype classification itself rather than interpretation of which features were selected — use cola's run_all_consensus_partitions() instead

## Inputs

- Two ranked probe lists (e.g. top 5000 probes by SD score and top 5000 by ATC score) from cola consensus partitioning on the GBM 450K methylation matrix (GSE36278)
- Illumina 450K probe annotation mapping each CpG probe to its CpG-island association category

## Outputs

- Fraction of CpG-island-associated probes per probe set (SD vs ATC; reported as 60.6% vs 6.6%)
- Overlap (intersection) of the two top-5000 probe sets (reported as ~1.76%)
- Comparison statistics written to gbm_450k_sd_atc_comparison.csv as part of the cola HTML report workflow

## How to apply

After cola consensus partitioning on the preprocessed GBM 450K matrix (GSE36278; probes on sex chromosomes and probes overlapping known SNPs removed, missing values imputed with impute.knn(), winsorized at the 5th/95th percentiles, rows with variance below the 5th percentile dropped), take the top 5000 probes separately by SD score and by ATC score. Annotate both probe sets with Illumina 450K CpG-island association categories (e.g. Island, Shore, Shelf, Open sea) using the 450K probe annotation, and compute the fraction of CpG-island probes in each set — the article reports 60.6% for SD-selected vs 6.6% for ATC-selected probes. Also compute the intersection of the two top-5000 sets (reported as ~1.76% overlap) to quantify how little the two top-value methods agree. The rationale: a high CpG-island fraction indicates probes associated with local, subtype-informative methylation, while ATC-selected probes are dominated by global methylation-level variation.

## Related tools

- **cola** (Provides the top-value methods (SD, ATC) whose top-5000 probe sets are annotated with CpG-island categories, and generates the HTML report containing the comparison) — https://github.com/jokergoo/cola_manuscript
- **R** (Language in which the 450K annotation lookup, probe-set intersection and fraction computation are performed)
- **Bioconductor** (Source of Illumina 450K probe annotation resources for mapping probes to CpG-island association categories)

## Examples

```
overlap_pct <- sum(top_sd_5000 %in% top_atc_5000)/5000*100  # expect ~1.76%; cpg_island_pct <- mean(anno[top_sd_5000, "Relation_to_Island"] == "Island")*100  # expect ~60.6% vs ~6.6% for ATC
```

## Evaluation signals

- The SD-selected top-5000 probe set has a CpG-island fraction near 60.6%, while the ATC-selected top-5000 set is near 6.6% — a large divergence is the expected outcome
- The intersection of the SD and ATC top-5000 probe sets is ~1.76%, so a much higher overlap indicates the preprocessing or top-value scoring deviated from the article
- Both probe sets contain exactly 5000 probes after ordering by SD and ATC scores
- Statistics table gbm_450k_sd_atc_comparison.csv and the gbm_450k_cola_report.html report are produced and contain the CpG-island fractions and overlap values

## Limitations

- ATC-selected features mainly reflect global methylation changes, so CpG-island enrichment (or its absence) should not be read as a failure of ATC but as a property of the method's global-correlation-based selection
- The result is specific to the Illumina 450K platform and its CpG-island association categories; other array designs need analogous annotations
- Interpretation depends on cola preprocessing choices (sex-chromosome/SNP probe removal, imputation, winsorization at the 5th/95th percentiles, 5th-percentile variance cutoff); changing these alters the top probe sets

## Evidence

- [other] Annotate the two top-5000 probe sets with Illumina 450K CpG-island association categories and compute the fraction of CpG-island probes in each set (reported as 60.6% for SD vs 6.6% for ATC).: [evidence span withheld — anchor status: not_found]
