---
name: orthologous-region-mapping-chip-seq
description: Use when you need to identify orthologous non-coding regulatory elements
  across species (e.g., human-mouse or rat-human ENCODE ChIP-seq/DNase-seq peaks)
  and existing sequence-conservation liftOver suffers from multi-mapping, low precision,
  or low mapping rate.
when_to_use_negative:
- The query regions are coding sequences or the species pair has no available UCSC
  reciprocal-best chain file or BSgenome object.
- No orthologous epigenomic datasets exist for the species pair, so epigenomic signal
  similarity cannot be computed and parameters cannot be trained.
- The task is simple synteny-based coordinate conversion of uniquely mappable regions
  where candidate prioritization is unnecessary — plain UCSC liftOver suffices.
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: AdaLiftOver
  role: R package providing adaptive_liftover, compute_similarity_epigenome, compute_similarity_grammar,
    gr_candidate_filter, and training_module for cross-species orthologous region
    mapping
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Framework used to extend query regions into candidate target regions via the
    chain file
- name: rtracklayer
  role: Imports the UCSC chain file with import.chain for the liftOver step
- name: GenomicRanges
  role: Operating on genomic intervals (query regions, candidate target regions, epigenome
    repertoires)
- name: motifmatchr
  role: Fast motif scanning for computing sequence grammar similarities
  repo: https://github.com/GreenleafLab/motifmatchr
- name: data.table
  role: Fast data manipulation and computation (e.g., rbindlist of training results
    per tissue)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:09b8305086b425c508a9246c82aaf1a13925090d9f404efc1ca526f7fba596f1
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
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 20
---

# orthologous-region-mapping-chip-seq

## Summary

Maps query regulatory regions (e.g., ChIP-seq / DNase-seq peaks) from one species to orthologous target regions in another using AdaLiftOver, which extends query regions via the UCSC liftOver framework and prioritizes candidates by epigenomic signal and sequence grammar similarity.

## When to use

Use when you need to identify orthologous non-coding regulatory elements across species (e.g., human-mouse or rat-human ENCODE ChIP-seq/DNase-seq peaks) and existing sequence-conservation liftOver suffers from multi-mapping, low precision, or low mapping rate.

## When NOT to use

- The query regions are coding sequences or the species pair has no available UCSC reciprocal-best chain file or BSgenome object.
- No orthologous epigenomic datasets exist for the species pair, so epigenomic signal similarity cannot be computed and parameters cannot be trained.
- The task is simple synteny-based coordinate conversion of uniquely mappable regions where candidate prioritization is unnecessary — plain UCSC liftOver suffices.

## Inputs

- Query GRanges of regulatory regions (e.g., ChIP-seq/DNase-seq peaks) in the source genome (e.g., mm10, rn6)
- UCSC reciprocal-best chain file (e.g., mm10.hg38.rbest.chain.gz or rn6.hg38.rbest.chain.gz)
- Orthologous epigenome repertoires as GRangesList objects (e.g., epigenome_mm10, epigenome_hg38, epigenome_rn6_test, epigenome_hg38_test)
- BSgenome objects for both species (e.g., BSgenome.Mmusculus.UCSC.mm10, BSgenome.Hsapiens.UCSC.hg38, BSgenome.Rnorvegicus.UCSC.rn6)
- JASPAR motif PFMs (data('jaspar_pfm_list'))
- Matched orthologous peak sets for training/validation (e.g., 67 matched human-mouse ENCODE TF/DNA methylation ChIP-seq and DNase-seq datasets)

## Outputs

- Scored and filtered list of candidate target regions per query region (GRanges with logistic probability scores)
- Training result table of averaged logistic regression parameters and suggested score threshold
- LOOCV AUROC/AUPR summary per window size (e.g., loocv_results.csv) with mean and standard deviation across folds

## How to apply

