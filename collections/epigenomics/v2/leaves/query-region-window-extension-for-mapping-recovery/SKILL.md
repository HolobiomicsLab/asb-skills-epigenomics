---
name: query-region-window-extension-for-mapping-recovery
description: Use when a query genomic region fails to map directly between species
  with UCSC liftOver and you need to recover candidate orthologous target regions
  via local window extension before scoring.
when_to_use_negative:
- Every query region maps directly with standard UCSC liftOver and only a 1:1 coordinate
  conversion is needed — plain liftOver/rtracklayer suffices without extension or
  scoring
- You need whole-genome synteny block mapping rather than high-resolution orthologous
  non-coding regulatory elements
- No UCSC chain file or matched orthologous functional genomic datasets are available
  for the species pair, so candidates cannot be generated or scored
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: AdaLiftOver
  role: R package implementing the adaptive two-step liftOver (direct pass, then window-extension
    pass) via adaptive_liftover()
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Coordinate conversion framework applied both directly and to the extended
    query window to produce candidate target regions
- name: rtracklayer
  role: Imports the UCSC chain file (import.chain) and runs the liftOver passes inside
    R
- name: GenomicRanges
  role: Represents query regions and candidate target regions as GRanges/GRangesList
    objects and supports interval extension
- name: motifmatchr
  role: Fast motif scanning to compute sequence grammar similarities between query
    regions and candidates
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:df1ce6d10dfcd7f7ab054535d99e99375a2c18f1ffff180c446f5e9f3334d673
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# query-region-window-extension-for-mapping-recovery

## Summary

AdaLiftOver's adaptive mapping strategy: when UCSC liftOver fails to map a query region directly between species (e.g. mm10 to hg38 via mm10.hg38.rbest.chain), the query region is extended with a local window on both sides and liftOver is re-applied to recover candidate target regions, which are then prioritized by epigenomic and sequence grammar similarity.

## When to use

Use when a query genomic region fails to map directly between species with UCSC liftOver and you need to recover candidate orthologous target regions via local window extension before scoring.

## When NOT to use

- Every query region maps directly with standard UCSC liftOver and only a 1:1 coordinate conversion is needed — plain liftOver/rtracklayer suffices without extension or scoring
- You need whole-genome synteny block mapping rather than high-resolution orthologous non-coding regulatory elements
- No UCSC chain file or matched orthologous functional genomic datasets are available for the species pair, so candidates cannot be generated or scored

## Inputs

- Query genomic regions as a GenomicRanges object (e.g. mouse mm10 peaks)
- UCSC reciprocal-best chain file (e.g. mm10.hg38.rbest.chain.gz or rn6.hg38.rbest.chain.gz), imported with rtracklayer::import.chain
- Orthologous epigenome repertoires as GRangesList (e.g. epigenome_mm10 and epigenome_hg38)
- BSgenome objects for source and target genomes (e.g. BSgenome.Mmusculus.UCSC.mm10, BSgenome.Hsapiens.UCSC.hg38)
- jaspar_pfm_list for motif scanning

## Outputs

- A list (GRangesList) of candidate target regions per query region, with epigenomic and sequence grammar similarity scores
- A scored and filtered list of candidate target regions per query after gr_candidate_filter() (best_k, threshold)

## How to apply

Load the query regions as a GenomicRanges object and import the reciprocal-best UCSC chain file (e.g. mm10.hg38.rbest.chain.gz via rtracklayer::import.chain). Run the first UCSC liftOver pass through rtracklayer; for query regions that fail this direct pass, extend each with a local window on both sides and re-run liftOver on the extended region to generate candidate target regions. Rank these candidates using epigenomic signal similarities from matched orthologous ENCODE functional genomic datasets (67 matched human-mouse TF/DNA methylation ChIP-seq and DNase-seq) and sequence grammar similarities from motifmatchr scanning against jaspar_pfm_list, aggregated into logistic regression probability scores. The rationale is that direct mapping loses hard-to-map non-coding regions, so extension plus scoring recovers functionally similar orthologs that conservation-only methods miss due to multi-mapping and low mapping rate.

## Related tools

- **AdaLiftOver** (R package implementing the adaptive two-step liftOver (direct pass, then window-extension pass) via adaptive_liftover()) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Coordinate conversion framework applied both directly and to the extended query window to produce candidate target regions)
- **rtracklayer** (Imports the UCSC chain file (import.chain) and runs the liftOver passes inside R)
- **GenomicRanges** (Represents query regions and candidate target regions as GRanges/GRangesList objects and supports interval extension)
- **motifmatchr** (Fast motif scanning to compute sequence grammar similarities between query regions and candidates)

## Examples

```
chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain)
```

## Evaluation signals

- Query regions that failed the direct liftOver pass return non-empty candidate target regions after the window-extension pass
- Output is a list keyed by query region (one entry per query), where each entry contains scored candidates from adaptive_liftover()
- Candidates for a given query are consistent with liftOver of its extended region under the same chain file (interval plausibility check)
- Logistic probability scores for retained candidates pass the suggested threshold without filtering out too many candidates (training_module suggests the optimal threshold; max_filter_proportion = 0.5 in the rat LOOCV example)
- gr_candidate_filter retains at most best_k (e.g. best_k = 1L) candidates per query region at the chosen threshold (e.g. threshold = 0.5)

## Limitations

- Existing sequence-conservation methods for mapping non-coding regions across species suffer from multi-mapping, low precision, and low mapping rate issues — extension alone does not resolve these without epigenomic/grammar scoring
- For organisms other than mouse, a user-defined orthologous epigenome repertoire and species-specific BSgenome and chain file must be collected, and logistic regression parameters learned via the training module
- The epigenomic similarity step depends on matched orthologous datasets, so datasets used for scoring must be excluded during leave-one-out cross validation to avoid circularity

## Evidence

- [abstract] We develop Adaptive liftOver (AdaLiftOver), a large-scale computational tool for identifying functionally similar orthologous non-coding regions across species.: "We develop Adaptive liftOver (AdaLiftOver), a large-scale computational tool for identifying functionally similar orthologous non-coding regions across species."
- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions"
- [intro] For each query region, AdaLiftOver outputs a scored and filtered list of candidate target regions that are most similar to the query region in terms of regulatory information.: "For each query region, AdaLiftOver outputs a scored and filtered list of candidate target regions that are most similar to the query region in terms of regulatory information."
- [abstract] existing methods for mapping non-coding genomic regions across species are challenged by the multi-mapping, low precision, and low mapping rate issues: "existing methods for mapping non-coding genomic regions across species are challenged by the multi-mapping, low precision, and low mapping rate issues"
- [readme] ## map the query regions
gr_list <- adaptive_liftover(gr, chain): "gr_list <- adaptive_liftover(gr, chain)"
