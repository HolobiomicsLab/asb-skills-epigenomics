---
name: epigenomic-sequence-conservation-scoring
description: Use when mapping non-coding regulatory regions (e.g. ATAC-seq, ChIP-seq,
  DNase-seq peaks) across species and sequence-based liftOver alone suffers from multi-mapping,
  low precision, or low mapping rate.
when_to_use_negative:
- When the query regions are coding or the goal is pure sequence alignment rather
  than regulatory element orthology — existing sequence-conservation methods already
  address this and AdaLiftOver targets non-coding regulatory regions
- When no matched orthologous epigenomic datasets exist for the species pair and no
  training pair is available to learn logistic regression parameters via training_module()
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: AdaLiftOver
  role: 'Core R package: adaptive_liftover(), compute_similarity_epigenome(), compute_similarity_grammar(),
    gr_candidate_filter(), and training_module() implement the scoring pipeline'
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver / rtracklayer
  role: liftOver framework used by adaptive_liftover() to extend query regions; import.chain()
    loads the UCSC chain file
- name: GenomicRanges
  role: Represents and operates on query regions, candidate target regions, and epigenome
    repertoires as GRanges/GRangesList objects
- name: motifmatchr
  role: Fast motif scanning used to compute sequence grammar similarities
  repo: https://github.com/GreenleafLab/motifmatchr
- name: data.table
  role: Fast data manipulation and computation; rbindlist() aggregates per-tissue
    training results
- name: Matrix
  role: Efficient matrix operations in similarity computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:e535ab179b00cdb44e3d9ffb9898043a4099d4b03f84b6a65fe33a68641fd749
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/keleslab/AdaLiftOver
  tools:
  - AdaLiftOver
  - GenomicRanges
  - Matrix
  - R
  - UCSC liftOver / rtracklayer
  - data.table
  - motifmatchr
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# epigenomic-sequence-conservation-scoring

## Summary

Prioritize orthologous non-coding regions across species by combining UCSC liftOver candidate target regions with logistic probability scores aggregating epigenomic signal similarity and sequence grammar similarity, so that regulatory (epigenomic) evidence can support or override sequence-based mapping.

## When to use

Use when mapping non-coding regulatory regions (e.g. ATAC-seq, ChIP-seq, DNase-seq peaks) across species and sequence-based liftOver alone suffers from multi-mapping, low precision, or low mapping rate.

## When NOT to use

- When the query regions are coding or the goal is pure sequence alignment rather than regulatory element orthology — existing sequence-conservation methods already address this and AdaLiftOver targets non-coding regulatory regions
- When no matched orthologous epigenomic datasets exist for the species pair and no training pair is available to learn logistic regression parameters via training_module()

## Inputs

- Query genomic regions as a GRanges object (e.g. human hg38 ATAC-seq peaks)
- UCSC chain file (e.g. mm10.hg38.rbest.chain.gz or rn6.hg38.rbest.chain.gz, unzipped and imported with rtracklayer::import.chain)
- Orthologous epigenome repertoires as GRangesList (e.g. epigenome_mm10/epigenome_hg38, or epigenome_rn6_test/epigenome_hg38_test)
- BSgenome objects for both species (e.g. BSgenome.Hsapiens.UCSC.hg38, BSgenome.Mmusculus.UCSC.mm10)
- JASPAR position frequency matrices (jaspar_pfm_list) for motif scanning

## Outputs

- Scored and filtered list of candidate target regions per query region (gr_list of GRanges with logistic probability scores)
- Partitioned mapping scores by sequence-based mapping multiplicity (e.g. Group I unique mappers vs Group II multi-mappers), e.g. written to adaliftover_group_scores.csv
- Fraction of uniquely mapped query regions with mapping scores below threshold (e.g. 0.5)
- Training results with estimated logistic regression parameters and suggested score threshold

## How to apply

