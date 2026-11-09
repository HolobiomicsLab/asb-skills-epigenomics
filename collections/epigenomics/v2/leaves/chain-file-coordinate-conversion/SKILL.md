---
name: chain-file-coordinate-conversion
description: Use when query regions in one genome (e.g., hg38 SNP coordinates) must
  be mapped to another genome (e.g., mm10), especially when regulatory function of
  mapped loci matters and plain chain-based liftOver may multi-map or miss orthologous
  elements.
when_to_use_negative:
- The input is a protein or transcript-level correspondence problem rather than genomic
  interval coordinate conversion
- Only syntactic coordinate transfer is needed and regulatory similarity is irrelevant,
  multi-mapping is acceptable, and the extra epigenomic/grammar scoring overhead is
  unnecessary
- No chain file or matched orthologous functional genomic datasets are available for
  the target species, so candidate regions cannot be extended or scored
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0622
tools:
- name: AdaLiftOver
  role: R package that extends query regions via the UCSC liftOver chain framework,
    scores candidate target regions on epigenomic and sequence grammar similarity,
    and filters them with gr_candidate_filter()
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Baseline chain-based coordinate conversion tool; provides the framework AdaLiftOver
    builds on and is used via rtracklayer to map SNPs for comparison
- name: rtracklayer
  role: R/Bioconductor package used to import the chain file (import.chain) and run
    liftOver conversion of query regions
- name: GenomicRanges
  role: R/Bioconductor package for operating on genomic intervals, used for intersecting
    mapped SNPs with chromatin accessible regions and computing enrichment
- name: data.table
  role: R package used for fast data manipulation and computation when tabulating
    mapping statistics and training results
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:ff924d03b71637dd59c91b66974c67e14bfbbe7976d847ecd48c08f717acefa8
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
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# chain-file-coordinate-conversion

## Summary

Converts genomic coordinates (e.g., fine-mapped GWAS SNPs or regulatory intervals) across species using UCSC reciprocal-best chain files, either via standard UCSC liftOver/rtracklayer or as the region-extension step that feeds AdaLiftOver's epigenomic and sequence grammar scoring.

## When to use

Use when query regions in one genome (e.g., hg38 SNP coordinates) must be mapped to another genome (e.g., mm10), especially when regulatory function of mapped loci matters and plain chain-based liftOver may multi-map or miss orthologous elements.

## When NOT to use

- The input is a protein or transcript-level correspondence problem rather than genomic interval coordinate conversion
- Only syntactic coordinate transfer is needed and regulatory similarity is irrelevant, multi-mapping is acceptable, and the extra epigenomic/grammar scoring overhead is unnecessary
- No chain file or matched orthologous functional genomic datasets are available for the target species, so candidate regions cannot be extended or scored

## Inputs

- Query genomic regions as GRanges (e.g., 1648 fine-mapped SCZ GWAS SNP coordinates in hg38)
- UCSC reciprocal-best chain file (e.g., mm10.hg38.rbest.chain.gz from hgdownload.cse.ucsc.edu)
- Orthologous epigenome repertoires (e.g., ENCODE TF/DNA methylation ChIP-seq and DNase-seq for human and mouse) for similarity scoring
- BSgenome objects for both species (e.g., BSgenome.Hsapiens.UCSC.hg38, BSgenome.Mmusculus.UCSC.mm10) and a JASPAR PFM list for grammar similarity

## Outputs

- Mapped and unmapped interval sets per method (GRanges lists of candidate target regions with scores)
- Filtered candidate target regions (gr_list_filter) after gr_candidate_filter with best_k and score threshold
- Mapping statistics table (e.g., snp_liftover_mapping_stats.csv with counts, mapping rates, and precision per method)

## How to apply

Load the query regions as GRanges and import the reciprocal-best UCSC chain file (e.g., mm10.hg38.rbest.chain or rn6.hg38.rbest.chain) with rtracklayer::import.chain. For baseline conversion, run UCSC liftOver through rtracklayer and record mapped vs. unmapped intervals. For adaptive conversion, call adaptive_liftover(gr, chain) to extend query regions via the chain into candidate target regions, then score candidates by epigenomic signal similarity and sequence grammar similarity, and retain at most best_k candidates per query above a score threshold with gr_candidate_filter() (e.g., best_k = 1L, threshold = 0.5). Compare mapping counts and rates between the two approaches to judge conversion quality.

## Related tools

- **AdaLiftOver** (R package that extends query regions via the UCSC liftOver chain framework, scores candidate target regions on epigenomic and sequence grammar similarity, and filters them with gr_candidate_filter()) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Baseline chain-based coordinate conversion tool; provides the framework AdaLiftOver builds on and is used via rtracklayer to map SNPs for comparison)
- **rtracklayer** (R/Bioconductor package used to import the chain file (import.chain) and run liftOver conversion of query regions)
- **GenomicRanges** (R/Bioconductor package for operating on genomic intervals, used for intersecting mapped SNPs with chromatin accessible regions and computing enrichment)
- **data.table** (R package used for fast data manipulation and computation when tabulating mapping statistics and training results)

## Examples

```
chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- Mapping counts and rates match reported baselines: UCSC liftOver maps 715 (43.3%) of the 1648 SCZ GWAS SNPs, AdaLiftOver maps 612 (37.5%)
- AdaLiftOver achieves higher average precision (8.07%) than UCSC liftOver on the SCZ SNP mapping task
- Filtered output obeys gr_candidate_filter parameters: at most best_k candidate target regions per query region and all scores above the threshold
- Every query region yields a scored, filtered candidate list; unmapped regions are explicitly recorded rather than silently dropped
- Imported chain file produces non-empty liftOver results on a known-mappable control region before running the full SNP set

## Limitations

- AdaLiftOver maps fewer SNPs than UCSC liftOver (612 vs. 715 on SCZ GWAS SNPs); it trades mapping rate for higher precision, so it is inappropriate when maximizing raw mapping rate is the goal
- For model organisms other than mouse (e.g., rat), a user-defined orthologous epigenome repertoire and a new training_module() fit (max_filter_proportion = 0.5) are needed before scoring/filtering is reliable
- Requires BSgenome packages for both species and chain files specific to the genome pair; no changelog is available for the repository

## Evidence

- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions"
- [readme] chain import and liftOver quick start: "chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain")

## map the query regions
gr_list <- adaptive_liftover(gr, chain)"
- [readme] filter step parameters: "gr_list_filter <- gr_candidate_filter(
    gr_list,
    best_k = 1L,
    threshold = 0.5
)"
- [intro] candidate filter semantics: "**best_k**: retain at most best k candidate target regions for each query region.
* **threshold**: the score threshold for all the candidate target regions."
- [intro] aggregation of similarities: "We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function."
