---
name: precision-and-true-positive-benchmark-evaluation
description: Use when you need to quantitatively compare two genomic interval mapping
  methods on matched orthologous peak sets, evaluating mean precision and true-positive
  counts against a gold-standard orthologous dataset.
when_to_use_negative:
- Your query regions have no matched orthologous epigenomic datasets, so epigenomic
  signal similarities and true-positive gold standards cannot be computed.
- You are mapping within the same genome or using synthetic intervals without an orthologous
  peak reference to define true positives.
- You only need a liftover coordinate conversion without benchmarking against a competing
  method or ground-truth orthologous peaks.
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: AdaLiftOver
  role: 'Method under test: extends query regions via the UCSC liftOver framework,
    prioritizes candidate target regions with epigenomic and sequence grammar similarities,
    and filters with gr_candidate_filter'
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Baseline mapping method (via rtracklayer) against which precision and true-positive
    recovery are compared, using the mm10.hg38.rbest.chain.gz reciprocal-best chain
- name: rtracklayer
  role: R interface to run UCSC liftOver and import the chain file with import.chain
    for the baseline arm of the benchmark
- name: GenomicRanges
  role: Used to represent the 55 matched human-mouse TF ChIP-seq peak sets as GRanges
    objects and compute true-positive overlaps with orthologous peaks
- name: data.table
  role: Fast data manipulation and aggregation for computing and summarizing benchmark
    results
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:8e25e51b1055d355b38e5db134445eceb6649d2b014c6d91736474205bf59132
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/keleslab/AdaLiftOver
  tools:
  - AdaLiftOver
  - GenomicRanges
  - R
  - UCSC liftOver
  - data.table
  - rtracklayer
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# precision-and-true-positive-benchmark-evaluation

## Summary

Benchmarks a cross-species liftover method (AdaLiftOver) against UCSC liftOver on matched human-mouse TF ChIP-seq peak sets, scoring each lifted peak by precision and true-positive recovery against orthologous ENCODE peaks.

## When to use

Use when you need to quantitatively compare two genomic interval mapping methods on matched orthologous peak sets, evaluating mean precision and true-positive counts against a gold-standard orthologous dataset.

## When NOT to use

- Your query regions have no matched orthologous epigenomic datasets, so epigenomic signal similarities and true-positive gold standards cannot be computed.
- You are mapping within the same genome or using synthetic intervals without an orthologous peak reference to define true positives.
- You only need a liftover coordinate conversion without benchmarking against a competing method or ground-truth orthologous peaks.

## Inputs

- 55 matched human-mouse TF ChIP-seq peak sets (erythroid and lymphoblast cells) from matched ENCODE functional genomic datasets, as GRanges objects
- UCSC reciprocal-best chain file mm10.hg38.rbest.chain.gz
- Matched orthologous functional genomic (epigenome) datasets for epigenomic signal similarity computation
- jaspar_pfm_list (JASPAR PFMs) for sequence grammar similarity
- UCSC liftOver mappings via rtracklayer as the comparison baseline

## Outputs

- Per-TF precision and true-positive counts for AdaLiftOver and UCSC liftOver
- Mean precision and mean true-positive counts aggregated across the 55 TF datasets
- liftover_benchmark_results.csv

## How to apply

Load the 55 matched human-mouse TF ChIP-seq peak sets (erythroid and lymphoblast cells) from the matched ENCODE functional genomic datasets as GRanges objects with GenomicRanges. Map each mouse query peak set to human with UCSC liftOver (via rtracklayer) using the reciprocal-best chain file mm10.hg38.rbest.chain.gz, and in parallel run AdaLiftOver's adaptive_liftover with epigenomic signal and sequence grammar similarities aggregated as logistic probability scores, filtered with gr_candidate_filter (best_k per query region and the suggested score threshold). Score each lifted peak as a true positive when it overlaps the orthologous peak from the matched ENCODE dataset, then compute precision and true-positive counts per TF for both methods. Accept the run if AdaLiftOver's mean precision reaches ~0.372 (>50% above UCSC liftOver) with ~4456 mean true positives versus ~4313 for UCSC liftOver, matching the paper's benchmark figures.

## Related tools

- **AdaLiftOver** (Method under test: extends query regions via the UCSC liftOver framework, prioritizes candidate target regions with epigenomic and sequence grammar similarities, and filters with gr_candidate_filter) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Baseline mapping method (via rtracklayer) against which precision and true-positive recovery are compared, using the mm10.hg38.rbest.chain.gz reciprocal-best chain)
- **rtracklayer** (R interface to run UCSC liftOver and import the chain file with import.chain for the baseline arm of the benchmark)
- **GenomicRanges** (Used to represent the 55 matched human-mouse TF ChIP-seq peak sets as GRanges objects and compute true-positive overlaps with orthologous peaks)
- **data.table** (Fast data manipulation and aggregation for computing and summarizing benchmark results)

## Examples

```
chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- AdaLiftOver's mean precision across the 55 TF datasets equals ~0.372, which is >50% above UCSC liftOver's mean precision
- AdaLiftOver's mean true-positive count is ~4456 versus ~4313 for UCSC liftOver (AdaLiftOver recovers more true positives on average)
- A lifted peak is counted as a true positive only when it overlaps the orthologous peak from the matched ENCODE dataset
- Results are written to liftover_benchmark_results.csv with per-TF precision and true-positive counts for both methods
- Candidate target regions are filtered with gr_candidate_filter retaining at most best_k candidates per query region at the suggested score threshold

## Limitations

- The benchmark relies on matched ENCODE functional genomic datasets (67 matched human-mouse TF/DNA methylation ChIP-seq and DNase-seq); without matched orthologous epigenome data, epigenomic similarity computation and true-positive labeling are impossible.
- AdaLiftOver requires genome-specific BSgenome objects and chain files (e.g., BSgenome.Mmusculus.UCSC.mm10, BSgenome.Hsapiens.UCSC.hg38, mm10.hg38.rbest.chain.gz); other model organisms (e.g., rat) need a user-defined epigenome repertoire and a training_module run to estimate logistic regression parameters.
- Existing sequence-conservation mapping methods suffer from multi-mapping, low precision, and low mapping rate issues, so baseline UCSC liftOver results may reflect those intrinsic limitations.

## Evidence

- [results] AdaLiftOver achieves the best precision while maintaining, on average, 4456 true positives compared to UCSC liftOver's 4313 true positives.: [evidence span withheld — anchor status: not_found]
- [intro] We uses 67 matched ENCODE functional genomic datasets including TF/DNA methylation ChIP-seq and DNase-seq between human and mouse: "We uses 67 matched ENCODE functional genomic datasets including TF/DNA methylation ChIP-seq and DNase-seq between human and mouse"
- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence"
- [intro] aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function: "We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function."
- [intro] best_k: retain at most best k candidate target regions for each query region.: "**best_k**: retain at most best k candidate target regions for each query region."
- [readme] chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"): "## load the UCSC chain file
chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain")"
