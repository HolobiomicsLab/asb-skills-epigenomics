---
name: chromatin-accessibility-expression-integration
description: Use when you have paired DNase1-seq and RNA-seq samples across multiple
  cell types/tissues and need to link regulatory regions to genes whose expression
  variation they explain.
when_to_use_negative:
- You only have a user-provided set of DNA regions and want enriched TF motifs with
  no expression data — use motif enrichment tools such as PASTAA, Clover, MotifCounter
  or Homer instead.
- You need REMs identified de novo from chromatin data without any paired gene expression
  variation across samples — the STITCHIT linear model requires correlated accessibility/expression
  variation.
- You want a reverse mapping (starting from genomic regions to find associated genes)
  — EpiRegio/STITCHIT explicitly starts with a gene and looks for REMs, not vice versa.
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: STITCHIT
  role: Linear model that interprets chromatin accessibility (DNase1-seq) data with
    respect to variation in gene expression to identify putative REMs per gene within
    the 100 kb-upstream / gene body / 100 kb-downstream search window
  repo: https://epiregio.de/
- name: EpiRegio
  role: Web server and REST API providing the REM resource built with StitchIt, including
    Gene Query, REM activity, Cell type score, and CSV download of results
  repo: https://github.com/TeamRegio/EpiRegioDB
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:ecf88c0764e3c3b844bfef2c4325057de2d7f2bcb2daa449ecf89af10802dce8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EpiRegio
  - STITCHIT
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# chromatin-accessibility-expression-integration

## Summary

Integrates paired DNase1-seq chromatin accessibility and RNA-seq gene expression data using the STITCHIT linear model (as implemented in EpiRegio) to identify Regulatory EleMents (REMs) whose accessibility variation is associated with changes in target gene expression across samples.

## When to use

Use when you have paired DNase1-seq and RNA-seq samples across multiple cell types/tissues and need to link regulatory regions to genes whose expression variation they explain.

## When NOT to use

- You only have a user-provided set of DNA regions and want enriched TF motifs with no expression data — use motif enrichment tools such as PASTAA, Clover, MotifCounter or Homer instead.
- You need REMs identified de novo from chromatin data without any paired gene expression variation across samples — the STITCHIT linear model requires correlated accessibility/expression variation.
- You want a reverse mapping (starting from genomic regions to find associated genes) — EpiRegio/STITCHIT explicitly starts with a gene and looks for REMs, not vice versa.

## Inputs

- Paired DNase1-seq (sequencing-depth normalized) and RNA-seq samples across multiple cell types/tissues (e.g. 110 Roadmap + 56 Blueprint samples)
- GENCODE V27 gene annotation

## Outputs

- REMs with regression coefficients, assigned to their target genes
- CREMs (Clusters of Regulatory EleMents, minimum two REMs, unique IDs)
- Database-level metrics: total REM count, associated gene count, CREM count, mean/SD REM length, number of tissues/cell types (e.g. stitchit_database_metrics.csv)

## How to apply

Download the uniformly preprocessed paired DNase1-seq and RNA-seq samples (e.g. the 110 Roadmap plus 56 Blueprint samples spanning 46 tissues and cell types) with the GENCODE V27 gene annotation. For every gene, restrict analysis to the STITCHIT search window: 100,000 bp upstream of the TSS, the entire gene body, and 100,000 bp downstream of the transcription termination site. Partition this window into distinct regions whose sequencing-depth-normalized DNase1-seq signal varies in correlation with gene expression quantified in transcripts per million (TPM), fitting the STITCHIT linear model per gene to obtain REMs with regression coefficients. Aggregate REMs across all samples, assign each REM to its gene, and group overlapping or directly adjacent REMs into CREMs (minimum two REMs, unique CREM ID). The gene-centric design (starting from a gene and looking for REMs, not vice versa) annotates regions in relation to actual gene expression changes, potentially yielding higher specificity than peak-calling-based approaches.

## Related tools

- **STITCHIT** (Linear model that interprets chromatin accessibility (DNase1-seq) data with respect to variation in gene expression to identify putative REMs per gene within the 100 kb-upstream / gene body / 100 kb-downstream search window) — https://epiregio.de/
- **EpiRegio** (Web server and REST API providing the REM resource built with StitchIt, including Gene Query, REM activity, Cell type score, and CSV download of results) — https://github.com/TeamRegio/EpiRegioDB

## Evaluation signals

- Reproduce the reported database-level metrics on the Roadmap+Blueprint data: 2,404,861 REMs, 35,379 protein-coding and non-protein-coding genes, 365,286 CREMs, 46 tissues/cell types, and average REM length of 229 bp (±235 bp).
- Confirm every REM lies within its gene's STITCHIT search window (100,000 bp upstream of TSS, full gene body, 100,000 bp downstream of transcription termination).
- Verify each CREM contains at least two REMs and has a unique CREM ID.
- Check that DNase1-seq signal is sequencing-depth normalized and expression values are quantified in transcripts per million (TPM) before fitting the per-gene linear model.
- Confirm REM-gene association directionality: each REM is assigned to the gene whose expression variation it explains, consistent with the gene-centric StitchIt design.

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification of the actual regulation of genes by their REMs.
- Identifying REMs is difficult, as there is no method yet to locate them with absolute certainty; StitchIt annotates putative REMs via indirect epigenomic indicators.
- No tool versions are stated for any of the software mentioned (PASTAA, EpiRegio, etc.).

## Evidence

- [intro] EpiRegio uses the StitchIt algorithm which interprets chromatin accessibility data with respect to variation in gene expression: "EpiRegio uses the StitchIt algorithm which interprets chromatin accessibility data with respect to variation in gene expression (21). The algorithm identifies putative REMs that explain variation of"
- [intro] In contrast to existing approaches, it starts with a gene and looks for REMs, not viceversa.: "In contrast to existing approaches, it starts with a gene and looks for REMs, not viceversa."
- [intro] StitchIt’s approach to interpret epigenomic variation in gene expression has the advantage of annotating regions that were observed in relation to actual gene expression changes: "StitchIt’s approach to interpret epigenomic variation in gene expression has the advantage of annotating regions that were observed in relation to actual gene expression changes, potentially leading"
- [discussion] the linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs: "the linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs"
- [other] StitchIt on the Roadmap and Blueprint data yielded 2,404,861 REMs associated to 35,379 protein-coding and non-protein-coding genes: [evidence span withheld — anchor status: not_found]
