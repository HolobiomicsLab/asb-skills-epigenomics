---
name: cell-type-score-ranking-of-rems
description: Use when a set of REMs from an EpiRegio Gene Query must be prioritized
  by cell type/tissue specificity (e.g. 'heart') before downstream TF motif enrichment
  such as PASTAA.
when_to_use_negative:
- The goal is finding a gene's REMs regardless of cell type specificity, with no downstream
  rank-aware enrichment tool — an unranked EpiRegio Gene Query suffices.
- The input is already ChIP-seq peak regions to intersect with REMs — use EpiRegio's
  Region Query with an Overlap percentage instead of Cell type score ranking.
- The downstream motif tool does not exploit sequence ranking (e.g. set-based tools
  like Clover or Homer accept unordered region sets).
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: EpiRegio
  role: Web server/REST API providing REMs linked to genes of interest, with the Cell
    type score used for ranking; result table downloaded as CSV
  repo: https://github.com/TeamRegio/EpiRegioDB
- name: STITCHIT
  role: Underlying algorithm whose linear model of chromatin accessibility vs. gene
    expression variation yields the Cell type score used to rank REMs
- name: PASTAA
  role: Downstream consumer of the ranked REM sequences; identifies TF binding motifs
    enriched in high-ranked vs. low-ranked sequences
  repo: https://github.com/TeamRegio/ApplicationScenarioExamples
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:2878c9cfb4e7496dd6811ee98c9d72a8bb72bd4c775e3b6f49cfadf863884dc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamRegio/EpiRegioDB
  tools:
  - EpiRegio
  - PASTAA
  - STITCHIT
  - TRAP
  - bedtools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cell-type-score-ranking-of-REMs

## Summary

Ranks regulatory elements (REMs) retrieved from EpiRegio by their Cell type score — the STITCHIT-derived estimate of each REM's cell type-specific contribution to its target gene's expression — so that motif enrichment tools like PASTAA receive meaningfully ordered high- and low-ranked sequences.

## When to use

Use when a set of REMs from an EpiRegio Gene Query must be prioritized by cell type/tissue specificity (e.g. 'heart') before downstream TF motif enrichment such as PASTAA.

## When NOT to use

- The goal is finding a gene's REMs regardless of cell type specificity, with no downstream rank-aware enrichment tool — an unranked EpiRegio Gene Query suffices.
- The input is already ChIP-seq peak regions to intersect with REMs — use EpiRegio's Region Query with an Overlap percentage instead of Cell type score ranking.
- The downstream motif tool does not exploit sequence ranking (e.g. set-based tools like Clover or Homer accept unordered region sets).

## Inputs

- Set of differentially expressed genes of interest (e.g. GeneSet.txt from TeamRegio/ApplicationScenarioExamples)
- EpiRegio web server / REST API access to the EpiRegioDB REM resource
- Target cell type/tissue filter string (e.g. 'heart')

## Outputs

- REM table (CSV) with per-REM annotations including Cell type score and Cell type DNase1 signal
- REM list ordered by Cell type score for downstream motif enrichment (e.g. convertible to BED for bedtools getfasta)

## How to apply

Run EpiRegio's Gene Query on the gene set of interest, filtering for the relevant cell type/tissue (e.g. 'heart') and optionally setting Cell type score / Cell type DNase1 signal thresholds. Download the resulting REM table as CSV; it includes each REM's Cell type score, which derives from STITCHIT's linear model linking chromatin accessibility variation to gene expression variation. Sort the REMs by Cell type score so high-scoring REMs are the cell type-relevant positives and low-scoring REMs serve as background for downstream analysis such as PASTAA motif enrichment. Because STITCHIT's linear model is a simplification of real gene regulation, treat the score as an estimated contribution rather than a measured regulatory effect.

## Related tools

- **EpiRegio** (Web server/REST API providing REMs linked to genes of interest, with the Cell type score used for ranking; result table downloaded as CSV) — https://github.com/TeamRegio/EpiRegioDB
- **STITCHIT** (Underlying algorithm whose linear model of chromatin accessibility vs. gene expression variation yields the Cell type score used to rank REMs)
- **PASTAA** (Downstream consumer of the ranked REM sequences; identifies TF binding motifs enriched in high-ranked vs. low-ranked sequences) — https://github.com/TeamRegio/ApplicationScenarioExamples

## Examples

```
bedtools getfasta -fi hg38.fa -bed REMs.bed -fo REMs.fa
```

## Evaluation signals

- The downloaded EpiRegio Gene Query CSV contains a Cell type score column for every REM and only REMs matching the specified cell type/tissue filter (e.g. 'heart').
- Sorting by Cell type score yields REMs whose Cell type DNase1 signal is coherent with the score (thresholds refer to these two quantities).
- Downstream PASTAA on the ranked sequences recovers expected enriched TFs (e.g. 230 TFs including SP1, NFKB1, SMAD3) at Benjamini–Hochberg FDR ≤ 0.05.
- The BED derived from the ranked REM table round-trips correctly: bedtools getfasta extracts exactly one sequence per REM into REMs.fa.

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs.
- No method exists to locate REMs with absolute certainty; REM identity rests on indirect epigenomic indicators.
- No tool versions are stated for EpiRegio or downstream software in the source material.

## Evidence

- [discussion] Cell type score from STITCHIT linear model: "the linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs"
- [discussion] Ranking REMs by Cell type score: "In order to rank the obtained REMs we sorted them according to their Cell type score"
- [methods] Gene Query with cell type/tissue filter: "Use EpiRegio's `Gene Query <https://epiregiodb.readthedocs.io/en/latest/UseCases.html#query-guide>`_ to identify the REMs associated to the genes of interest"
- [methods] Thresholds on Cell type score and DNase1 signal: "The thresholds refer to the *Cell type score* and the *Cell type DNase1 signal* of the REMs in the cell types/tissues."
- [methods] CSV download of result table: "Download the resulting table by clicking on the bottom *CSV*."
