---
name: liftover-mapping-rate-precision-comparison
description: Use when fine-mapped SNP coordinates (e.g., hg38 GWAS SNPs) must be lifted
  to another genome (e.g., mouse mm10) and a head-to-head mapping-rate vs. precision
  comparison of UCSC liftOver and AdaLiftOver is needed.
when_to_use_negative:
- Query regions are protein-coding or otherwise well-conserved intervals where standard
  UCSC liftOver is sufficient and epigenomic signal data for the target organism is
  unavailable.
- No orthologous functional genomic datasets exist for the target species, since AdaLiftOver's
  scoring depends on epigenomic signal and sequence grammar similarities computed
  from matched datasets.
- The analysis needs a single-method coordinate conversion rather than a benchmark
  comparison of mapping rate versus precision.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: AdaLiftOver
  role: 'Adaptive liftover: extends query SNPs via the chain file to candidate target
    regions and scores/filters them by epigenomic and sequence grammar similarity
    (adaptive_liftover, compute_similarity_epigenome, compute_similarity_grammar,
    gr_candidate_filter).'
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Baseline coordinate conversion of SNPs, run via rtracklayer, providing the
    comparison arm for mapping counts and rates.
  repo: https://github.com/keleslab/AdaLiftOver
- name: rtracklayer
  role: Imports the UCSC chain file (import.chain) and executes the baseline liftOver
    conversion of SNP coordinates.
- name: GenomicRanges
  role: Intersects mapped SNPs with mouse neuron chromatin accessible regions to compute
    precision statistics.
- name: data.table
  role: Fast data manipulation for tabulating mapping counts, rates, and precision
    into the comparison CSV.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:a69bbfade5e94ef7a6ad5a2bc62c5dc8b3678290e236d693dff3044de4ae8925
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# liftover-mapping-rate-precision-comparison

## Summary

Benchmarks UCSC liftOver against AdaLiftOver for mapping fine-mapped GWAS SNPs across species (e.g., hg38 SCZ SNPs to mouse mm10), tabulating mapped-SNP counts, mapping rates, and average precision to quantify the trade-off between mapping more SNPs and mapping functionally similar loci.

## When to use

Use when fine-mapped SNP coordinates (e.g., hg38 GWAS SNPs) must be lifted to another genome (e.g., mouse mm10) and a head-to-head mapping-rate vs. precision comparison of UCSC liftOver and AdaLiftOver is needed.

## When NOT to use

- Query regions are protein-coding or otherwise well-conserved intervals where standard UCSC liftOver is sufficient and epigenomic signal data for the target organism is unavailable.
- No orthologous functional genomic datasets exist for the target species, since AdaLiftOver's scoring depends on epigenomic signal and sequence grammar similarities computed from matched datasets.
- The analysis needs a single-method coordinate conversion rather than a benchmark comparison of mapping rate versus precision.

## Inputs

- Fine-mapped GWAS SNP coordinates (hg38), e.g., 1648 Schizophrenia GWAS SNPs
- UCSC reciprocal-best chain file (mm10.hg38.rbest.chain.gz, unzipped) imported via rtracklayer::import.chain
- Orthologous epigenomic repertoire (e.g., 67 matched ENCODE TF/DNA methylation ChIP-seq and DNase-seq datasets for human and mouse)
- Target-organism chromatin accessibility regions (e.g., mouse excitatory and inhibitory neuron populations) for precision computation

## Outputs

- Per-SNP mapping status (mapped/unmapped) for UCSC liftOver and AdaLiftOver
- Mapping counts and rates (e.g., AdaLiftOver 612/37.5% vs. UCSC liftOver 715/43.3%)
- Average precision per method (e.g., 8.07% for AdaLiftOver) from overlap with target-cell chromatin accessible regions
- Comparison table written to snp_liftover_mapping_stats.csv

## How to apply

Load the fine-mapped SNP set (e.g., 1648 hg38 SCZ GWAS SNPs) and the human-mouse reciprocal-best chain file (mm10.hg38.rbest.chain.gz, imported via rtracklayer::import.chain). Convert SNP coordinates with UCSC liftOver through rtracklayer, recording mapped and unmapped SNPs. Run AdaLiftOver's adaptive_liftover on the same SNPs: extend query regions via the chain file to candidate target regions, score candidates by epigenomic signal and sequence grammar similarity aggregated as logistic probability scores, then apply gr_candidate_filter with best_k and a score threshold (e.g., best_k = 1L, threshold = 0.5) so at most best_k candidates are retained per query region. Intersect mapped SNPs with target-organism chromatin accessible regions (e.g., mouse excitatory and inhibitory neuron populations) using GenomicRanges to compute precision. Tabulate mapping counts, mapping rates, and precision for both methods and verify against the reported outcomes: AdaLiftOver maps 612 (37.5%) of 1648 SNPs with 8.07% average precision, while UCSC liftOver maps 715 (43.3%); write the comparison to snp_liftover_mapping_stats.csv.

## Related tools

- **AdaLiftOver** (Adaptive liftover: extends query SNPs via the chain file to candidate target regions and scores/filters them by epigenomic and sequence grammar similarity (adaptive_liftover, compute_similarity_epigenome, compute_similarity_grammar, gr_candidate_filter).) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Baseline coordinate conversion of SNPs, run via rtracklayer, providing the comparison arm for mapping counts and rates.) — https://github.com/keleslab/AdaLiftOver
- **rtracklayer** (Imports the UCSC chain file (import.chain) and executes the baseline liftOver conversion of SNP coordinates.)
- **GenomicRanges** (Intersects mapped SNPs with mouse neuron chromatin accessible regions to compute precision statistics.)
- **data.table** (Fast data manipulation for tabulating mapping counts, rates, and precision into the comparison CSV.)

## Examples

```
chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- AdaLiftOver mapped count and rate reproduce the reported values: 612 of 1648 SNPs (37.5%).
- UCSC liftOver mapped count and rate reproduce the reported values: 715 of 1648 SNPs (43.3%).
- AdaLiftOver average precision from overlap with mouse neuron chromatin accessible regions reaches approximately 8.07%, exceeding UCSC liftOver despite its lower mapping rate.
- gr_candidate_filter output contains at most best_k (e.g., 1) candidate target regions per query region, all with scores above the chosen threshold.
- Final comparison table is written to snp_liftover_mapping_stats.csv with matching counts, rates, and precision columns for both methods.

## Limitations

- UCSC liftOver maps more SNPs (715 vs. 612), so AdaLiftOver trades mapping rate for higher precision — the correct interpretation is a precision/rate trade-off, not uniform superiority.
- AdaLiftOver's scoring requires matched orthologous epigenomic datasets between the two species; for organisms other than mouse (e.g., rat), parameters must be re-estimated with the training_module and orthologous epigenome repertoires collected from scratch.
- The filter step discards candidate target regions below the score threshold, which can filter out many candidates if the threshold is set too aggressively; the training module suggests a threshold 'without filtering out too many candidate target regions'.

## Evidence

- [results] AdaLiftOver maps 612 (37.5%) of the 1648 SNPs to the mouse genome and achieves a higher average precision of 8.07%, in comparison to UCSC liftOver which maps 715 (43.3%) SNPs.: [evidence span withheld — anchor status: not_found]
- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence"
- [intro] We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function.: "We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function."
- [intro] **best_k**: retain at most best k candidate target regions for each query region.: "**best_k**: retain at most best k candidate target regions for each query region."
- [readme] AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions.: "AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions."
