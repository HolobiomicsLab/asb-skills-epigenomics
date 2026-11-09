---
name: tissue-specific-gene-enrichment-analysis
description: Use when a differential alignment or ranking of genes across two tissue
  conditions needs biological validation against a curated set of known tissue-specific
  genes and GO terms.
when_to_use_negative:
- Input is a raw alignment score matrix rather than a ranked gene list with per-gene
  P-values — enrichment testing requires the ranked differential output first
- The gene set under study has no prior tissue-specificity labels or GO annotation,
  making hypergeometric and GOrilla validation impossible
- Genes on chromosome Y should be excluded per the article's filter, so do not include
  them when counting top-ranked genes
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: EpiAlign
  role: Produces the differential alignment scores and Wilcoxon P-values that are
    ranked and validated by enrichment analysis
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Encodes multi-track epigenomic signals from Roadmap samples (e.g. brain E071
    and heart) into the single-track chromatin state sequences aligned upstream
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:be9f8810c59ae9468ae672626e73ccc1f85a5c2d0487571f3f0ff307c90b8dff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tissue-specific-gene-enrichment-analysis

## Summary

Validates chromatin-state-based differential alignment results (EpiAlign) by testing whether previously known tissue-specific genes are enriched among the top differentially aligned genes, using hypergeometric enrichment and GO term analysis (GOrilla, P < 0.0001).

## When to use

Use when a differential alignment or ranking of genes across two tissue conditions needs biological validation against a curated set of known tissue-specific genes and GO terms.

## When NOT to use

- Input is a raw alignment score matrix rather than a ranked gene list with per-gene P-values — enrichment testing requires the ranked differential output first
- The gene set under study has no prior tissue-specificity labels or GO annotation, making hypergeometric and GOrilla validation impossible
- Genes on chromosome Y should be excluded per the article's filter, so do not include them when counting top-ranked genes

## Inputs

- Ranked gene list with differential alignment P-values (Wilcoxon rank-sum on EpiAlign alignment scores)
- Brain-specific and heart-specific gene labels for protein-coding genes (GENCODE v10 set compatible with Roadmap)
- Gene coordinates for the 19,935 protein-coding genes

## Outputs

- Hypergeometric enrichment P-value for tissue-specific genes among the top 100 differential genes (expected P < 10^-30)
- List of enriched GO terms from GOrilla with P < 0.0001
- Identity of the top-ranked gene (expected: STMN4)

## How to apply

After ranking all protein-coding genes by differential alignment P-value (two-sample one-sided Wilcoxon rank-sum test comparing within-brain vs brain-vs-heart EpiAlign local alignment scores), take the top 100 ranked genes and count how many carry brain-specific or heart-specific labels (expected ~9 brain-specific and ~4 heart-specific). Assess significance with a hypergeometric test, requiring P < 10^-30. Additionally, run GOrilla GO enrichment on the top-ranked aligned genes and extract terms with P < 0.0001 to confirm coherent biological function. The validation succeeds if STMN4 receives the smallest P-value and both the hypergeometric and GO thresholds are met.

## Related tools

- **EpiAlign** (Produces the differential alignment scores and Wilcoxon P-values that are ranked and validated by enrichment analysis) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Encodes multi-track epigenomic signals from Roadmap samples (e.g. brain E071 and heart) into the single-track chromatin state sequences aligned upstream)

## Examples

```
Rscript differential_align_enrichment.R --brain E071 --heart E075 --gencode v10 --top 100 --hypergeom --gorilla-cutoff 1e-4
```

## Evaluation signals

- STMN4 receives the top (smallest) differential alignment P-value
- Hypergeometric enrichment of tissue-specific genes in the top 100 yields P < 10^-30 (expected ~9 brain-specific and ~4 heart-specific genes)
- GOrilla GO terms extracted at P < 0.0001 from the top-ranked aligned genes
- Ranking derived from a two-sample one-sided Wilcoxon rank-sum test over within-brain vs brain-versus-heart score groups for each of the 19,935 protein-coding genes

## Limitations

- Genes on chromosome Y are excluded from the analysis
- Alignment results depend on ChromHMM annotations, which may contain likely noisy states requiring further biological validation (Supplementary section 12 not included)
- No specialized substitution matrices for chromatin states are available; constructing them is an open research question

## Evidence

- [results] we perform the two-sample one-sided Wilcoxon test between male-vs-male scores and male-vs-female scores for each protein-coding gene: "we perform the two-sample one-sided Wilcoxon test between male-vs-male scores and male-vs-female scores for each protein-coding gene"
- [results] Vertical alignment of genes between samples: "we compare the chromatin state sequences for each of the 19,935 protein-coding genes between every pair of samples"
- [results] GO enrichment on top genes: "We next perform the Gene Ontology (GO) enrichment analysis (30) on the top 200 genes that receive the smallest P-values"
- [methods] ChromHMM state encoding: "We first apply a chromatin state annotation method (e.g. ChromHMM (9)) to encode multi-track epigenomic signals into single-track chromatin state sequences"
- [results] Excluded chromosome Y genes: "with the exception of genes on chromosome Y"
