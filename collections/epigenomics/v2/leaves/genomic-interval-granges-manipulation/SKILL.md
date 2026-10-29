---
name: genomic-interval-granges-manipulation
description: Use when coordinate-based query regions (e.g., fine-mapped BMD GWAS SNPs
  at PIP ≥0.001, or epigenome peak sets) must be loaded as genomic intervals in R
  and lifted over between hg38 and mm10 via a UCSC chain file.
when_to_use_negative:
- Input is already an annotated feature table (gene expression matrix, variant summary)
  with no per-interval coordinates to manipulate
- Task requires read-level sequence alignment rather than mapping of pre-defined genomic
  intervals across species
- No chain file or matched orthologous epigenome datasets exist for the target organism
  — use plain UCSC liftOver or a different approach instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: GenomicRanges
  role: Represents and operates on the query and candidate target genomic intervals
    throughout the liftOver and filtering workflow
- name: rtracklayer
  role: Provides the UCSC liftOver framework and import.chain() to load the reciprocal-best
    chain file
- name: data.table
  role: Fast loading and manipulation of SNP coordinate tables before conversion to
    GRanges
- name: AdaLiftOver
  role: R package supplying adaptive_liftover, similarity computation, and gr_candidate_filter
    for interval-based orthologous mapping
  repo: https://github.com/keleslab/AdaLiftOver
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:0eaade0a1323beab0c9fa838d3f6f6094020553615464e5bc5f68d616de26201
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# genomic-interval-granges-manipulation

## Summary

A method for representing query regions such as fine-mapped GWAS SNPs as GRanges objects in R and operating on them across genomes, using rtracklayer chain imports and GenomicRanges interval arithmetic within the AdaLiftOver framework to obtain scored, orthologous target intervals.

## When to use

Use when coordinate-based query regions (e.g., fine-mapped BMD GWAS SNPs at PIP ≥0.001, or epigenome peak sets) must be loaded as genomic intervals in R and lifted over between hg38 and mm10 via a UCSC chain file.

## When NOT to use

- Input is already an annotated feature table (gene expression matrix, variant summary) with no per-interval coordinates to manipulate
- Task requires read-level sequence alignment rather than mapping of pre-defined genomic intervals across species
- No chain file or matched orthologous epigenome datasets exist for the target organism — use plain UCSC liftOver or a different approach instead

## Inputs

- Fine-mapped GWAS SNP coordinate lists (e.g., UK Biobank BMD SNPs at PIP ≥ 0.1 and PIP ≥ 0.001) as BED-like genomic intervals
- UCSC reciprocal-best chain file (mm10.hg38.rbest.chain / mm10.hg38.rbest.chain.gz)
- Orthologous epigenome repertoires as GRangesList objects (epigenome_mm10, epigenome_hg38; or epigenome_rn6_test/epigenome_hg38_test for rat)
- JASPAR position frequency matrix list (jaspar_pfm_list) for motif/grammar similarity
- BSgenome objects for both species (BSgenome.Hsapiens.UCSC.hg38, BSgenome.Mmusculus.UCSC.mm10)

## Outputs

- GRangesList of candidate target regions per query interval, each annotated with epigenomic similarity, grammar similarity, and logistic probability scores
- Filtered GRangesList after gr_candidate_filter (at most best_k candidates per query, scores above threshold)
- Flat CSV export of filtered, scored orthologous mouse target regions (e.g., bmd_snp_adaliftover_results.csv)

## How to apply

Load the SNP/interval lists (e.g., the 3,125 SNPs at PIP ≥0.1 and 116,402 SNPs at PIP ≥0.001 from the UK Biobank BMD GWAS) with data.table and convert them to GRanges objects via GenomicRanges. Import the UCSC reciprocal-best chain (mm10.hg38.rbest.chain) with rtracklayer::import.chain, then extend each query interval to candidate target regions using AdaLiftOver::adaptive_liftover. Score candidates by epigenomic signal similarity (67 matched ENCODE human-mouse TF/DNA methylation ChIP-seq and DNase-seq datasets) and sequence grammar similarity (jaspar_pfm_list motif scanning with motifmatchr), aggregate into logistic probability scores, and prune with gr_candidate_filter using best_k and a score threshold. The rationale is that plain liftOver suffers from multi-mapping and low precision; the GRanges-based candidate structure is what enables per-query filtering and prioritization.

## Related tools

- **GenomicRanges** (Represents and operates on the query and candidate target genomic intervals throughout the liftOver and filtering workflow)
- **rtracklayer** (Provides the UCSC liftOver framework and import.chain() to load the reciprocal-best chain file)
- **data.table** (Fast loading and manipulation of SNP coordinate tables before conversion to GRanges)
- **AdaLiftOver** (R package supplying adaptive_liftover, similarity computation, and gr_candidate_filter for interval-based orthologous mapping) — https://github.com/keleslab/AdaLiftOver

## Examples

```
chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- import.chain returns a valid Chain object and adaptive_liftover returns a GRangesList keyed by query interval with non-empty candidate sets
- After gr_candidate_filter, every query region retains at most best_k candidate targets and all retained scores exceed the chosen threshold
- Positive control: human SNP rs9486902 at the FOXO3 promoter maps to the mouse Foxo3 promoter region
- Benchmark against UCSC liftOver on the 90 BMD genes reproduces the reported 65.2% precision increase
- Output CSV contains coordinate, score, and query-ID columns for all SNPs with no dropped queries beyond chain-file unmappability

## Limitations

- Existing sequence-conservation liftOver approaches suffer from multi-mapping, low precision, and low mapping rate issues — motivates but also bounds the interval-candidate approach
- best_k and threshold parameters require tuning; for organisms other than mouse, the training_module on a pair of matched epigenome datasets is needed to estimate logistic regression parameters
- Requires BSgenome packages for both species and matched orthologous functional genomic datasets; without them scoring cannot proceed

## Evidence

- [intro] GenomicRanges is used for operating genomic intervals.: "**GenomicRanges** is used for operating genomic intervals."
- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions"
- [readme] **best_k**: retain at most best k candidate target regions for each query region.: "**best_k**: retain at most best k candidate target regions for each query region."
- [readme] chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"): "chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain")"
