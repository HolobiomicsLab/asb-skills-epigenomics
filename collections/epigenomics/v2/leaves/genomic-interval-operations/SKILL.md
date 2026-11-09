---
name: genomic-interval-operations
description: Use when genomic intervals (e.g., ATAC-seq or ChIP-seq peak BED regions)
  in one genome assembly (hg38, mm10, rn6) must be lifted over, partitioned by mapping
  multiplicity, or compared against intervals in another species using UCSC chain
  files and GRanges.
when_to_use_negative:
- Input data is not interval-based (e.g., a gene expression matrix or variant VCF
  without per-region epigenomic signal) — the liftOver/GRanges workflow requires genomic
  coordinates with strand and associated epigenomic signals.
- No matched orthologous epigenomic datasets exist for the species pair and no training
  data is available, since compute_similarity_epigenome and training_module() require
  a matched epigenome repertoire.
- The goal is simple coordinate conversion without scoring or filtering — plain UCSC
  liftOver suffices, whereas AdaLiftOver adds epigenomic and sequence grammar prioritization.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: AdaLiftOver
  role: Provides adaptive_liftover(), compute_similarity_epigenome(), compute_similarity_grammar(),
    gr_candidate_filter(), and training_module() for interval mapping and scoring
  repo: https://github.com/keleslab/AdaLiftOver
- name: GenomicRanges
  role: Operating and storing genomic intervals for query peaks and candidate target
    regions
- name: rtracklayer
  role: Imports UCSC chain files with import.chain() for the liftOver framework
- name: data.table
  role: Fast data manipulation, e.g., aggregating training results with rbindlist
- name: motifmatchr
  role: Fast motif scanning underlying sequence grammar similarity computation
  repo: https://github.com/GreenleafLab/motifmatchr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:44b0cc3cbfa8fdbaf0b489a37b25caacbeacc78ff21f94be698daac647a22d38
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# genomic-interval-operations

## Summary

Operates on genomic intervals (ATAC-seq/ChIP-seq peaks) across genome assemblies using GenomicRanges and UCSC chain files, as implemented in AdaLiftOver to map query peaks to scored candidate orthologous target regions.

## When to use

Use when genomic intervals (e.g., ATAC-seq or ChIP-seq peak BED regions) in one genome assembly (hg38, mm10, rn6) must be lifted over, partitioned by mapping multiplicity, or compared against intervals in another species using UCSC chain files and GRanges.

## When NOT to use

- Input data is not interval-based (e.g., a gene expression matrix or variant VCF without per-region epigenomic signal) — the liftOver/GRanges workflow requires genomic coordinates with strand and associated epigenomic signals.
- No matched orthologous epigenomic datasets exist for the species pair and no training data is available, since compute_similarity_epigenome and training_module() require a matched epigenome repertoire.
- The goal is simple coordinate conversion without scoring or filtering — plain UCSC liftOver suffices, whereas AdaLiftOver adds epigenomic and sequence grammar prioritization.

## Inputs

- Query peak regions as GenomicRanges (BED-style genomic intervals, e.g., human brain ATAC-seq peaks in hg38)
- UCSC reciprocal-best chain file (e.g., mm10.hg38.rbest.chain, rn6.hg38.rbest.chain)
- Orthologous epigenome repertoire as GRangesList (e.g., epigenome_mm10 / epigenome_hg38, ENCODE datasets)
- BSgenome objects for both assemblies (e.g., BSgenome.Hsapiens.UCSC.hg38, BSgenome.Mmusculus.UCSC.mm10)
- jaspar_pfm_list for motif-based sequence grammar similarity

## Outputs

- GRangesList of scored candidate target regions per query region (adaptive_liftover output)
- Filtered candidate target regions with logistic probability scores (gr_candidate_filter output)
- Partitioned regions with scores and Group I low-score fraction written to adaliftover_group_scores.csv
- Logistic regression parameters and suggested score threshold from training_module()

## How to apply

Load query peaks into a GenomicRanges object and import the appropriate UCSC reciprocal-best chain file (e.g., mm10.hg38.rbest.chain, rn6.hg38.rbest.chain) with rtracklayer::import.chain. Run AdaLiftOver's adaptive_liftover(gr, chain), which extends queries via the liftOver framework and returns a list of scored candidate target regions per query. Compute epigenomic signal similarities (compute_similarity_epigenome) using matched orthologous functional genomic datasets and sequence grammar similarities (compute_similarity_grammar with jaspar_pfm_list), then filter candidates with gr_candidate_filter(best_k = 1L, threshold = 0.5). For organisms other than mouse, estimate logistic regression parameters with training_module() on matched epigenome datasets (max_filter_proportion = 0.5). In the zebrafish case, partition mapped regions by mapping multiplicity into Group I (unique sequence-based mapping) and Group II (multi-mappers), and quantify the fraction of Group I regions with AdaLiftOver mapping scores below 0.5.

## Related tools

- **AdaLiftOver** (Provides adaptive_liftover(), compute_similarity_epigenome(), compute_similarity_grammar(), gr_candidate_filter(), and training_module() for interval mapping and scoring) — https://github.com/keleslab/AdaLiftOver
- **GenomicRanges** (Operating and storing genomic intervals for query peaks and candidate target regions)
- **rtracklayer** (Imports UCSC chain files with import.chain() for the liftOver framework)
- **data.table** (Fast data manipulation, e.g., aggregating training results with rbindlist)
- **motifmatchr** (Fast motif scanning underlying sequence grammar similarity computation) — https://github.com/GreenleafLab/motifmatchr

## Examples

```
chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain)
```

## Evaluation signals

- adaptive_liftover(gr, chain) returns a GRangesList structure with one entry of candidate target regions per query region after chain import succeeds.
- After gr_candidate_filter(best_k = 1L, threshold = 0.5), each query retains at most one candidate target region with a logistic probability score above the 0.5 threshold.
- In the zebrafish brain ATAC-seq case, the expected finding is that 16.4% of Group I (uniquely sequence-mapped) query regions have AdaLiftOver mapping scores < 0.5; the fraction should fall in a plausible range and be recorded in adaliftover_group_scores.csv.
- Partitioning by mapping multiplicity must yield Group I (unique sequence-based mapping) and Group II (multi-mappers) subsets that together account for all mapped query regions.
- For new species pairs, training_module() on matched epigenomes should label candidate target regions overlapping human orthologous peaks as positives and produce averaged logistic regression parameters (e.g., max_filter_proportion = 0.5, interaction = FALSE).

## Limitations

- Existing sequence-conservation methods for mapping non-coding regions across species suffer from multi-mapping, low precision, and low mapping rate issues; multi-mappers (Group II) require epigenomic prioritization and cannot be resolved by sequence alone.
- For model organisms other than mouse (e.g., rat, zebrafish), the orthologous epigenome repertoire and BSgenome objects must be collected and installed from scratch, and logistic regression parameters may need retraining via training_module().
- The default ENCODE epigenome repertoire (67 matched human-mouse TF/DNA methylation ChIP-seq and DNase-seq datasets) applies only to human-mouse mapping; other species pairs require user-defined GRangesList repertoires.

## Evidence

- [abstract] AdaLiftOver builds on the UCSC liftOver framework: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions"
- [intro] GenomicRanges for intervals: "`r Biocpkg("GenomicRanges")` is used for operating genomic intervals."
- [readme] Filter parameters: "gr_list_filter <- gr_candidate_filter(
    gr_list,
    best_k = 1L,
    threshold = 0.5
)"
- [readme] Chain import invocation: "chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain")"
- [intro] Training module purpose: "AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold"