Load the query regions (e.g. human hg38 brain ATAC-seq peaks) as a GRanges object and import the relevant UCSC reciprocal-best chain file (e.g. hg38-to-zebrafish or mm10.hg38.rbest.chain) with rtracklayer::import.chain. Run adaptive_liftover() to extend queries via the liftOver framework and obtain scored candidate target regions, then compute epigenomic signal similarities from matched orthologous functional genomic datasets with compute_similarity_epigenome() and sequence grammar similarities with compute_similarity_grammar() using jaspar_pfm_list motifs. Aggregate the two similarities as logistic probability scores and filter with gr_candidate_filter() (best_k candidates per query, score threshold, e.g. 0.5); for non-ENCODE organisms use training_module() on a pair of matched epigenome datasets to estimate logistic regression parameters and suggest an optimal threshold without filtering out too many candidates. Rationale: mapping multiplicity partitions queries into unique vs multi-mapper groups, and low scores (e.g. <0.5) among uniquely sequence-mapped queries quantify where epigenomic profiles contradict sequence-based conservation.

## Related tools

- **AdaLiftOver** (Core R package: adaptive_liftover(), compute_similarity_epigenome(), compute_similarity_grammar(), gr_candidate_filter(), and training_module() implement the scoring pipeline) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver / rtracklayer** (liftOver framework used by adaptive_liftover() to extend query regions; import.chain() loads the UCSC chain file)
- **GenomicRanges** (Represents and operates on query regions, candidate target regions, and epigenome repertoires as GRanges/GRangesList objects)
- **motifmatchr** (Fast motif scanning used to compute sequence grammar similarities) — https://github.com/GreenleafLab/motifmatchr
- **data.table** (Fast data manipulation and computation; rbindlist() aggregates per-tissue training results)
- **Matrix** (Efficient matrix operations in similarity computation)

## Examples

```
gr_list <- adaptive_liftover(gr, chain); gr_list <- compute_similarity_epigenome(gr, gr_list, epigenome_mm10, epigenome_hg38); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- Mapping multiplicity partition holds: Group I queries map uniquely by sequence conservation, Group II queries are multi-mappers; the Group I low-score fraction (scores <0.5, e.g. 16.4% for zebrafish brain ATAC-seq) is reproducible when re-run on the same chain and peak sets
- gr_candidate_filter() respects both parameters: at most best_k candidate target regions retained per query and all candidates below the score threshold (e.g. 0.5) removed
- Positives/negatives in the training module are correctly labeled: human candidates overlapping the corresponding human epigenome peaks are positives, others negatives; LOOCV excludes the current dataset from epigenomic signal similarity calculations
- Output candidate regions fall within liftOver-provided target coordinates for the chain used (e.g. hg38-to-zebrafish or mm10.hg38.rbest.chain) and scores lie in [0,1] as logistic probabilities
- training_module() suggests a threshold without filtering out too many candidate target regions (max_filter_proportion respected, e.g. 0.5 in the rat example)

## Limitations

- For organisms other than mouse/rat/human, a user-defined orthologous epigenome repertoire and BSgenome must be collected from scratch, and logistic regression parameters must be learned via training_module()
- Installation depends on several non-CRAN dependencies (motifmatchr from GitHub, rtracklayer/GenomicRanges/BSgenome via BiocManager), which can fail without BiocManager
- No changelog or discussion content is available for the repository, so version history and known issues are undocumented
- Epigenomic signal similarity requires matched orthologous functional genomic datasets (e.g. ENCODE 67 matched human-mouse TF/DNA methylation ChIP-seq and DNase-seq, or ChIP-Atlas H3K4me3 for human/rat tissues)

## Evidence

- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence"
- [intro] We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function.: "We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function."
- [intro] AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions.: "AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions."
- [intro] **best_k**: retain at most best k candidate target regions for each query region.
* **threshold**: the score threshold for all the candidate target regions.: "**best_k**: retain at most best k candidate target regions for each query region."
- [readme] gr_list_filter <- gr_candidate_filter(
    gr_list,
    best_k = 1L,
    threshold = 0.5
): "gr_list_filter <- gr_candidate_filter(
    gr_list,
    best_k = 1L,
    threshold = 0.5
)"
- [readme] For each query region, AdaLiftOver outputs a scored and filtered list of candidate target regions that are most similar to the query region in terms of regulatory information.: "For each query region, AdaLiftOver outputs a scored and filtered list of candidate target regions that are most similar to the query region in terms of regulatory information."
