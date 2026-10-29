---
name: atac-seq-peak-liftover
description: Use when query genomic intervals (e.g., human brain ATAC-seq peaks in
  hg38) must be mapped to another species' genome (e.g., zebrafish rn6 via a UCSC
  reciprocal- chain file) and scored for regulatory conservation rather than mapped
  by sequence alone.
when_to_use_negative:
- When the goal is simple coordinate conversion of well-annotated regions without
  regulatory-score prioritization — plain UCSC liftOver suffices
- When no orthologous functional genomic datasets exist for the target organism to
  compute epigenomic signal similarities and no matched pair is available for the
  training module
- When queries are not genomic intervals (e.g., gene expression tables or SNPs without
  lifted coordinates defined in the chain)
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: AdaLiftOver
  role: Core R package providing adaptive_liftover, compute_similarity_epigenome,
    compute_similarity_grammar, gr_candidate_filter, and training_module for scored
    orthologous-region mapping
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Framework AdaLiftOver builds on to extend query regions into candidate target
    regions via the chain file
  repo: https://github.com/keleslab/AdaLiftOver
- name: rtracklayer
  role: Imports the UCSC chain file (import.chain) used by adaptive_liftover
- name: GenomicRanges
  role: Represents and operates on query peak intervals and candidate target regions
- name: motifmatchr
  role: Fast motif scanning used to compute sequence grammar similarity between query
    and candidate regions
- name: data.table
  role: Fast data manipulation and computation within the mapping pipeline (e.g.,
    rbindlist over tissues in training)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:504e90a9eed7c784226cb7f05576b1bf9717442e462164288a032f1a8b56df0b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# atac-seq-peak-liftover

## Summary

Maps ATAC-seq/DNase peak query regions across species using AdaLiftOver, which extends queries via the UCSC liftOver chain framework and prioritizes candidate orthologous target regions by epigenomic signal and sequence grammar similarity scores.

## When to use

Use when query genomic intervals (e.g., human brain ATAC-seq peaks in hg38) must be mapped to another species' genome (e.g., zebrafish rn6 via a UCSC reciprocal-best chain file) and scored for regulatory conservation rather than mapped by sequence alone.

## When NOT to use

- When the goal is simple coordinate conversion of well-annotated regions without regulatory-score prioritization — plain UCSC liftOver suffices
- When no orthologous functional genomic datasets exist for the target organism to compute epigenomic signal similarities and no matched pair is available for the training module
- When queries are not genomic intervals (e.g., gene expression tables or SNPs without lifted coordinates defined in the chain)

## Inputs

- Query peak regions as GRanges (e.g., human brain ATAC-seq peaks in hg38 BED format)
- UCSC chain file for the source-to-target genome pair (e.g., mm10.hg38.rbest.chain, rn6.hg38.rbest.chain) imported with rtracklayer
- Matched orthologous epigenome repertoire as GRangesList (e.g., data(epigenome_mm10)/data(epigenome_hg38), or ChIP-Atlas H3K4me3 datasets)
- JASPAR PFMs for grammar similarity (data('jaspar_pfm_list'))
- BSgenome objects for both genomes (e.g., BSgenome.Mmusculus.UCSC.mm10, BSgenome.Hsapiens.UCSC.hg38)

## Outputs

- Scored and filtered list of candidate target regions per query (GRanges list) from adaptive_liftover and gr_candidate_filter
- Logistic regression parameters and suggested score threshold from training_module
- Per-region mapping scores enabling partition into unique mappers (Group I) and multi-mappers (Group II), e.g., written to adaliftover_group_scores.csv

## How to apply

Load the query peak set as a GenomicRanges GRanges object and import the relevant UCSC chain file (e.g., hg38-to-zebrafish or mm10.hg38.rbest.chain) with rtracklayer::import.chain(). Run AdaLiftOver::adaptive_liftover(gr, chain) to extend queries through the liftOver framework and obtain scored candidate target regions. Compute epigenomic signal similarity (compute_similarity_epigenome with matched species-specific datasets, e.g., the ENCODE repertoire) and sequence grammar similarity (compute_similarity_grammar with jaspar_pfm_list motif scanning via motifmatchr). Filter candidates with gr_candidate_filter(best_k, threshold) — e.g., best_k=1L and threshold=0.5 as in the vignette — and, for unmodeled organisms like rat or zebrafish, fit logistic regression parameters with training_module(gr_candidate, gr_true) using matched orthologous epigenome datasets, capped by max_filter_proportion (0.5 in the README) so filtering does not discard too many candidates. Finally, partition mapped queries by mapping multiplicity (unique mappers vs multi-mappers) and quantify the fraction of uniquely mapped regions whose AdaLiftOver score falls below 0.5 to detect epigenomic overriding of sequence-based mappability.

## Related tools

- **AdaLiftOver** (Core R package providing adaptive_liftover, compute_similarity_epigenome, compute_similarity_grammar, gr_candidate_filter, and training_module for scored orthologous-region mapping) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Framework AdaLiftOver builds on to extend query regions into candidate target regions via the chain file) — https://github.com/keleslab/AdaLiftOver
- **rtracklayer** (Imports the UCSC chain file (import.chain) used by adaptive_liftover)
- **GenomicRanges** (Represents and operates on query peak intervals and candidate target regions)
- **motifmatchr** (Fast motif scanning used to compute sequence grammar similarity between query and candidate regions)
- **data.table** (Fast data manipulation and computation within the mapping pipeline (e.g., rbindlist over tissues in training))

## Examples

```
library(AdaLiftOver); chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- adaptive_liftover returns candidate target regions only within chain-mappable loci; every query should either yield candidates or be flagged as unmapped
- After gr_candidate_filter, at most best_k candidates per query remain and all retained scores exceed the threshold (e.g., threshold=0.5)
- training_module output should estimate logistic regression parameters and suggest a threshold that does not filter out too many candidates (max_filter_proportion, e.g., 0.5)
- For the zebrafish prioritization analysis, the fraction of uniquely (Group I) mapped queries with scores <0.5 should be recomputable from the output CSV (reported as 16.4%)
- Replicate the README quick-start order: adaptive_liftover → compute_similarity_epigenome → compute_similarity_grammar → gr_candidate_filter; skipping a similarity step yields no logistic probability scores

## Limitations

- Existing sequence-conservation methods suffer from multi-mapping, low precision, and low mapping rate issues; multi-mapped queries must be partitioned (Group II) and interpreted cautiously
- For model organisms other than mouse, the user must collect an orthologous epigenome repertoire from scratch and install the corresponding BSgenome, or default parameters will not apply
- Score threshold choice trades off candidate retention vs precision; AdaLiftOver suggests an optimal threshold 'without filtering out too many candidate target regions' rather than fixing it
- No changelog found for the repository (discussion section content is absent), so version-specific behavior cannot be traced

## Evidence

- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions"
- [intro] For each query region, AdaLiftOver outputs a scored and filtered list of candidate target regions that are most similar to the query region in terms of regulatory information.: "For each query region, AdaLiftOver outputs a scored and filtered list of candidate target regions that are most similar to the query region in terms of regulatory information."
- [intro] best_k: retain at most best k candidate target regions for each query region; threshold: the score threshold for all the candidate target regions.: "**best_k**: retain at most best k candidate target regions for each query region."
- [intro] AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions.: "AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions."
- [readme] chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain): "chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain")"
