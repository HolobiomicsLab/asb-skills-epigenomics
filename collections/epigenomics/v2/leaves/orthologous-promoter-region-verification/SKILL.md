---
name: orthologous-promoter-region-verification
description: Use when a GWAS SNP or query regulatory element has been mapped across
  species (e.g., human hg38 to mouse mm10 via mm10.hg38.r.chain) and you need positive-control
  verification that a promoter-region SNP maps to the orthologous promoter.
when_to_use_negative:
- Query regions are coding sequences or other conserved elements better handled by
  sequence-alignment-based orthology tools, since AdaLiftOver targets orthologous
  non-coding regulatory regions
- No matched human-target-species epigenomic datasets exist and no training-module
  parameters can be estimated, so epigenomic similarity scores cannot be computed
- You need a many-to-many whole-genome synteny map rather than a filtered, scored
  list of candidate target regions per query region
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: AdaLiftOver
  role: 'Core R package performing adaptive liftOver: extends query regions via the
    UCSC liftOver framework and prioritizes candidate target regions by epigenomic
    and sequence grammar similarity; installed via devtools::install_github("ThomasDCY/AdaLiftOver")'
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Underlying framework used by adaptive_liftover to extend query regions into
    candidate target regions using the reciprocal-best chain file
  repo: https://github.com/keleslab/AdaLiftOver
- name: rtracklayer
  role: Imports the UCSC chain file (import.chain) for use in adaptive_liftover
- name: GenomicRanges
  role: Represents query SNPs and candidate target regions as genomic intervals throughout
    the workflow
- name: motifmatchr
  role: Fast motif scanning used to compute sequence grammar similarities between
    query and candidate target regions
  repo: https://github.com/GreenleafLab/motifmatchr
- name: data.table
  role: Fast data manipulation for loading SNP lists and aggregating training results
    (e.g., rbindlist over tissues)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:82ad00fd6ec6c013cef7d717ccfe1bab909ca7e82f13fdb4f6cfeb92ef49be76
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
yes_rate: 0.75
n_claims: 12
---

# orthologous-promoter-region-verification

## Summary

Validates cross-species regulatory element mappings (e.g., AdaLiftOver SNP-to-target maps) by checking that a known human promoter SNP lands in the orthologous mouse promoter region, using reciprocal-best UCSC chain liftOver plus epigenomic and sequence-grammar scoring as the mapping engine.

## When to use

Use when a GWAS SNP or query regulatory element has been mapped across species (e.g., human hg38 to mouse mm10 via mm10.hg38.rbest.chain) and you need positive-control verification that a promoter-region SNP maps to the orthologous promoter.

## When NOT to use

- Query regions are coding sequences or other conserved elements better handled by sequence-alignment-based orthology tools, since AdaLiftOver targets orthologous non-coding regulatory regions
- No matched human-target-species epigenomic datasets exist and no training-module parameters can be estimated, so epigenomic similarity scores cannot be computed
- You need a many-to-many whole-genome synteny map rather than a filtered, scored list of candidate target regions per query region

## Inputs

- Fine-mapped GWAS SNP list as genomic intervals (GRanges), e.g., 116,402 BMD SNPs at PIP >= 0.001
- UCSC reciprocal-best chain file (mm10.hg38.rbest.chain.gz), unzipped and imported via rtracklayer::import.chain
- Matched human-mouse epigenome repertoire (e.g., ENCODE epigenome_hg38 and epigenome_mm10 GRangesList objects)
- JASPAR PFMS list (jaspar_pfm_list) for motif scanning
- Training module output: estimated logistic regression parameters and optimal score threshold

## Outputs

- Scored and filtered candidate orthologous mouse target regions per query SNP (e.g., bmd_snp_adaliftover_results.csv)
- Verification result: rs9486902 (human FOXO3 promoter) mapping to the mouse Foxo3 promoter region
- Precision benchmark of AdaLiftOver vs UCSC liftOver (65.2% precision increase on 90 BMD genes)

