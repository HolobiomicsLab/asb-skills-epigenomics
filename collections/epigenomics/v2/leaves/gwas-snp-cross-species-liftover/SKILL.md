---
name: gwas-snp-cross-species-liftover
description: Use when fine-mapped human GWAS SNP coordinates (e.g., PIP ≥ 0.1 or PIP
  ≥ 0.001 fine-mapping outputs) need to be mapped to candidate orthologous regulatory
  regions in a model organism genome such as mm10.
when_to_use_negative:
- The query regions are coding sequences or whole genes rather than non-coding regulatory
  intervals — AdaLiftOver is designed for identifying functionally similar orthologous
  non-coding regions.
- No matched orthologous epigenomic datasets are available for the species pair and
  no training-module data can be collected, so epigenomic signal similarity cannot
  be computed.
- You only need raw coordinate conversion of SNPs/regions without prioritization by
  regulatory similarity — plain UCSC liftOver (rtracklayer::liftOver) suffices and
  avoids the logistic-scoring overhead.
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: AdaLiftOver
  role: 'Core R package: adaptive_liftover extends query SNPs via the UCSC liftOver
    framework and computes/prioritizes candidate target regions by epigenomic and
    sequence grammar similarity; includes gr_candidate_filter and training_module'
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Baseline coordinate-conversion framework that AdaLiftOver builds on; also
    used as the comparison benchmark for mapping counts and precision
- name: rtracklayer
  role: Imports the UCSC reciprocal-best chain file (import.chain) and runs baseline
    liftOver conversion of SNP coordinates
- name: GenomicRanges
  role: Represents GWAS SNPs, epigenome peak sets, and candidate target regions as
    GRanges/GRangesList objects and performs interval intersections for enrichment
    analysis
- name: data.table
  role: Fast loading and manipulation of SNP lists and result tables
- name: motifmatchr
  role: Fast motif scanning used to compute sequence grammar similarities between
    query and candidate target regions
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:57832bb80afab76cb90a796697322355e5da4dd708fc0503ac2c20b9c5a04f70
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
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# gwas-snp-cross-species-liftover

## Summary

Maps fine-mapped human GWAS SNPs to orthologous non-coding target regions in another species (e.g., hg38 to mm10) using AdaLiftOver, which extends query SNPs via the UCSC liftOver framework and prioritizes candidate target regions by epigenomic signal and sequence grammar similarity.

## When to use

Use when fine-mapped human GWAS SNP coordinates (e.g., PIP ≥ 0.1 or PIP ≥ 0.001 fine-mapping outputs) need to be mapped to candidate orthologous regulatory regions in a model organism genome such as mm10.

## When NOT to use

- The query regions are coding sequences or whole genes rather than non-coding regulatory intervals — AdaLiftOver is designed for identifying functionally similar orthologous non-coding regions.
- No matched orthologous epigenomic datasets are available for the species pair and no training-module data can be collected, so epigenomic signal similarity cannot be computed.
- You only need raw coordinate conversion of SNPs/regions without prioritization by regulatory similarity — plain UCSC liftOver (rtracklayer::liftOver) suffices and avoids the logistic-scoring overhead.

## Inputs

- Fine-mapped GWAS SNP coordinates in hg38 (e.g., 1648 SCZ SNPs; 3125 BMD SNPs at PIP >= 0.1 and 116,402 BMD SNPs at PIP >= 0.001)
- UCSC reciprocal-best chain file (e.g., mm10.hg38.rbest.chain.gz)
- Matched human-mouse epigenomic datasets (67 matched ENCODE TF/DNA methylation ChIP-seq and DNase-seq; epigenome_mm10 and epigenome_hg38 GRangesList)
- JASPAR PWM list (jaspar_pfm_list) for sequence grammar similarity
- BSgenome objects for both species (e.g., BSgenome.Hsapiens.UCSC.hg38, BSgenome.Mmusculus.UCSC.mm10)

## Outputs

- Scored and filtered candidate orthologous target regions per query SNP (GRanges/GRangesList)
- Mapping counts, mapping rates, and precision comparison table versus UCSC liftOver (e.g., snp_liftover_mapping_stats.csv)
- Per-SNP orthologous mouse target region results (e.g., bmd_snp_adaliftover_results.csv)
- Cell-population enrichment statistics from intersecting mapped SNPs with chromatin accessible regions (e.g., mouse excitatory/inhibitory neuron ATAC regions)

## How to apply

