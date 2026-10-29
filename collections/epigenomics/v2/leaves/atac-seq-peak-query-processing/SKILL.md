---
name: atac-seq-peak-query-processing
description: Use when a set of genomic intervals (e.g. mouse mm10 ATAC-seq peaks)
  must be mapped to orthologous candidate target regions in another species (e.g.
  human hg38) with epigenomic and sequence-grammar prioritization.
when_to_use_negative:
- When only syntentic position mapping is needed without regulatory-information prioritization
  — plain UCSC liftOver suffices, and AdaLiftOver's similarity scoring and filtering
  add unneeded complexity
- When no orthologous epigenome repertoire or chain file exists for the organism pair
  (e.g. non-model organisms lacking matched ENCODE-style datasets)
- When the query intervals are already feature tables or per-peak signal matrices
  rather than genomic intervals — gr_candidate_filter() and adaptive_liftover() operate
  on GRanges input
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: AdaLiftOver
  role: R package providing adaptive_liftover(), compute_similarity_epigenome(), compute_similarity_grammar(),
    gr_candidate_filter(), and training_module() for mapping and prioritizing candidate
    target regions
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Underlying framework used by adaptive_liftover() to extend query regions and
    generate candidate target regions via the reciprocal-best chain file
- name: rtracklayer
  role: Imports the UCSC chain file with import.chain() for use in adaptive_liftover()
- name: motifmatchr
  role: Performs fast motif scanning of query and candidate regions to compute sequence
    grammar similarities
  repo: https://github.com/GreenleafLab/motifmatchr
- name: GenomicRanges
  role: Operates on genomic intervals for query regions, candidates, and epigenome
    peak sets
- name: data.table
  role: Fast data manipulation and computation, e.g. rbindlist() over per-tissue training
    results
- name: Matrix
  role: Efficient matrix operations for similarity computations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:c872b6ef5616ace7779e4e7c4a7d350720404895fab50db30283c6b336fdd118
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/keleslab/AdaLiftOver
  tools:
  - AdaLiftOver
  - GenomicRanges
  - Matrix
  - R
  - UCSC liftOver
  - data.table
  - motifmatchr
  - rtracklayer
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 22
---

# atac-seq-peak-query-processing

## Summary

Process ATAC-seq peak query regions (e.g. 46,676 mouse pancreatic islet peaks) through AdaLiftOver's adaptive_liftover(), similarity computation, logistic-score aggregation, and gr_candidate_filter() to obtain a scored, filtered list of orthologous candidate target regions across species.

## When to use

Use when a set of genomic intervals (e.g. mouse mm10 ATAC-seq peaks) must be mapped to orthologous candidate target regions in another species (e.g. human hg38) with epigenomic and sequence-grammar prioritization.

## When NOT to use

- When only syntentic position mapping is needed without regulatory-information prioritization — plain UCSC liftOver suffices, and AdaLiftOver's similarity scoring and filtering add unneeded complexity
- When no orthologous epigenome repertoire or chain file exists for the organism pair (e.g. non-model organisms lacking matched ENCODE-style datasets)
- When the query intervals are already feature tables or per-peak signal matrices rather than genomic intervals — gr_candidate_filter() and adaptive_liftover() operate on GRanges input

## Inputs

- Query regions as GRanges (e.g. mouse pancreatic islet ATAC-seq peaks, mm10)
- UCSC reciprocal-best chain file (mm10.hg38.rbest.chain.gz, unzipped and imported via rtracklayer::import.chain)
- Orthologous epigenome repertoires as GRangesList (epigenome_mm10, epigenome_hg38; 67 matched ENCODE TF/DNA methylation ChIP-seq and DNase-seq datasets)
- JASPAR position frequency matrices (jaspar_pfm_list) for sequence grammar similarity
- BSgenome objects for both species (e.g. BSgenome.Mmusculus.UCSC.mm10, BSgenome.Hsapiens.UCSC.hg38)
- Optionally, labeled candidate target regions and true human peaks for training_module() parameter refitting

## Outputs

- GRanges list of scored candidate target regions per query region with epigenomic signal similarity, sequence grammar similarity, and logistic probability scores
- Filtered candidate target region list after gr_candidate_filter() (best_k and threshold applied)
- Evaluation metrics (AUROC, AUPR) and scored candidates (e.g. islet_liftover_evaluation.csv) when parameter settings are compared
- Estimated logistic regression parameters and suggested score threshold from training_module()

## How to apply

