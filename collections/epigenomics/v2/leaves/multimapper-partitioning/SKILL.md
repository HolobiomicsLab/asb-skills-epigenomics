---
name: multimapper-partitioning
description: Use when you have AdaLiftOver candidate target regions for query peaks
  and need to separate uniquely mapped regions from multi-mappers, or to test whether
  epigenomic signal overrides sequence-conservation-based mapping support.
when_to_use_negative:
- When the goal is orthologous region identification itself rather than post-hoc quality
  stratification — use the full AdaLiftOver pipeline with gr_candidate_filter() instead
  of the partitioning analysis.
- When query regions are coding or already unambiguously annotated orthologs, where
  multi-mapping assessment of non-coding regulatory elements is not the question.
edam_operation: http://edamontology.org/operation_0322
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: AdaLiftOver
  role: Produces the scored candidate target regions whose per-query mapping multiplicity
    defines the Group I/II partition; provides adaptive_liftover() and gr_candidate_filter()
    with a score threshold parameter
  repo: https://github.com/keleslab/AdaLiftOver
- name: GenomicRanges
  role: Operates on the query genomic intervals and mapped candidate regions used
    in the partitioning
- name: data.table
  role: Fast data manipulation and computation for tabulating mapping multiplicities
    and writing the partitioned score table
- name: rtracklayer
  role: Part of the UCSC liftOver framework used to import chain files and extend
    query regions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:2145b8941ac23fdaf341892fed760b41ba62390aae0007e890845331d0cf69fa
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
  edam_operation: http://edamontology.org/operation_0322
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# multimapper-partitioning

## Summary

Partition AdaLiftOver-mapped query regions by mapping multiplicity into Group I (unique sequence-based mappings) and Group II (multi-mappers), then quantify how often epigenomic mapping scores (<0.5) contradict sequence-based mappability in the uniquely mapped group.

## When to use

Use when you have AdaLiftOver candidate target regions for query peaks and need to separate uniquely mapped regions from multi-mappers, or to test whether epigenomic signal overrides sequence-conservation-based mapping support.

## When NOT to use

- When the goal is orthologous region identification itself rather than post-hoc quality stratification — use the full AdaLiftOver pipeline with gr_candidate_filter() instead of the partitioning analysis.
- When query regions are coding or already unambiguously annotated orthologs, where multi-mapping assessment of non-coding regulatory elements is not the question.

## Inputs

- GRanges of query regions (e.g. human hg38 brain ATAC-seq peaks)
- AdaLiftOver output: scored candidate target regions per query region from adaptive_liftover()
- UCSC chain file used for the liftOver mapping (e.g. rn6.hg38.rbest.chain)
- AdaLiftOver mapping scores (logistic probability scores per candidate target region)

## Outputs

- Partitioned region sets: Group I (unique mappers) and Group II (multi-mappers), each with mapping scores
- Fraction of Group I regions with AdaLiftOver mapping scores < 0.5
- CSV table of partitioned regions with scores (e.g. adaliftover_group_scores.csv)

## How to apply

After running adaptive_liftover() on query regions (e.g. human hg38 brain ATAC-seq peaks against a UCSC chain file such as the hg38-to-zebrafish chain), count the number of mapped candidate target regions per query region. Assign queries with a single sequence-based mapping to Group I and queries with multiple mappings to Group II. For Group I regions, extract the AdaLiftOver logistic probability mapping scores and compute the fraction below the 0.5 threshold, which measures how often the epigenomic (ATAC-seq/ChIP-seq signal and sequence grammar) evidence fails to support regulatory conservation despite sequence-based mappability. Write the partitioned regions with their scores and the Group I low-score fraction to an output table such as adaliftover_group_scores.csv.

## Related tools

- **AdaLiftOver** (Produces the scored candidate target regions whose per-query mapping multiplicity defines the Group I/II partition; provides adaptive_liftover() and gr_candidate_filter() with a score threshold parameter) — https://github.com/keleslab/AdaLiftOver
- **GenomicRanges** (Operates on the query genomic intervals and mapped candidate regions used in the partitioning)
- **data.table** (Fast data manipulation and computation for tabulating mapping multiplicities and writing the partitioned score table)
- **rtracklayer** (Part of the UCSC liftOver framework used to import chain files and extend query regions)

## Examples

```
gr_list <- adaptive_liftover(gr, chain); grpI <- sapply(gr_list, length) == 1; frac_low <- mean(mapply(function(g) g$score, gr_list[grpI]) < 0.5)
```

## Evaluation signals

- Every query region is assigned to exactly one group; Group I regions have exactly one mapped candidate target region while Group II regions have multiple.
- All retained Group I mapping scores are logistic probabilities in [0, 1]; the fraction below the 0.5 threshold is reported (e.g. 16.4% in the zebrafish brain ATAC-seq case study).
- The partitioned regions with scores and the Group I low-score fraction are written to the expected CSV (adaliftover_group_scores.csv) with consistent region identifiers.
- Multi-mapper queries (Group II) are excluded from the low-score fraction denominator, since their sequence-based mappability itself is ambiguous.

## Limitations

- Existing sequence-conservation methods for mapping non-coding regions across species suffer from multi-mapping, low precision, and low mapping rate issues, so Group II membership reflects uncertainty in the sequence-based mapping itself.
- A low AdaLiftOver score for a uniquely mapped region can reflect lack of matching epigenomic data rather than true absence of regulatory conservation.
- No changelog found for the repository, so version-specific behavior of mapping outputs cannot be traced.

## Evidence

- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence"
- [intro] partition by mapping multiplicity: "existing methods for mapping non-coding genomic regions across species are challenged by the multi-mapping, low precision, and low mapping rate issues"
- [intro] score threshold 0.5: "We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function."
- [readme] filter parameters: "gr_list_filter <- gr_candidate_filter(
    gr_list,
    best_k = 1L,
    threshold = 0.5
)"
