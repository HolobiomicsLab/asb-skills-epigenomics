---
name: genome-wide-differential-region-validation
description: Use when reproducing or validating published genome-wide differential
  ChIP-seq region calls for broad histone marks (e.g., H3K9me3/H3K27me3) with histoneHMM
  and a quantitative target such as total differential Mb or genome fraction.
when_to_use_negative:
- The histone mark produces sharp peak-like footprints rather than broad diffuse modifications,
  where peak-calling methods are more appropriate
- The input is already a list of called regions or a feature table rather than replicate
  BAM alignments, so no binning/HMM fitting is needed
- Only a single-sample modification call is required, not a two-sample differential
  comparison
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: Bivariate HMM that bins the genome (1000 bp default), classifies bins as modified/unmodified/differential,
    and calls differentially modified regions via get.differential.regions
  repo: github.com/matthiasheinig/histoneHMM
- name: samtools
  role: Removes duplicated reads from replicate alignments prior to binning and read
    counting
- name: R/Bioconductor
  role: Environment in which histoneHMM runs and through which getBins and countBamInGRanges
    (min.mapq filtering) are invoked
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:a202e8e7feed027b32ca8345a854364766fdbaef1cb888b5c745ab16636def6c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - R/Bioconductor
  - histoneHMM
  - samtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# genome-wide-differential-region-validation

## Summary

Validates genome-wide differential histone-modification region calls by reproducing a histoneHMM bivariate HMM run (e.g., H3K9me3 male vs female CD-1 mouse liver ChIP-seq from ENA SRP018808) and checking that the merged differential-region span reproduces the reported value (~121.89 Mb, 4.6% of the mouse genome).

## When to use

Use when reproducing or validating published genome-wide differential ChIP-seq region calls for broad histone marks (e.g., H3K9me3/H3K27me3) with histoneHMM and a quantitative target such as total differential Mb or genome fraction.

## When NOT to use

- The histone mark produces sharp peak-like footprints rather than broad diffuse modifications, where peak-calling methods are more appropriate
- The input is already a list of called regions or a feature table rather than replicate BAM alignments, so no binning/HMM fitting is needed
- Only a single-sample modification call is required, not a two-sample differential comparison

## Inputs

- Replicate ChIP-seq BAM alignment files for the two conditions (e.g., female and male CD-1 mouse liver H3K9me3 from ENA SRP018808)
- Chromosome-length/genome definition for binning (e.g., mouse genome partitioned into 1000 bp bins)
- samtools for duplicate removal
- Published target statistic for validation (e.g., Table 2 total differential region length in Mb and % genome)

## Outputs

- differential_regions_CD1_H3K9me3.tsv with merged differentially modified region coordinates
- Total differential-region span in Mb and percent of genome (target ≈ 121.89 Mb / 4.6%)
- Fitted bivariate HMM model with EM mixture parameters, Baum-Welch transition probabilities, and forward-backward posteriors

## How to apply

Download replicate BAM alignments from the cited accession (e.g., ENA SRP018808 CD-1 mouse liver H3K9me3 ChIP-seq), remove duplicated reads with samtools, and install histoneHMM via devtools::install_github('matthiasheinig/histoneHMM'). Partition the genome into 1000 bp bins with getBins (default bin.size=1000) and count reads per bin per replicate with countBamInGRanges using min.mapq filtering. Truncate all read counts at the upper 0.1 percentile to avoid artifacts from extreme counts and numerical problems with very small emission probabilities. Fit the bivariate HMM with run.bivariate.hmm, which estimates emission mixture parameters via EM, transition probabilities via Baum-Welch, and per-bin enrichment posteriors via the forward-backward algorithm. Call differentially modified regions with get.differential.regions using maxq=0.999 (matching the 0.1 percentile truncation), merge adjacent bins, and validate by summing region widths against the published total (e.g., 121.89 Mb ≈ 4.6% of the mouse genome) within a small tolerance.

## Related tools

- **histoneHMM** (Bivariate HMM that bins the genome (1000 bp default), classifies bins as modified/unmodified/differential, and calls differentially modified regions via get.differential.regions) — github.com/matthiasheinig/histoneHMM
- **samtools** (Removes duplicated reads from replicate alignments prior to binning and read counting)
- **R/Bioconductor** (Environment in which histoneHMM runs and through which getBins and countBamInGRanges (min.mapq filtering) are invoked)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM"); bins <- getBins(genome, bin.size=1000); counts <- countBamInGRanges(bam.file, bins, min.mapq); regions <- get.differential.regions(run.bivariate.hmm(counts), maxq=0.999)
```

## Evaluation signals

- Summed width of merged differential regions falls within a small tolerance of the reported value (121.89 Mb, i.e., ~4.6% of the mouse genome for CD-1 H3K9me3)
- All read-count values in the binned table do not exceed the upper 0.1 percentile (truncation applied)
- Binning uses 1000 bp bins (getBins default bin.size=1000) and get.differential.regions is run with maxq=0.999 consistent with the truncation
- Duplicated reads are removed with samtools before counting, and countBamInGRanges applies min.mapq filtering
- Output TSV contains contiguous genomic coordinates (chromosome, start, end) that merge adjacent called bins without overlaps

## Limitations

- Read counts must be truncated at the upper 0.1 percentile to avoid artifacts from extreme read counts and numerical problems with very small emission probabilities; skipping this changes the maxq setting and results
- histoneHMM is designed for modifications with broad genomic footprints (H3K9me3, H3K27me3); applying peak-oriented assumptions yields false positives/negatives
- The package provides no version number or archival DOI in the article; the README notes recent versions changed dependencies (v1.6 removed the GNU scientific library dependency; v1.7 requires Rcpp > 0.12), so reproducibility may depend on the installed version

## Evidence

- [discussion] we partition the genome into m equally sized bins (1000 bp by default): "we partition the genome into m equally sized bins (1000 bp by default)"
- [discussion] Truncation rationale: "In order to avoid artifacts from regions with extreme read counts [40] and to avoid numerical problems with very small emission probabilities we truncated read counts at the upper 0.1 percentile"
- [discussion] Parameter estimation algorithms: "Parameter estimates are obtained via the EM algorithm [43]"
- [discussion] Transition estimation: "We use the Baum-Welch algorithm [44] to determine the transition probabilities between states"
- [discussion] Posteriors: "calculate the probability of enrichment for each bin in the genome using the forward-backward algorithm [45]"
- [methods] Package install: "install.packages("devtools")
devtools::install_github("matthiasheinig/histoneHMM")"
- [methods] countBamInGRanges signature: "countBamInGRanges(bam.file, granges, min.mapq, read.width)"
- [discussion] Duplicate removal: "and removed duplicated reads using samtools"
- [readme] Version 1.6 note: "Version 1.6 fixes some smaller bugs and removes the dependency on the GNU scientific library"