Load the query regions as GRanges (e.g. 46,676 mouse pancreatic islet ATAC-seq peaks) and extend/map them to human candidate target regions with adaptive_liftover() using the mm10.hg38 reciprocal-best UCSC chain file (mm10.hg38.rbest.chain.gz) at a 2 kb window size. Compute epigenomic signal similarities between each query region and its candidates using the 67 matched ENCODE human-mouse TF/DNA methylation ChIP-seq and DNase-seq datasets (epigenome_mm10, epigenome_hg38), then compute sequence grammar similarities with fast motif scanning via motifmatchr and the jaspar_pfm_list motifs. Aggregate the epigenomic signal and sequence grammar similarities into logistic probability scores and filter candidates with gr_candidate_filter() using best_k (retain at most best k candidates per query region) and a score threshold (e.g. threshold = 0.5, best_k = 1). The default logistic parameters come from LOOCV over the 67 ENCODE epigenome datasets; if labeled data exist for a new organism/tissue (e.g. islet), refit parameters with the training_module() and verify they agree with the defaults (islet example: default AUROC 0.880 / AUPR 0.879 vs refit AUROC 0.879 / AUPR 0.878).

## Related tools

- **AdaLiftOver** (R package providing adaptive_liftover(), compute_similarity_epigenome(), compute_similarity_grammar(), gr_candidate_filter(), and training_module() for mapping and prioritizing candidate target regions) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Underlying framework used by adaptive_liftover() to extend query regions and generate candidate target regions via the reciprocal-best chain file)
- **rtracklayer** (Imports the UCSC chain file with import.chain() for use in adaptive_liftover())
- **motifmatchr** (Performs fast motif scanning of query and candidate regions to compute sequence grammar similarities) — https://github.com/GreenleafLab/motifmatchr
- **GenomicRanges** (Operates on genomic intervals for query regions, candidates, and epigenome peak sets)
- **data.table** (Fast data manipulation and computation, e.g. rbindlist() over per-tissue training results)
- **Matrix** (Efficient matrix operations for similarity computations)

## Examples

```
gr_list <- adaptive_liftover(gr, chain); gr_list <- compute_similarity_epigenome(gr, gr_list, epigenome_mm10, epigenome_hg38); gr_list <- compute_similarity_grammar(gr, gr_list, "mm10", "hg38", jaspar_pfm_list); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- Each query region yields a scored, filtered candidate list: at most best_k candidate target regions per query region, all above the score threshold
- Under default LOOCV-trained parameters, islet mapping metrics fall in the expected range (AUROC ≈ 0.880, AUPR ≈ 0.879); refit parameters should agree closely (AUROC 0.879, AUPR 0.878)
- Similarity columns (epigenomic signal similarity, sequence grammar similarity, logistic probability score) are populated and finite for all retained candidates
- In LOOCV, the epigenome dataset being evaluated is excluded from the epigenomic signal similarity calculations
- Filtered output is a valid GRanges object with candidate target coordinates on the target genome (hg38) and no candidates below the score threshold

## Limitations

- Requires an orthologous epigenome repertoire and reciprocal-best UCSC chain file for the species pair; for other model organisms (e.g. rat), repertoires must be collected from scratch and parameters refit via training_module()
- Existing sequence-conservation methods for mapping non-coding regions across species suffer from multi-mapping, low precision, and low mapping rate issues; the training module suggests a score threshold 'without filtering out too many candidate target regions' (max_filter_proportion, e.g. 0.5)
- Default logistic parameters are derived from LOOCV with the 67 ENCODE datasets; they may need refitting for organism/tissue pairs unlike the training repertoire
- No changelog found for the repository per the discussion-section scan

## Evidence

- [intro] For each query region, AdaLiftOver outputs a scored and filtered list of candidate target regions that are most similar to the query region in terms of regulatory information.: "For each query region, AdaLiftOver outputs a scored and filtered list of candidate target regions that are most similar to the query region in terms of regulatory information."
- [intro] gr_candidate_filter best_k and threshold: "**best_k**: retain at most best k candidate target regions for each query region.
* **threshold**: the score threshold for all the candidate target regions."
- [readme] README quick start with adaptive_liftover and gr_candidate_filter: "gr_list_filter <- gr_candidate_filter(
    gr_list,
    best_k = 1L,
    threshold = 0.5
)"
- [intro] Training module purpose: "AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions."
- [intro] LOOCV exclusion of the evaluated dataset: "To conduct LOOCV, we first apply AdaLiftOver to map each rat epigenome dataset to human while excluding the current dataset for epigenomic signal similarity calculations."
