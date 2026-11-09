---
name: cross-method-benchmark-comparison
description: Use when you have ranked TR lists from multiple TR-identification methods
  applied to the same query region sets (e.g., TR perturbation-derived DARs or cancer-type-specific
  accessible regions) and need to quantify inter-method agreement or rank-based accuracy.
when_to_use_negative:
- Input is raw sequencing data (FASTQ/BAM) rather than already-generated ranked TR
  lists — peak calling and TR identification must happen first
- Only one method's ranked TR list is available, so no pairwise or MRR comparison
  can be made
- The goal is de novo TR identification from a query region set rather than benchmarking
  agreement among existing method outputs
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3168
tools:
- name: BIT
  role: Benchmarked method producing Bayesian posterior-based TR rankings from epigenomic
    query region sets
  repo: https://github.com/ZeyuL01/BIT
- name: BART2
  role: ChIP-seq-based comparison method; run in region mode on the same query region
    sets to produce TR rankings
  repo: https://github.com/zanglab/bart2
- name: ChIP-Atlas
  role: ChIP-seq-based comparison method ranking TRs using Fisher's exact test p values
- name: I-cisTarget
  role: Motif-based comparison method using ENCODE references and recovery analysis
- name: HOMER
  role: Motif-based comparison method leveraging TR binding motifs as reference
- name: WhichTF
  role: Motif-based comparison method using JASPAR/TRANSFAC motifs with hypergeometric
    testing
- name: R
  role: Language used to implement the Jaccard matrix computation, CSV export, and
    heatmap rendering
  repo: https://github.com/ZeyuL01/BIT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:ca8939338c8ec9375776dbc2abacbb9143ae1eff3bbcccdddd3928b51468092d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZeyuL01/BIT
  repo_url: https://github.com/ZeyuL01/BIT
  tools:
  - BART
  - BART2
  - BIT
  - ChIP-Atlas
  - HOMER
  - I-cisTarget
  - R
  - WhichTF
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-method-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the paper's cross-method benchmarking by computing all pairwise Jaccard indices of top-10/50/100 ranked TR lists from six methods (BIT, BART, ChIP-Atlas, i-cisTarget, HOMER, WhichTF) and benchmarking ranking quality with mean reciprocal rank (MRR).

## When to use

Use when you have ranked TR lists from multiple TR-identification methods applied to the same query region sets (e.g., TR perturbation-derived DARs or cancer-type-specific accessible regions) and need to quantify inter-method agreement or rank-based accuracy.

## When NOT to use

- Input is raw sequencing data (FASTQ/BAM) rather than already-generated ranked TR lists — peak calling and TR identification must happen first
- Only one method's ranked TR list is available, so no pairwise or MRR comparison can be made
- The goal is de novo TR identification from a query region set rather than benchmarking agreement among existing method outputs

## Inputs

- Ranked TR tables (one per method) from BIT, BART2, ChIP-Atlas, i-cisTarget, HOMER, and WhichTF for the same query region sets
- TR perturbation-derived differentially accessible region (DAR) sets
- Cancer-type-specific accessible region sets (derived from TCGA)
- Top-k cutoff values (k = 10, 50, 100)
- Ground-truth perturbed TR identities (five perturbed TRs in bulk datasets) for MRR computation

## Outputs

- Pairwise method-by-method Jaccard similarity matrices for each k and region-set type (CSV)
- Heatmaps of Jaccard similarity mirroring Supplementary Fig. 9
- Mean reciprocal rank (MRR) values per method (Fig. 3f)

## How to apply

Load the ranked TR tables produced by each of the six methods (BIT, BART2, ChIP-Atlas, i-cisTarget, HOMER, WhichTF) for the TR perturbation-derived DARs and the cancer-type-specific accessible region sets. For each query region set and each top-k cutoff (k = 10, 50, 100), extract the top-k ranked TR names per method and compute all pairwise Jaccard indices as |intersection| / |union| of the top-k sets. Assemble the values into method-by-method similarity matrices per k and region-set type, write them to CSV, and render heatmaps mirroring Supplementary Fig. 9. Separately, compute mean reciprocal rank (MRR) for each method using the five perturbed TRs in bulk datasets (Fig. 3f) to benchmark ranking quality; expect low off-diagonal Jaccard similarity across methods and highest MRR for BIT on both bulk and single-cell perturbation data.

## Related tools

- **BIT** (Benchmarked method producing Bayesian posterior-based TR rankings from epigenomic query region sets) — https://github.com/ZeyuL01/BIT
- **BART2** (ChIP-seq-based comparison method; run in region mode on the same query region sets to produce TR rankings) — https://github.com/zanglab/bart2
- **ChIP-Atlas** (ChIP-seq-based comparison method ranking TRs using Fisher's exact test p values)
- **I-cisTarget** (Motif-based comparison method using ENCODE references and recovery analysis)
- **HOMER** (Motif-based comparison method leveraging TR binding motifs as reference)
- **WhichTF** (Motif-based comparison method using JASPAR/TRANSFAC motifs with hypergeometric testing)
- **R** (Language used to implement the Jaccard matrix computation, CSV export, and heatmap rendering) — https://github.com/ZeyuL01/BIT

## Examples

```
jaccard <- function(a, b) length(intersect(a, b)) / length(union(a, b))
```

## Evaluation signals

- All Jaccard values lie in [0, 1]; each similarity matrix has a diagonal of exactly 1 and is symmetric
- Off-diagonal Jaccard values are generally low, consistent with the paper's report that Jaccard similarity among these approaches is generally low (Supplementary Fig. 9)
- Matrices are produced for every combination of top-k cutoff (10, 50, 100) and region-set type (TR perturbation-derived DARs and cancer-type-specific accessible regions)
- MRR computed over the five perturbed TRs in bulk data should be highest for BIT, as reported for both bulk and single-cell perturbation data (Fig. 3f)
- The CSV output contains one similarity matrix per k and region-set type with all six methods represented as rows and columns

## Limitations

- Jaccard similarity among the six approaches is generally low, so agreement matrices reflect method-specific reference libraries (ChIP-seq vs motif-based) rather than one method being wrong
- Reproducibility gap: comparisons reference Supplementary Figures 9, 10a, 10b, and 10c, which are not present in the provided text, so exact target values cannot be verified from the article alone
- BIT's performance can be reduced by biological complexities of TR regulation (dynamic binding, chromatin structure, cooperative binding) and sequencing biases such as GC content or PCR amplification bias, which affect the reliability of input region sets used in the comparison

## Evidence

- [readme] BART2 provides TR rankings from the same region sets in its region mode, one of the six compared methods.: [evidence span withheld — anchor status: not_found]