## How to apply

Load the fine-mapped SNP list (e.g., 116,402 UK Biobank BMD GWAS SNPs at PIP ≥ 0.001, or 3,125 SNPs at PIP ≥ 0.1) as genomic intervals with GenomicRanges and data.table. Run AdaLiftOver's adaptive_liftover with the UCSC reciprocal-best chain file mm10.hg38.rbest.chain.gz to extend each query SNP to candidate mouse target regions, then score candidates with the training-module logistic regression on epigenomic signal similarity (67 matched ENCODE human-mouse TF/DNA methylation ChIP-seq and DNase-seq datasets) and sequence grammar similarity (jaspar_pfm_list via motifmatchr). Filter with gr_candidate_filter using best_k and threshold (README example: best_k = 1L, threshold = 0.5). Verify the mapping by confirming that the positive-control SNP rs9486902 at the human FOXO3 promoter maps to the mouse Foxo3 promoter region; benchmark precision against UCSC liftOver (reported 65.2% precision increase over 90 BMD genes).

## Related tools

- **AdaLiftOver** (Core R package performing adaptive liftOver: extends query regions via the UCSC liftOver framework and prioritizes candidate target regions by epigenomic and sequence grammar similarity; installed via devtools::install_github("ThomasDCY/AdaLiftOver")) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Underlying framework used by adaptive_liftover to extend query regions into candidate target regions using the reciprocal-best chain file) — https://github.com/keleslab/AdaLiftOver
- **rtracklayer** (Imports the UCSC chain file (import.chain) for use in adaptive_liftover)
- **GenomicRanges** (Represents query SNPs and candidate target regions as genomic intervals throughout the workflow)
- **motifmatchr** (Fast motif scanning used to compute sequence grammar similarities between query and candidate target regions) — https://github.com/GreenleafLab/motifmatchr
- **data.table** (Fast data manipulation for loading SNP lists and aggregating training results (e.g., rbindlist over tissues))

## Examples

```
chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- Positive-control check: SNP rs9486902 at the human FOXO3 promoter maps to the mouse Foxo3 promoter region in the filtered output
- Precision benchmark: AdaLiftOver precision on BMD gene mappings exceeds UCSC liftOver by the reported 65.2% (90 BMD genes identified from the 116,402-SNP PIP >= 0.001 set)
- gr_candidate_filter enforces at most best_k candidate target regions per query region and a score threshold (README example: best_k = 1L, threshold = 0.5)
- All output candidate regions carry logistic probability scores aggregating epigenomic signal and sequence grammar similarities
- Training module suggests an optimal score threshold without filtering out too many candidate target regions (max_filter_proportion, e.g., 0.5)

## Limitations

- Existing sequence-conservation methods for mapping non-coding regions suffer from multi-mapping, low precision, and low mapping rate issues; candidate target regions can still be ambiguous without epigenomic scoring
- For model organisms other than mouse (e.g., rat), a user-defined epigenome repertoire and new training-module parameter estimation are required (LOOCV workflow with orthologous datasets, e.g., ChIP-Atlas H3K4me3 for rat)
- Mapping depends on availability of a reciprocal-best chain file between the query and target genomes

## Evidence

- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence"
- [intro] We uses 67 matched ENCODE functional genomic datasets including TF/DNA methylation ChIP-seq and DNase-seq between human and mouse: "We uses 67 matched ENCODE functional genomic datasets including TF/DNA methylation ChIP-seq and DNase-seq between human and mouse"
- [intro] **best_k**: retain at most best k candidate target regions for each query region.: "**best_k**: retain at most best k candidate target regions for each query region."
- [readme] gr_list_filter <- gr_candidate_filter(
    gr_list,
    best_k = 1L,
    threshold = 0.5
): "gr_list_filter <- gr_candidate_filter(
    gr_list,
    best_k = 1L,
    threshold = 0.5
)"
- [intro] AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions.: "AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions."
