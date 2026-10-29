---
name: liftover-precision-benchmarking
description: Use when comparing cross-species regulatory-element mapping precision
  of AdaLiftOver vs UCSC liftOver, or when validating liftOver-based SNP-to-gene mappings
  against a known positive-control gene list such as BMD genes.
when_to_use_negative:
- When the query regions are coding/exonic and standard UCSC liftOver chain mapping
  is sufficient — AdaLiftOver targets orthologous non-coding regulatory elements
- When no matched human-target-species epigenomic datasets or training data are available
  to estimate logistic regression parameters and an optimal score threshold
- When the goal is coordinate conversion only (no prioritization of candidate target
  regions by regulatory similarity)
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3517
tools:
- name: AdaLiftOver
  role: 'Adaptive liftover tool: extends query regions via the UCSC liftOver framework,
    scores candidates via epigenomic signal and sequence grammar similarities, and
    filters scored target regions'
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Baseline coordinate-conversion framework that AdaLiftOver builds on; serves
    as the precision comparison reference in the benchmark
- name: rtracklayer
  role: Imports the UCSC reciprocal-best chain file via import.chain() for adaptive_liftover
- name: GenomicRanges
  role: Represents GWAS SNPs and candidate target regions as genomic interval objects
    throughout the workflow
- name: data.table
  role: Fast data manipulation and computation when loading and processing SNP interval
    tables
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:fbe1a0a7546b3a8b4e59751bf10a85927157c78f35bd5bbbc4ba38fdf0893d1a
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
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# liftOver-precision-benchmarking

## Summary

Benchmark the precision of AdaLiftOver's epigenome- and grammar-aware orthologous mapping against UCSC liftOver using curated positive-control gene sets, as demonstrated by mapping fine-mapped human BMD GWAS SNPs (PIP ≥ 0.001) to mouse and scoring recovery of 90 BMD genes.

## When to use

Use when comparing cross-species regulatory-element mapping precision of AdaLiftOver vs UCSC liftOver, or when validating liftOver-based SNP-to-gene mappings against a known positive-control gene list such as BMD genes.

## When NOT to use

- When the query regions are coding/exonic and standard UCSC liftOver chain mapping is sufficient — AdaLiftOver targets orthologous non-coding regulatory elements
- When no matched human-target-species epigenomic datasets or training data are available to estimate logistic regression parameters and an optimal score threshold
- When the goal is coordinate conversion only (no prioritization of candidate target regions by regulatory similarity)

## Inputs

- Fine-mapped GWAS SNP lists with PIP values (e.g., 116,402 SNPs at PIP ≥ 0.001; 3,125 SNPs at PIP ≥ 0.1) loaded as GenomicRanges intervals
- UCSC reciprocal-best chain file mm10.hg38.rbest.chain.gz
- 67 matched ENCODE human-mouse TF/DNA methylation ChIP-seq and DNase-seq epigenomic datasets
- JASPAR position frequency matrix list (jaspar_pfm_list) for sequence grammar similarity
- Curated positive-control gene set (e.g., 90 BMD genes) and known orthologous SNP-gene examples (rs9486902 / FOXO3–Foxo3)

## Outputs

- Filtered, scored orthologous mouse target regions per query SNP (e.g., bmd_snp_adaliftover_results.csv)
- Per-query candidate target-region GRanges list with logistic probability scores after gr_candidate_filter
- Precision comparison table/statistic: AdaLiftOver vs UCSC liftOver (e.g., 65.2% precision increase) on the BMD gene set

## How to apply

Load the fine-mapped GWAS SNP sets (e.g., 3,125 SNPs at PIP ≥ 0.1 and 116,402 SNPs at PIP ≥ 0.001 from UK Biobank BMD GWAS) as genomic intervals with GenomicRanges and data.table. Run adaptive_liftover with the UCSC reciprocal-best chain file mm10.hg38.rbest.chain.gz to extend each query SNP to candidate mouse target regions, then score candidates with the training-module logistic regression on epigenomic signal and sequence grammar similarities from the 67 matched ENCODE human-mouse datasets, and filter with gr_candidate_filter (best_k and threshold settings). Compute precision for both AdaLiftOver and UCSC liftOver against the curated positive-control gene set (90 BMD genes), reporting the relative precision gain — the reference result is a 65.2% precision increase for AdaLiftOver. Confirm a known positive-control locus, e.g., SNP rs9486902 at the human FOXO3 promoter, maps to the mouse Foxo3 promoter region. Write the filtered, scored orthologous target regions (e.g., bmd_snp_adaliftover_results.csv) for downstream inspection.

## Related tools

- **AdaLiftOver** (Adaptive liftover tool: extends query regions via the UCSC liftOver framework, scores candidates via epigenomic signal and sequence grammar similarities, and filters scored target regions) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Baseline coordinate-conversion framework that AdaLiftOver builds on; serves as the precision comparison reference in the benchmark)
- **rtracklayer** (Imports the UCSC reciprocal-best chain file via import.chain() for adaptive_liftover)
- **GenomicRanges** (Represents GWAS SNPs and candidate target regions as genomic interval objects throughout the workflow)
- **data.table** (Fast data manipulation and computation when loading and processing SNP interval tables)

## Examples

```
chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- AdaLiftOver precision exceeds UCSC liftOver precision on the same SNP set; the reference result is a 65.2% precision increase on the 90 BMD genes
- Positive-control check: SNP rs9486902 at the human FOXO3 promoter maps to the mouse Foxo3 promoter region
- gr_candidate_filter retains at most best_k candidate target regions per query region and all retained regions meet the score threshold
- Output CSV (bmd_snp_adaliftover_results.csv) contains filtered, scored orthologous mouse target regions for every input SNP
- Training module suggests an optimal score threshold without filtering out too many candidate target regions (max_filter_proportion respected)

## Limitations

- Requires 67 matched ENCODE human-mouse TF/DNA methylation ChIP-seq and DNase-seq datasets; for other model organisms (e.g., rat), an orthologous epigenome repertoire must be collected from scratch and parameters re-estimated via training_module
- AdaLiftOver's logistic regression parameters and threshold are learned from a pair of matched epigenome datasets, so results depend on the training repertoire
- Existing sequence-conservation-based mapping methods suffer from multi-mapping, low precision, and low mapping rate issues; benchmark precision is only as good as the positive-control gene set used

## Evidence

- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence"
- [intro] We uses 67 matched ENCODE functional genomic datasets including TF/DNA methylation ChIP-seq and DNase-seq between human and mouse: "We uses 67 matched ENCODE functional genomic datasets including TF/DNA methylation ChIP-seq and DNase-seq between human and mouse"
- [intro] **best_k**: retain at most best k candidate target regions for each query region.
* **threshold**: the score threshold for all the candidate target regions.: "**best_k**: retain at most best k candidate target regions for each query region."
- [intro] We can download the UCSC chain file from [mm10.hg38.rbest.chain.gz](http://hgdownload.cse.ucsc.edu/goldenpath/mm10/vsHg38/reciprocalBest/mm10.hg38.rbest.chain.gz): "We can download the UCSC chain file from [mm10.hg38.rbest.chain.gz](http://hgdownload.cse.ucsc.edu/goldenpath/mm10/vsHg38/reciprocalBest/mm10.hg38.rbest.chain.gz)"
- [readme] gr_list_filter <- gr_candidate_filter(
    gr_list,
    best_k = 1L,
    threshold = 0.5
): "gr_list_filter <- gr_candidate_filter(
    gr_list,
    best_k = 1L,
    threshold = 0.5
)"
