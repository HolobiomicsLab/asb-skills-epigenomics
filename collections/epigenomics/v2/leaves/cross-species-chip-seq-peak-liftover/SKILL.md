---
name: cross-species-chip-seq-peak-liftover
description: Use when orthologous non-coding regulatory elements (e.g., TF ChIP-seq
  peaks) must be lifted between species and naive chain-file liftOver suffers from
  multi-mapping, low precision, or low mapping rate.
when_to_use_negative:
- The input regions are coding sequences or whole-genome alignments, not non-coding
  regulatory intervals whose orthology depends on regulatory signal
- No orthologous epigenomic datasets exist for the species pair, and no matched data
  is available to train the logistic parameters via training_module
- The task is simple synteny/gene orthology mapping rather than functional orthologous
  regulatory element identification
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: AdaLiftOver
  role: 'Core R package: adaptive_liftover extends query regions, compute_similarity_epigenome
    and compute_similarity_grammar score candidates, gr_candidate_filter and training_module
    filter and calibrate'
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Baseline chain-based cross-species interval mapping framework that AdaLiftOver
    extends; run via rtracklayer for comparison
- name: rtracklayer
  role: Imports UCSC chain files (import.chain) and runs liftOver mapping
- name: GenomicRanges
  role: Represents query peaks and candidate target regions as GRanges/GRangesList
    objects
- name: motifmatchr
  role: Fast motif scanning used for sequence grammar similarity computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:9cb82be295b14be30231273bff3c415fb180f176d20f3d6193048cea1470f8aa
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-species-chip-seq-peak-liftover

## Summary

Maps TF ChIP-seq peak sets across species (e.g., mouse mm10 to human hg38) using AdaLiftOver, which extends query regions via the UCSC liftOver framework and prioritizes candidate target regions by epigenomic signal and sequence grammar similarity, outperforming UCSC liftOver in precision and true-positive recovery.

## When to use

Use when orthologous non-coding regulatory elements (e.g., TF ChIP-seq peaks) must be lifted between species and naive chain-file liftOver suffers from multi-mapping, low precision, or low mapping rate.

## When NOT to use

- The input regions are coding sequences or whole-genome alignments, not non-coding regulatory intervals whose orthology depends on regulatory signal
- No orthologous epigenomic datasets exist for the species pair, and no matched data is available to train the logistic parameters via training_module
- The task is simple synteny/gene orthology mapping rather than functional orthologous regulatory element identification

## Inputs

- Query peak sets (e.g., mouse mm10 TF ChIP-seq BED/GRanges) from ENCODE erythroid and lymphoblast cells
- Matched orthologous functional genomic datasets (67 matched human-mouse TF/DNA methylation ChIP-seq and DNase-seq) for similarity computation
- Reciprocal-best UCSC chain file, e.g., mm10.hg38.rbest.chain.gz
- Reference genome BSgenome objects (e.g., BSgenome.Mmusculus.UCSC.mm10, BSgenome.Hsapiens.UCSC.hg38) for sequence grammar similarity
- JASPAR PFMs (jaspar_pfm_list) for motif scanning

## Outputs

- Scored and filtered GRangesList of candidate target regions per query region (gr_list_filter)
- Per-TF precision and true-positive counts for AdaLiftOver vs UCSC liftOver, summarized as liftover_benchmark_results.csv
- Logistic regression parameters and suggested score threshold from training_module when using a custom epigenome repertoire

## How to apply

Load the matched query and orthologous peak sets (e.g., 55 human-mouse TF ChIP-seq peak sets from erythroid and lymphoblast cells in ENCODE) as GRanges objects with GenomicRanges. Map query regions to candidate target regions with AdaLiftOver's adaptive_liftover using a reciprocal-best UCSC chain file such as mm10.hg38.rbest.chain, imported via rtracklayer::import.chain; in parallel run UCSC liftOver (via rtracklayer) as the baseline. Compute epigenomic signal similarities and sequence grammar similarities (motif scanning via motifmatchr with jaspar_pfm_list) between query and candidate regions using matched orthologous functional genomic datasets. Aggregate similarities as logistic probability scores and apply gr_candidate_filter, retaining at most best_k candidates per query region and applying a score threshold (e.g., best_k = 1L, threshold = 0.5). Score a lifted peak as a true positive when it overlaps the orthologous peak from the matched ENCODE dataset, then compute precision and true-positive counts per TF.

## Related tools

- **AdaLiftOver** (Core R package: adaptive_liftover extends query regions, compute_similarity_epigenome and compute_similarity_grammar score candidates, gr_candidate_filter and training_module filter and calibrate) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Baseline chain-based cross-species interval mapping framework that AdaLiftOver extends; run via rtracklayer for comparison)
- **rtracklayer** (Imports UCSC chain files (import.chain) and runs liftOver mapping)
- **GenomicRanges** (Represents query peaks and candidate target regions as GRanges/GRangesList objects)
- **motifmatchr** (Fast motif scanning used for sequence grammar similarity computation)

## Examples

```
chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain); gr_list <- compute_similarity_epigenome(gr, gr_list, epigenome_mm10, epigenome_hg38)
```

## Evaluation signals

- On the 55 matched human-mouse TF ChIP-seq benchmark, AdaLiftOver's mean precision should reach ~0.372 (>50% above UCSC liftOver)
- Mean true-positive count should be ~4456 for AdaLiftOver vs ~4313 for UCSC liftOver across the 55 TF datasets
- gr_candidate_filter should retain at most best_k candidate target regions per query region and drop candidates below the score threshold
- Every lifted peak assigned as a true positive must overlap the orthologous peak from the matched ENCODE dataset
- Results table liftover_benchmark_results.csv should contain per-TF precision and true-positive counts for both methods

## Limitations

- Existing sequence-conservation methods (including naive liftOver) suffer from multi-mapping, low precision, and low mapping rate for non-coding regions
- For model organisms other than mouse (e.g., rat), a user-defined orthologous epigenome repertoire must be collected and logistic parameters learned via training_module before filtering
- The tool requires matched orthologous functional genomic datasets and species-specific BSgenome objects; performance depends on their quality and coverage

## Evidence

- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions"
- [intro] We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function.: "We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function."
- [intro] **best_k**: retain at most best k candidate target regions for each query region.: "**best_k**: retain at most best k candidate target regions for each query region."
- [readme] gr_list <- adaptive_liftover(gr, chain): "## map the query regions
gr_list <- adaptive_liftover(gr, chain)

## compute epigenome signal similarity"
- [abstract] existing methods for mapping non-coding genomic regions across species are challenged by the multi-mapping, low precision, and low mapping rate issues: "existing methods for mapping non-coding genomic regions across species are challenged by the multi-mapping, low precision, and low mapping rate issues"