Import a UCSC reciprocal-best chain file (e.g., mm10.hg38.rbest.chain) with rtracklayer::import.chain, then run adaptive_liftover on the query GRanges to obtain candidate target regions. Compute epigenomic signal similarities with compute_similarity_epigenome using orthologous epigenome repertoires (GRangesList objects like epigenome_mm10/epigenome_hg38), and sequence grammar similarities with compute_similarity_grammar using BSgenome objects and jaspar_pfm_list motif PFMs. Aggregate the two similarities as logistic probability scores and filter with gr_candidate_filter (best_k retains at most k candidates per query, threshold applies a score cutoff, e.g., best_k=1L, threshold=0.5). For organisms other than mouse, first estimate logistic regression parameters with training_module() on a pair of matched epigenome datasets (max_filter_proportion=0.5, interaction=FALSE) and average the parameters as defaults. Validate by LOOCV: hold out each of the 67 matched ENCODE datasets, exclude it from signal similarity calculations, and count a top-scored candidate as correct if it overlaps the matched orthologous region; the ~2 kb local window is the default (AUROC 0.820 ± 0.0113, AUPR 0.604 ± 0.0371).

## Related tools

- **AdaLiftOver** (R package providing adaptive_liftover, compute_similarity_epigenome, compute_similarity_grammar, gr_candidate_filter, and training_module for cross-species orthologous region mapping) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Framework used to extend query regions into candidate target regions via the chain file)
- **rtracklayer** (Imports the UCSC chain file with import.chain for the liftOver step)
- **GenomicRanges** (Operating on genomic intervals (query regions, candidate target regions, epigenome repertoires))
- **motifmatchr** (Fast motif scanning for computing sequence grammar similarities) — https://github.com/GreenleafLab/motifmatchr
- **data.table** (Fast data manipulation and computation (e.g., rbindlist of training results per tissue))

## Examples

```
chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain); gr_list <- compute_similarity_epigenome(gr, gr_list, epigenome_mm10, epigenome_hg38)
```

## Evaluation signals

- LOOCV on the 67 matched ENCODE human-mouse datasets at the ~2 kb window yields AUROC ≈ 0.820 ± 0.0113 and AUPR ≈ 0.604 ± 0.0371.
- gr_candidate_filter output contains at most best_k candidate target regions per query region, each with a logistic probability score above the threshold (e.g., best_k=1L, threshold=0.5).
- A top-scored candidate is classified as correct only if it overlaps the matched orthologous region of the held-out dataset.
- training_module() returns estimated logistic regression parameters and a suggested score threshold without filtering out too many candidates (max_filter_proportion=0.5).
- Per-window-size AUROC/AUPR table written to loocv_results.csv covers 13 window sizes from 0 to 5 kb in 400 bp increments.

## Limitations

- Requires orthologous epigenomic datasets and UCSC chain files; for organisms other than mouse (e.g., rat), the epigenome repertoire must be collected from scratch and logistic regression parameters must be re-trained and averaged as defaults.
- Sequence-conservation-based cross-species mapping suffers from multi-mapping, low precision, and low mapping rate issues that AdaLiftOver mitigates but cannot fully eliminate.
- No changelog found for the repository.

## Evidence

- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence"
- [intro] For each query region, AdaLiftOver outputs a scored and filtered list of candidate target regions that are most similar to the query region in terms of regulatory information.: "For each query region, AdaLiftOver outputs a scored and filtered list of candidate target regions that are most similar to the query region in terms of regulatory information."
- [intro] **best_k**: retain at most best k candidate target regions for each query region. * **threshold**: the score threshold for all the candidate target regions.: "**best_k**: retain at most best k candidate target regions for each query region. * **threshold**: the score threshold for all the candidate target regions."
- [readme] gr_list_filter <- gr_candidate_filter(
    gr_list,
    best_k = 1L,
    threshold = 0.5
): "gr_list_filter <- gr_candidate_filter(
    gr_list,
    best_k = 1L,
    threshold = 0.5
)"
- [readme] AdaLiftOver provides a handy training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions.: "AdaLiftOver provides a handy training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions."
- [intro] To conduct LOOCV, we first apply AdaLiftOver to map each rat epigenome dataset to human while excluding the current dataset for epigenomic signal similarity calculations.: "To conduct LOOCV, we first apply AdaLiftOver to map each rat epigenome dataset to human while excluding the current dataset for epigenomic signal similarity calculations."
