---
name: orthologous-regulatory-element-mapping
description: Use when you need to lift over orthologous regulatory elements (TF ChIP-seq,
  DNase-seq, DNA methylation peaks) between species, e.g. mouse mm10 to human hg38,
  and reduce multi-mapping with functional-genomic similarity scoring.
when_to_use_negative:
- The species pair lacks matched orthologous epigenomic datasets, so epigenomic signal
  similarity cannot be computed and no training data exist for the training module
- The task is simple synteny-based coordinate conversion where functional prioritization
  is unnecessary (plain UCSC liftOver via rtracklayer suffices)
- Input regions are coding variants or whole-genome coordinate translation rather
  than non-coding regulatory elements
edam_operation: http://edamontology.org/operation_3195
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0622
tools:
- name: AdaLiftOver
  role: 'Core R package: adaptive_liftover extends query regions, compute_similarity_epigenome
    and compute_similarity_grammar score candidates, gr_candidate_filter and training_module
    filter and calibrate'
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Underlying chain-based coordinate mapping framework extended by AdaLiftOver;
    also the baseline comparator in the TF ChIP-seq benchmark
- name: rtracklayer
  role: Imports the UCSC reciprocal-best chain file (import.chain) used by adaptive_liftover
- name: GenomicRanges
  role: Represents and operates on query peaks, candidate target regions, and epigenome
    repertoires as GRanges/GRangesList objects
- name: motifmatchr
  role: Fast motif scanning used to compute sequence grammar similarities between
    query and candidate regions
- name: data.table
  role: Fast data manipulation and computation within the AdaLiftOver pipeline (e.g.
    rbindlist over training results)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:fa75dd332f070c3d5e2b179ae63e7fb45dcdce2bd9a779787953484c116919b6
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
  - motifmatchr
  - rtracklayer
  edam_operation: http://edamontology.org/operation_3195
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# orthologous-regulatory-element-mapping

## Summary

Maps non-coding regulatory regions across species with AdaLiftOver, which extends query regions via the UCSC liftOver framework and prioritizes candidate target regions by epigenomic signal and sequence grammar similarity, then filters them with logistic probability scores.

## When to use

Use when you need to lift over orthologous regulatory elements (TF ChIP-seq, DNase-seq, DNA methylation peaks) between species, e.g. mouse mm10 to human hg38, and reduce multi-mapping with functional-genomic similarity scoring.

## When NOT to use

- The species pair lacks matched orthologous epigenomic datasets, so epigenomic signal similarity cannot be computed and no training data exist for the training module
- The task is simple synteny-based coordinate conversion where functional prioritization is unnecessary (plain UCSC liftOver via rtracklayer suffices)
- Input regions are coding variants or whole-genome coordinate translation rather than non-coding regulatory elements

## Inputs

- Query peak regions as GRanges objects (e.g. mouse mm10 TF ChIP-seq peaks)
- UCSC reciprocal-best chain file (e.g. mm10.hg38.rbest.chain.gz, rn6.hg38.rbest.chain.gz) imported via rtracklayer::import.chain
- Matched orthologous functional genomic datasets as GRangesList (epigenome_mm10 / epigenome_hg38 from 67 matched ENCODE TF/DNA methylation ChIP-seq and DNase-seq datasets)
- JASPAR position frequency matrices (jaspar_pfm_list) and BSgenome objects for both species (BSgenome.Mmusculus.UCSC.mm10, BSgenome.Hsapiens.UCSC.hg38)

## Outputs

- A scored and filtered list of candidate target regions per query region (GRanges with logistic probability scores)
- Lifted orthologous peak sets with precision and true-positive counts when benchmarked (e.g. liftover_benchmark_results.csv)
- Training result table of averaged logistic regression parameters and suggested score threshold from training_module

## How to apply

Load query peaks as GRanges objects and map them with adaptive_liftover using a reciprocal-best UCSC chain file (e.g. mm10.hg38.rbest.chain.gz imported via rtracklayer::import.chain). Compute epigenomic signal similarities against matched orthologous ENCODE functional genomic datasets (epigenome_mm10, epigenome_hg38) and sequence grammar similarities using jaspar_pfm_list motif scanning (motifmatchr) against BSgenome mm10/hg38. Aggregate the two similarities as logistic regression probability scores, then run gr_candidate_filter with best_k (retain at most best_k candidates per query region) and a score threshold. For organisms without pretrained parameters (e.g. rat rn6), estimate logistic regression parameters with training_module from a pair of matched epigenome datasets, labeling human candidates positive when they overlap the corresponding human epigenome peaks, with max_filter_proportion = 0.5. Benchmark against UCSC liftOver by scoring a lifted peak as a true positive when it overlaps the orthologous peak from the matched ENCODE dataset, then compare precision and true-positive counts.

## Related tools

- **AdaLiftOver** (Core R package: adaptive_liftover extends query regions, compute_similarity_epigenome and compute_similarity_grammar score candidates, gr_candidate_filter and training_module filter and calibrate) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Underlying chain-based coordinate mapping framework extended by AdaLiftOver; also the baseline comparator in the TF ChIP-seq benchmark)
- **rtracklayer** (Imports the UCSC reciprocal-best chain file (import.chain) used by adaptive_liftover)
- **GenomicRanges** (Represents and operates on query peaks, candidate target regions, and epigenome repertoires as GRanges/GRangesList objects)
- **motifmatchr** (Fast motif scanning used to compute sequence grammar similarities between query and candidate regions)
- **data.table** (Fast data manipulation and computation within the AdaLiftOver pipeline (e.g. rbindlist over training results))

## Examples

```
library(AdaLiftOver); chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain); gr_list <- compute_similarity_epigenome(gr, gr_list, epigenome_mm10, epigenome_hg38); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- On the 55 matched human-mouse TF ChIP-seq benchmark (erythroid and lymphoblast cells), AdaLiftOver mean precision should be 0.372, >50% above UCSC liftOver
- Mean true positives should be ~4456 for AdaLiftOver versus ~4313 for UCSC liftOver across the 55 TF datasets
- gr_candidate_filter output must retain at most best_k candidate target regions per query region, and all retained candidates must meet the score threshold (e.g. threshold = 0.5 in the quick-start)
- A lifted peak is a true positive only when it overlaps the orthologous peak from the matched ENCODE dataset; precision and TP counts are computed per TF
- Training_module with max_filter_proportion = 0.5 should suggest a score threshold that does not filter out too many candidate target regions (LOOCV on rat rn6 data)

## Limitations

- Requires matched orthologous functional genomic datasets between the two species; for non-model organisms (e.g. rat) a user-defined epigenome repertoire and BSgenome object must be collected from scratch
- Logistic regression parameters are organism- and repertoire-specific; the ENCODE-trained defaults may not transfer, requiring training_module estimation (possibly averaged across tissues)
- Existing liftOver-based mapping suffers from multi-mapping, low precision, and low mapping rate issues when used without functional prioritization

## Evidence

- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions"
- [intro] filter functions: "We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function."
- [intro] best_k and threshold parameters: "**best_k**: retain at most best k candidate target regions for each query region.
* **threshold**: the score threshold for all the candidate target regions."
- [readme] README quick-start invocation: "gr_list <- adaptive_liftover(gr, chain)"
- [readme] README filter call: "gr_list_filter <- gr_candidate_filter(
    gr_list,
    best_k = 1L,
    threshold = 0.5
)"
- [intro] training module rationale: "AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions."
