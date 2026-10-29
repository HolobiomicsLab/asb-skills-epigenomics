---
name: epigenomic-similarity-based-candidate-filtering
description: Use when UCSC liftOver or adaptive_liftover has produced many candidate
  orthologous target regions per query interval and you need to prioritize them by
  functional regulatory similarity rather than sequence conservation alone.
when_to_use_negative:
- When the query regions are coding regions or whole genes rather than candidate non-coding
  regulatory elements — AdaLiftOver targets orthologous regulatory elements via epigenomic
  signal and sequence grammar conservation.
- When no matched orthologous functional genomic (epigenome) datasets exist for the
  species pair and no training data is available to estimate logistic regression parameters.
- When simple synteny/sequence-conservation mapping suffices and multi-mapping ambiguity
  is not a concern.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3295
tools:
- name: AdaLiftOver
  role: R package providing compute_similarity_epigenome, compute_similarity_grammar,
    gr_candidate_filter, and training_module for scoring and filtering candidate orthologous
    target regions
  repo: https://github.com/keleslab/AdaLiftOver
- name: rtracklayer
  role: Imports the UCSC reciprocal-best chain file to generate candidate target regions
    prior to filtering
- name: GenomicRanges
  role: Operates on the genomic intervals of query and candidate target regions
- name: motifmatchr
  role: Fast motif scanning used to compute sequence grammar similarities between
    query and candidate regions
  repo: https://github.com/GreenleafLab/motifmatchr
- name: data.table
  role: Fast data manipulation and computation for similarity aggregation and result
    assembly
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:92393890a62a22dd01a61a160af364a4cd656b6020244547087ba860e5cd7fe7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/keleslab/AdaLiftOver
  tools:
  - AdaLiftOver
  - GenomicRanges
  - R
  - data.table
  - motifmatchr
  - rtracklayer
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# epigenomic-similarity-based-candidate-filtering

## Summary

A filtering skill from AdaLiftOver that prioritizes liftOver candidate target regions by aggregating epigenomic signal similarities (from 67 matched ENCODE human-mouse TF/DNA methylation ChIP-seq and DNase-seq datasets) and sequence grammar similarities into logistic probability scores, then retaining at most best_k regions per query above a score threshold.

## When to use

Use when UCSC liftOver or adaptive_liftover has produced many candidate orthologous target regions per query interval and you need to prioritize them by functional regulatory similarity rather than sequence conservation alone.

## When NOT to use

- When the query regions are coding regions or whole genes rather than candidate non-coding regulatory elements — AdaLiftOver targets orthologous regulatory elements via epigenomic signal and sequence grammar conservation.
- When no matched orthologous functional genomic (epigenome) datasets exist for the species pair and no training data is available to estimate logistic regression parameters.
- When simple synteny/sequence-conservation mapping suffices and multi-mapping ambiguity is not a concern.

## Inputs

- GRangesList of candidate target regions per query region from adaptive_liftover
- GRangesList orthologous epigenome repertoires for both species (e.g. epigenome_mm10, epigenome_hg38 from 67 matched ENCODE datasets)
- Jaspar PFM list (jaspar_pfm_list) for motif/sequence grammar scanning via motifmatchr
- BSgenome objects for both species (e.g. BSgenome.Mmusculus.UCSC.mm10, BSgenome.Hsapiens.UCSC.hg38)
- Optional: a pair of matched candidate/true epigenome peak sets (gr_candidate, gr_true) for training_module logistic regression parameter estimation

## Outputs

- GRangesList of filtered, scored candidate target regions (logistic probability score per candidate)
- Optional training_result data.table of averaged logistic regression parameters and suggested optimal score threshold
- Per-query orthologous mappings (e.g. bmd_snp_adaliftover_results.csv for BMD GWAS SNPs)

## How to apply

After mapping query regions (e.g. hg38 GWAS SNP intervals) to candidate target regions with adaptive_liftover using a reciprocal-best chain file such as mm10.hg38.rbest.chain, compute epigenomic signal similarities between each query region and its candidates with compute_similarity_epigenome() over matched orthologous epigenome repertoires (e.g. epigenome_mm10 and epigenome_hg38 from ENCODE). Next compute sequence grammar similarities with compute_similarity_grammar() using Jaspar PFMs (jaspar_pfm_list) and the appropriate BSgenome objects. Aggregate the epigenome signal and grammar similarities as logistic probability scores — parameters can be estimated with training_module() on a pair of matched epigenome datasets (e.g. rat example with max_filter_proportion = 0.5, interaction = FALSE). Finally apply gr_candidate_filter() with best_k (retain at most best k candidate target regions per query region) and a threshold (score cutoff for all candidate target regions; the README example uses best_k = 1L and threshold = 0.5).

## Related tools

- **AdaLiftOver** (R package providing compute_similarity_epigenome, compute_similarity_grammar, gr_candidate_filter, and training_module for scoring and filtering candidate orthologous target regions) — https://github.com/keleslab/AdaLiftOver
- **rtracklayer** (Imports the UCSC reciprocal-best chain file to generate candidate target regions prior to filtering)
- **GenomicRanges** (Operates on the genomic intervals of query and candidate target regions)
- **motifmatchr** (Fast motif scanning used to compute sequence grammar similarities between query and candidate regions) — https://github.com/GreenleafLab/motifmatchr
- **data.table** (Fast data manipulation and computation for similarity aggregation and result assembly)

## Examples

```
gr_list <- compute_similarity_epigenome(gr, gr_list, epigenome_mm10, epigenome_hg38); gr_list <- compute_similarity_grammar(gr, gr_list, "mm10", "hg38", jaspar_pfm_list); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- Each query region retains at most best_k candidate target regions and all retained candidates have scores above the threshold (README example: best_k = 1L, threshold = 0.5).
- Positive-control checks map as expected, e.g. human SNP rs9486902 at the FOXO3 promoter maps to the mouse Foxo3 promoter region.
- Filtered mappings yield expected gene associations, e.g. 90 BMD genes for the expanded UK Biobank BMD GWAS SNP set (PIP >= 0.001) with a 65.2% precision increase over UCSC liftOver.
- Training-module output suggests an optimal score threshold without filtering out too many candidate target regions (max_filter_proportion bounds the filtered fraction).
- In LOOCV (e.g. rat), candidates labeled positive overlap the corresponding human epigenome peaks for the matched tissue.

## Limitations

- Requires matched orthologous functional genomic datasets between the two species; for non-mouse model organisms the user must collect an epigenome repertoire from scratch and re-train logistic regression parameters with training_module().
- Existing sequence-conservation-based mapping of non-coding regions suffers from multi-mapping, low precision, and low mapping rate issues; this filter mitigates but depends on the quality of candidate regions from the chain file.
- Filter parameters (best_k, threshold) are context-specific — thresholds learned for one species pair (e.g. averaged rat parameters) may not transfer directly.

## Evidence

- [intro] We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function.: "We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function."
- [intro] best_k: retain at most best k candidate target regions for each query region. threshold: the score threshold for all the candidate target regions.: "**best_k**: retain at most best k candidate target regions for each query region.
* **threshold**: the score threshold for all the candidate target regions."
- [intro] AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions.: "AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions."
- [readme] gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5): "gr_list_filter <- gr_candidate_filter(
    gr_list,
    best_k = 1L,
    threshold = 0.5
)"
- [intro] We uses 67 matched ENCODE functional genomic datasets including TF/DNA methylation ChIP-seq and DNase-seq between human and mouse: "We uses 67 matched ENCODE functional genomic datasets including TF/DNA methylation ChIP-seq and DNase-seq between human and mouse"
