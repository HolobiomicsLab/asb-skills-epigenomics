---
name: chromatin-accessibility-enrichment-analysis
description: Use when lifted-over SNP or interval sets must be linked to cell-type-specific
  regulatory activity — e.g., testing whether mapped SCZ GWAS SNPs are enriched in
  chromatin accessible regions of mouse excitatory vs inhibitory neuron populations.
when_to_use_negative:
- The input is already a feature/enrichment table — no interval intersection is needed.
- Query regions lack a defined target genome or the relevant cell-population accessibility
  data for the species of interest is unavailable.
- The goal is plain coordinate conversion (liftOver) without any downstream functional
  or cell-type interpretation.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: GenomicRanges
  role: Operating genomic intervals for intersecting mapped SNPs with accessible chromatin
    regions of neuronal cell populations
  repo: https://github.com/keleslab/AdaLiftOver
- name: AdaLiftOver
  role: Produces the mapped SNP/interval set (adaptive liftOver of fine-mapped GWAS
    SNPs) that is then tested for accessibility enrichment
  repo: https://github.com/keleslab/AdaLiftOver
- name: rtracklayer
  role: Runs UCSC liftOver via chain files (e.g., mm10.hg38.rbest.chain) to generate
    the comparator mapped SNP set
- name: data.table
  role: Fast data manipulation for tabulating mapping counts, rates, and enrichment
    statistics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:567b82a5197e3d38d382c9e04627ca76139f2cf49086f5bce9d4407e3b66745f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/keleslab/AdaLiftOver
  tools:
  - AdaLiftOver
  - GenomicRanges
  - R
  - data.table
  - rtracklayer
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# chromatin-accessibility-enrichment-analysis

## Summary

Intersects lifted-over GWAS SNPs (or query intervals) with chromatin accessible regions of specific cell populations (e.g., mouse excitatory and inhibitory neurons) using GenomicRanges, then computes enrichment statistics to identify cell-type-specific regulatory associations, as done in the AdaLiftOver SCZ GWAS case study.

## When to use

Use when lifted-over SNP or interval sets must be linked to cell-type-specific regulatory activity — e.g., testing whether mapped SCZ GWAS SNPs are enriched in chromatin accessible regions of mouse excitatory vs inhibitory neuron populations.

## When NOT to use

- The input is already a feature/enrichment table — no interval intersection is needed.
- Query regions lack a defined target genome or the relevant cell-population accessibility data for the species of interest is unavailable.
- The goal is plain coordinate conversion (liftOver) without any downstream functional or cell-type interpretation.

## Inputs

- Mapped (lifted-over) SNP coordinates in target-genome coordinates (e.g., mouse mm10 GRanges)
- Chromatin accessible region peak sets for cell populations (e.g., mouse excitatory and inhibitory neuron ATAC/DNase peaks)
- Unmapped/unfiltered SNP lists for background or comparison
- Mapping statistics from AdaLiftOver and UCSC liftOver runs

## Outputs

- Per-cell-population overlap/enrichment statistics for mapped SNPs
- Comparison table (e.g., snp_liftover_mapping_stats.csv) with mapping counts, mapping rates, precision, and enrichment results

## How to apply

After mapping query SNPs with AdaLiftOver or UCSC liftOver, intersect the mapped SNP coordinates with chromatin accessible regions of the target cell populations (e.g., mouse excitatory and inhibitory neurons) using GenomicRanges interval operations. Tabulate overlaps per cell population and compute enrichment statistics relative to an appropriate background (e.g., all mapped SNPs or accessible-region coverage). In the SCZ case study, 1648 fine-mapped hg38 SNPs were lifted to mouse (AdaLiftOver: 612 mapped, 37.5% at average precision 8.07%; UCSC liftOver: 715 mapped, 43.3%) and the mapped set was intersected with neuronal accessibility data. Write the comparison/enrichment table to a CSV (e.g., snp_liftover_mapping_stats.csv) so both mapping and enrichment outcomes are verifiable.

## Related tools

- **GenomicRanges** (Operating genomic intervals for intersecting mapped SNPs with accessible chromatin regions of neuronal cell populations) — https://github.com/keleslab/AdaLiftOver
- **AdaLiftOver** (Produces the mapped SNP/interval set (adaptive liftOver of fine-mapped GWAS SNPs) that is then tested for accessibility enrichment) — https://github.com/keleslab/AdaLiftOver
- **rtracklayer** (Runs UCSC liftOver via chain files (e.g., mm10.hg38.rbest.chain) to generate the comparator mapped SNP set)
- **data.table** (Fast data manipulation for tabulating mapping counts, rates, and enrichment statistics)

## Examples

```
chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- Reproduce reported mapping counts before enrichment: AdaLiftOver maps 612 of 1648 SCZ SNPs (37.5%) at average precision 8.07%; UCSC liftOver maps 715 (43.3%).
- Overlap counts between mapped SNPs and each cell population's accessible regions are non-negative and consistent between GRanges intersection and independent tabulation.
- Enrichment statistics are computed against a defined background and can be written to snp_liftover_mapping_stats.csv without missing cell-population columns.
- All intersected intervals share the same genome build (mm10) and coordinate system as the mapped SNP set.
- Sanity check: SNPs reported as unmapped by both methods contribute zero overlaps.

## Limitations

- UCSC liftOver maps a higher raw number of SNPs (715 vs 612), so raw mapping rate alone is not evidence of better functional mapping; enrichment interpretation must account for precision differences.
- Enrichment depends on availability of matched, orthologous accessibility datasets for the target species and cell populations.
- Chain-file-driven extension and filtering (best_k, score threshold in gr_candidate_filter) affect which SNPs are available for enrichment testing.

## Evidence

- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions"
- [results] In the SCZ GWAS case study, AdaLiftOver maps 612 (37.5%) of the 1648 SNPs to the mouse genome and achieves a higher average precision of 8.07%, in comparison to UCSC liftOver which maps 715 (43.3%) SNPs.: [evidence span withheld — anchor status: not_found]
- [full_text] Intersect mapped SNPs with chromatin accessible regions of mouse excitatory and inhibitory neuron populations using GenomicRanges. Compute enrichment statistics and write the comparison table to snp_liftover_mapping_stats.csv.: [evidence span withheld — anchor status: not_found]
- [intro] filter the scores with gr_candidate_filter() function: "We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function."
- [readme] chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"): "chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain")"
