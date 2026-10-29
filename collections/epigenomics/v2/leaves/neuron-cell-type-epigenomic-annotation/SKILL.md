---
name: neuron-cell-type-epigenomic-annotation
description: Use when fine-mapped human GWAS SNPs in hg38 must be mapped to the mouse
  genome and annotated against cell-type-specific chromatin accessible regions (e.g.,
  mouse excitatory vs inhibitory neurons) to test cell-type enrichment.
when_to_use_negative:
- Input regions are not human-to-mouse (hg38-to-mm10) ortholog queries and no reciprocal-best
  chain file or matched orthologous epigenome repertoire is available
- The analysis only needs raw cross-species coordinate conversion without functional
  prioritization, and no cell-type chromatin annotation is required — plain UCSC liftOver
  via rtracklayer suffices
- Mapped SNPs cannot be annotated because no mouse neuron chromatin accessibility
  datasets are provided for excitatory or inhibitory populations
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3304
tools:
- name: AdaLiftOver
  role: Adaptively maps hg38 SNPs to mm10 candidate target regions, scoring candidates
    by epigenomic signal and sequence grammar similarity and filtering with gr_candidate_filter(best_k,
    threshold)
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Baseline coordinate conversion of SNPs to the mouse genome via rtracklayer,
    producing the comparison mapping counts and rates
- name: rtracklayer
  role: Imports the mm10.hg38.rbest.chain chain file and runs the UCSC liftOver framework
    on the SNP set
- name: GenomicRanges
  role: Intersects mapped SNPs with chromatin accessible regions of mouse excitatory
    and inhibitory neuron populations to compute enrichment
- name: data.table
  role: Fast data manipulation and computation for tabulating mapping counts, rates,
    and enrichment statistics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:032b3d5981687fc831262514f7db48d18f6d221d0aa7626ebe1feabed713d231
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
  - rtracklayer
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3304
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# neuron-cell-type-epigenomic-annotation

## Summary

A workflow for lifting fine-mapped human GWAS SNPs (e.g., 1648 hg38 SCZ SNPs) to the mouse genome with AdaLiftOver versus UCSC liftOver, then annotating the mapped SNPs against chromatin accessibility of mouse excitatory and inhibitory neuron populations to compute cell-type enrichment statistics.

## When to use

Use when fine-mapped human GWAS SNPs in hg38 must be mapped to the mouse genome and annotated against cell-type-specific chromatin accessible regions (e.g., mouse excitatory vs inhibitory neurons) to test cell-type enrichment.

## When NOT to use

- Input regions are not human-to-mouse (hg38-to-mm10) ortholog queries and no reciprocal-best chain file or matched orthologous epigenome repertoire is available
- The analysis only needs raw cross-species coordinate conversion without functional prioritization, and no cell-type chromatin annotation is required — plain UCSC liftOver via rtracklayer suffices
- Mapped SNPs cannot be annotated because no mouse neuron chromatin accessibility datasets are provided for excitatory or inhibitory populations

## Inputs

- 1648 fine-mapped Schizophrenia GWAS SNP coordinates (hg38 BED/GRanges)
- mm10.hg38.rbest.chain.gz reciprocal-best UCSC chain file
- ENCODE matched human-mouse epigenomic datasets (TF/DNA methylation ChIP-seq and DNase-seq)
- jaspar_pfm_list (JASPAR position frequency matrices for sequence grammar similarity)
- Mouse excitatory and inhibitory neuron chromatin accessible region peaks (GRanges)

## Outputs

- Mapped and unmapped SNP lists for both UCSC liftOver and AdaLiftOver
- Mapping counts, mapping rates, and average precision per method
- Per-cell-type (excitatory vs inhibitory neuron) overlap and enrichment statistics
- Comparison table written to snp_liftover_mapping_stats.csv

## How to apply

Load the 1648 fine-mapped SCZ GWAS SNP coordinates (hg38) and the mm10.hg38 reciprocal-best chain file, importing the chain with rtracklayer::import.chain(). Run UCSC liftOver via rtracklayer as the baseline, then run AdaLiftOver's adaptive_liftover() to extend query SNPs to candidate target regions, score them with compute_similarity_epigenome() and compute_similarity_grammar(), and filter with gr_candidate_filter(best_k, threshold). Tabulate mapping counts, rates, and precision for both methods, checking against the reported outcomes (AdaLiftOver: 612 mapped, 37.5%; UCSC liftOver: 715 mapped, 43.3%; AdaLiftOver average precision 8.07%). Intersect the mapped SNPs with chromatin accessible regions of mouse excitatory and inhibitory neuron populations using GenomicRanges overlap operations, compute enrichment statistics per cell type, and write the comparison table to snp_liftover_mapping_stats.csv.

## Related tools

- **AdaLiftOver** (Adaptively maps hg38 SNPs to mm10 candidate target regions, scoring candidates by epigenomic signal and sequence grammar similarity and filtering with gr_candidate_filter(best_k, threshold)) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Baseline coordinate conversion of SNPs to the mouse genome via rtracklayer, producing the comparison mapping counts and rates)
- **rtracklayer** (Imports the mm10.hg38.rbest.chain chain file and runs the UCSC liftOver framework on the SNP set)
- **GenomicRanges** (Intersects mapped SNPs with chromatin accessible regions of mouse excitatory and inhibitory neuron populations to compute enrichment)
- **data.table** (Fast data manipulation and computation for tabulating mapping counts, rates, and enrichment statistics)

## Examples

```
chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- Mapping counts and rates match the reported outcomes: AdaLiftOver maps 612 (37.5%) of 1648 SNPs; UCSC liftOver maps 715 (43.3%)
- AdaLiftOver achieves an average precision of 8.07% on the mapped SCZ SNPs
- Every mapped SNP overlaps at least one candidate target region retained after gr_candidate_filter with the chosen best_k and score threshold
- Overlap counts against excitatory and inhibitory neuron accessible regions are finite, non-negative integers summing consistently with the total mapped SNP set
- The output file snp_liftover_mapping_stats.csv contains rows for both methods with count, rate, and precision columns populated

## Limitations

- UCSC liftOver maps more SNPs (715 vs 612) than AdaLiftOver on the SCZ case study, so raw mapping rate alone is not evidence of superiority; precision scoring is required for interpretation
- AdaLiftOver requires matched orthologous functional genomic datasets between species; for organisms other than mouse, the training_module parameters must be estimated from a new epigenome pair (e.g., max_filter_proportion = 0.5) and averaged logistic regression parameters used as defaults
- Filtering with best_k and the score threshold can remove many candidate target regions if the threshold is set too aggressively

## Evidence

- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions"
- [abstract] Evaluations spanning GWAS SNPs: "Evaluations of AdaLiftOver with multiple case studies, spanning both genomic intervals from epigenome datasets across a wide range of model organisms and GWAS SNPs"
- [intro] GenomicRanges for interval operations: "`r Biocpkg("GenomicRanges")` is used for operating genomic intervals."
- [readme] Epigenomic similarity computation call: "gr_list <- compute_similarity_epigenome(gr, gr_list, epigenome_mm10, epigenome_hg38)"
