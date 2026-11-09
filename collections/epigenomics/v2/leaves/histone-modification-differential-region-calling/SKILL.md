---
name: histone-modification-differential-region-calling
description: Use when comparing ChIP-seq profiles of broad diffuse histone marks (e.g.
  H3K27me3, H3K9me3, H3K36me3, H3K79me2) between two samples/cell lines and peak-style
  callers would yield false positives/negatives.
when_to_use_negative:
- The histone mark has sharp, peak-like footprints rather than broad diffuse domains
  — methods that search for peak-like features are the intended use case, and applying
  peak-style reasoning here generates many false positive or false negative calls
- Only a single sample needs modification calling without a two-sample comparison,
  and the univariate mode (run.univariate.hmm) alone would suffice
- The input is already a binned read-count feature table with no underlying BAM alignments
  for the comparison of interest
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: Bivariate HMM that bins the genome, fits emission/transition parameters, and
    calls differentially modified regions between two samples
  repo: https://github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment in which histoneHMM runs and from which the binning/HMM
    functions are invoked
- name: samtools
  role: Removes duplicated reads from ChIP-seq BAM alignments before binning
- name: Bioconductor
  role: Ecosystem (e.g. GRanges-based binning via getBins/countBamInGRanges) that
    histoneHMM integrates with
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:e0152cefafcefc273132b8ece3d370228670e074816ee4426e24931a9f3ed6e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matthiasheinig/histoneHMM
  tools:
  - Bioconductor
  - C++
  - R
  - histoneHMM
  - samtools
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# histone-modification-differential-region-calling

## Summary

Apply histoneHMM, a bivariate Hidden Markov Model R/C++ package, to call differentially modified histone regions between two ChIP-seq samples with broad genomic footprints (e.g. H3K27me3, H3K9me3). The method aggregates short reads into 1000 bp binned bivariate read counts and probabilistically classifies bins as modified in both, unmodified in both, or differentially modified.

## When to use

Use when comparing ChIP-seq profiles of broad diffuse histone marks (e.g. H3K27me3, H3K9me3, H3K36me3, H3K79me2) between two samples/cell lines and peak-style callers would yield false positives/negatives.

## When NOT to use

- The histone mark has sharp, peak-like footprints rather than broad diffuse domains — methods that search for peak-like features are the intended use case, and applying peak-style reasoning here generates many false positive or false negative calls
- Only a single sample needs modification calling without a two-sample comparison, and the univariate mode (run.univariate.hmm) alone would suffice
- The input is already a binned read-count feature table with no underlying BAM alignments for the comparison of interest

## Inputs

- Aligned ChIP-seq BAM files for two samples/cell lines per histone mark (with .bai indexes), e.g. ENCODE hg19 BAMs or ENA SRP018808 mouse liver replicates
- Chromosome lengths table (e.g. chroms.txt) for genome binning
- Optionally gene annotation GFF and expression table for preprocess.for.hmm
- R environment with histoneHMM installed from github.com/matthiasheinig/histoneHMM and samtools for deduplication

## Outputs

- Per-bin probabilistic classifications: modified in both samples, unmodified in both, or differentially modified
- Differentially modified region coordinates (e.g. TSV/BED of merged bins) per histone mark
- Total differential-region span per mark (e.g. 121.89 Mb = 4.6% of mouse genome for H3K9me3; 843.03 Mb H3K9me3, 284.58 Mb H3K79me2, 324.85 Mb H3K36me3, 591.27 Mb H3K27me3 for H1-hESC vs K562)

## How to apply

Remove duplicated reads from the aligned BAM files with samtools, then partition the reference genome into 1000 bp bins using getBins and count reads per bin per replicate with countBamInGRanges (with min.mapq filtering). Truncate all read counts at the upper 0.1 percentile (maxq=0.999) to avoid artifacts from extreme read counts and numerical problems with small emission probabilities. Fit the bivariate HMM with run.bivariate.hmm: mixture parameters are estimated via the EM algorithm, transition probabilities between states via the Baum-Welch algorithm, and per-bin enrichment posteriors via the forward-backward algorithm. Call differentially modified bins with get.differential.regions, merge adjacent bins, and report the summed differential-region length (e.g. Mb or % of genome) as the headline result.

## Related tools

- **histoneHMM** (Bivariate HMM that bins the genome, fits emission/transition parameters, and calls differentially modified regions between two samples) — https://github.com/matthiasheinig/histoneHMM
- **R** (Computing environment in which histoneHMM runs and from which the binning/HMM functions are invoked)
- **samtools** (Removes duplicated reads from ChIP-seq BAM alignments before binning)
- **Bioconductor** (Ecosystem (e.g. GRanges-based binning via getBins/countBamInGRanges) that histoneHMM integrates with)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM")
```

## Evaluation signals

- Summed widths of merged differential regions fall within a small tolerance of published Table 2 values (e.g. ~121.89 Mb / 4.6% of the mouse genome for CD-1 liver H3K9me3)
- ENCODE H1-hESC vs K562 totals match Table 2: 843.03 Mb (H3K9me3), 284.58 Mb (H3K79me2), 324.85 Mb (H3K36me3), 591.27 Mb (H3K27me3), i.e. 9%-26% of the human genome
- Read counts were truncated at the upper 0.1 percentile and duplicated reads removed before HMM fitting (maxq=0.999 consistent with the truncation)
- Output contains three-state posterior classifications per bin (modified both / unmodified both / differential) rather than only binary peak calls
- Differential regions span a plausible fraction of the genome for broad marks, not a small peak-like fraction

## Limitations

- Read counts above the upper 0.1 percentile must be truncated to avoid artifacts from regions with extreme read counts and numerical problems with very small emission probabilities
- Methods designed for peak-like features (including naive application of this pipeline's expectations) perform poorly on broad diffuse modifications such as H3K27me3, generating false positives or false negatives
- Published ENCODE GEO accession lists contain duplicates, making the exact sample list ambiguous; no software version number or archival DOI is stated for histoneHMM

## Evidence

- [discussion] Binning and model steps: "we partition the genome into m equally sized bins (1000 bp by default)"
- [discussion] Truncation rationale: "In order to avoid artifacts from regions with extreme read counts [40] and to avoid numerical problems with very small emission probabilities we truncated read counts at the upper 0.1 percentile"
- [discussion] Deduplication step: "and removed duplicated reads using samtools"
- [discussion] Parameter estimation algorithms: "Parameter estimates are obtained via the EM algorithm [43]"
- [readme] Installation from README: "devtools::install_github("matthiasheinig/histoneHMM")"
- [readme] Scope from README: "It allows for calling modified regions in single samples as well as for calling differentially modified regions in a comparison of two samples."
- [intro] Peak-caller failure mode: "Application of methods that search for peak-like features in such data can generate many false positive or false negative calls."