Load the fine-mapped SNP list (e.g., 1648 SCZ SNPs or 116,402 BMD SNPs at PIP ≥ 0.001) as genomic intervals with GenomicRanges and data.table, and import the reciprocal-best chain file (mm10.hg38.rbest.chain.gz) with rtracklayer::import.chain. Run adaptive_liftover to extend each query SNP to candidate target regions, then score candidates with compute_similarity_epigenome (67 matched ENCODE human-mouse TF/DNA methylation ChIP-seq and DNase-seq datasets) and compute_similarity_grammar using jaspar_pfm_list motif scanning via motifmatchr. Estimate logistic regression parameters with the training_module on a pair of matched epigenome datasets, then filter candidates with gr_candidate_filter using best_k (e.g., best_k = 1L) and a score threshold (e.g., threshold = 0.5). Benchmark mapping counts and precision against UCSC liftOver (run via rtracklayer) and validate with a positive control such as rs9486902 at the human FOXO3 promoter mapping to the mouse Foxo3 promoter. Write the filtered, scored orthologous target regions to a results CSV.

## Related tools

- **AdaLiftOver** (Core R package: adaptive_liftover extends query SNPs via the UCSC liftOver framework and computes/prioritizes candidate target regions by epigenomic and sequence grammar similarity; includes gr_candidate_filter and training_module) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Baseline coordinate-conversion framework that AdaLiftOver builds on; also used as the comparison benchmark for mapping counts and precision)
- **rtracklayer** (Imports the UCSC reciprocal-best chain file (import.chain) and runs baseline liftOver conversion of SNP coordinates)
- **GenomicRanges** (Represents GWAS SNPs, epigenome peak sets, and candidate target regions as GRanges/GRangesList objects and performs interval intersections for enrichment analysis)
- **data.table** (Fast loading and manipulation of SNP lists and result tables)
- **motifmatchr** (Fast motif scanning used to compute sequence grammar similarities between query and candidate target regions)

## Examples

```
library(AdaLiftOver); chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain); gr_list <- compute_similarity_epigenome(gr, gr_list, epigenome_mm10, epigenome_hg38); gr_list <- compute_similarity_grammar(gr, gr_list, "mm10", "hg38", jaspar_pfm_list); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- Mapping counts and rates match reported baselines: for SCZ GWAS SNPs, AdaLiftOver maps 612 (37.5%) of 1648 SNPs while UCSC liftOver maps 715 (43.3%).
- Precision improvement is reproduced: AdaLiftOver achieves a higher average precision of 8.07% on SCZ SNPs and a 65.2% increase in precision over UCSC liftOver for the 116,402 BMD SNP set.
- Positive-control SNP rs9486902 at the human FOXO3 promoter maps to the mouse Foxo3 promoter region.
- BMD gene association count is reproduced: the expanded UK Biobank BMD GWAS SNP set (PIP ≥ 0.001) maps to 90 BMD genes.
- gr_candidate_filter output respects both parameters: at most best_k candidate target regions per query region, and no candidate below the score threshold is retained.

## Limitations

- Existing sequence-conservation mapping methods (the baseline setting) suffer from multi-mapping, low precision, and low mapping rate issues; results depend on the quality of the matched epigenomic repertoire.
- For organisms other than mouse (e.g., rat), the user must collect an orthologous epigenome repertoire from scratch and estimate averaged logistic regression parameters via training_module (e.g., with max_filter_proportion = 0.5, interaction = FALSE).
- Mapping rate and precision can trade off against each other: in the SCZ case, UCSC liftOver mapped more SNPs (715 vs 612) but with lower precision, so users must decide which metric matters for their downstream use.

## Evidence

- [abstract] We develop Adaptive liftOver (AdaLiftOver), a large-scale computational tool for identifying functionally similar orthologous non-coding regions across species.: "We develop Adaptive liftOver (AdaLiftOver), a large-scale computational tool for identifying functionally similar orthologous non-coding regions across species."
- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence"
- [intro] best_k: retain at most best k candidate target regions for each query region. threshold: the score threshold for all the candidate target regions.: "**best_k**: retain at most best k candidate target regions for each query region.
* **threshold**: the score threshold for all the candidate target regions."
- [intro] We uses 67 matched ENCODE functional genomic datasets including TF/DNA methylation ChIP-seq and DNase-seq between human and mouse: "We uses 67 matched ENCODE functional genomic datasets including TF/DNA methylation ChIP-seq and DNase-seq between human and mouse"
- [intro] AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions.: "AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions."
- [readme] gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5): "gr_list_filter <- gr_candidate_filter(
    gr_list,
    best_k = 1L,
    threshold = 0.5
)"
