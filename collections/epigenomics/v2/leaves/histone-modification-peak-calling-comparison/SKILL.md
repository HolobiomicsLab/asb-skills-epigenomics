---
name: histone-modification-peak-calling-comparison
description: Use when a qPCR-validated or otherwise gold-standard ChIP-seq dataset
  of a broad histone mark (e.g. H3K27me3, H3K9me3) is available and you need to quantify
  which peak caller detects modified versus unmodified regions via sensitivity-specificity/ROC
  analysis.
when_to_use_negative:
- The mark has narrow, peak-like footprints (e.g. transcription factor binding or
  H3K4me3-like sharp peaks) — methods that search for peak-like features were designed
  for that regime, and the paper's rationale applies to broad diffuse modifications
  like H3K27me3
- The goal is differential calling between two samples rather than single-sample modified-region
  detection — histoneHMM's bivariate differential mode, not this ROC benchmark, is
  the right tool
- No gold-standard (qPCR-validated) region set exists, so sensitivity/specificity
  cannot be computed against a truth set
edam_operation: http://edamontology.org/operation_3577
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: histoneHMM
  role: Univariate HMM caller whose posterior bin scores are swept to build the ROC
    curve being benchmarked
  repo: https://github.com/matthiasheinig/histoneHMM
- name: samtools
  role: Removes duplicated reads from the ChIP-seq BAM files before binning and HMM
    fitting
- name: MACS2
  role: Competing peak caller run in broad mode with author-recommended defaults
- name: Zinba
  role: Competing peak caller (fitzinba) run with author-recommended default parameters
- name: SICER
  role: Competing broad-domain caller run with author-recommended default parameters
- name: BroadPeak
  role: Competing broad-peak caller run with author-recommended default parameters
- name: Rseg
  role: Competing broad-domain segmentation tool run with author-recommended default
    parameters
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:8bdad34c03afb380f3d04a6dd719096f172f6e5ce5914da5ca97942d8c8da964
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matthiasheinig/histoneHMM
  tools:
  - Bioconductor
  - BroadPeak
  - C++
  - MACS2
  - R
  - Rseg
  - SICER
  - Zinba
  - histoneHMM
  - samtools
  edam_operation: http://edamontology.org/operation_3577
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# histone-modification-peak-calling-comparison

## Summary

Benchmark histoneHMM's single-sample HMM-based calling of broad histone modifications (e.g. H3K27me3) against competing peak callers (MACS2 broad, Zinba, SICER, BroadPeak, Rseg) by sweeping score thresholds and building ROC curves over qPCR-validated regions.

## When to use

Use when a qPCR-validated or otherwise gold-standard ChIP-seq dataset of a broad histone mark (e.g. H3K27me3, H3K9me3) is available and you need to quantify which peak caller best detects modified versus unmodified regions via sensitivity-specificity/ROC analysis.

## When NOT to use

- The mark has narrow, peak-like footprints (e.g. transcription factor binding or H3K4me3-like sharp peaks) — methods that search for peak-like features were designed for that regime, and the paper's rationale applies to broad diffuse modifications like H3K27me3
- The goal is differential calling between two samples rather than single-sample modified-region detection — histoneHMM's bivariate differential mode, not this ROC benchmark, is the right tool
- No gold-standard (qPCR-validated) region set exists, so sensitivity/specificity cannot be computed against a truth set

## Inputs

- ChIP-seq BAM files for the modified sample (e.g. GEO GSM721294, GSM721306, aligned to hg19)
- List of 197 qPCR-validated H3K27me3 regions as the gold-standard set
- Genome partition specification (1000 bp bin size, chromosome lengths)
- SAMtools for duplicate removal

## Outputs

- Per-bin posterior enrichment probabilities from hmm.posterior for histoneHMM
- Called regions from each competitor (MACS2 broad, Zinba, SICER, BroadPeak, Rseg)
- ROC curves of per-base-pair sensitivity vs specificity per method
- Comparison figure (e.g. figure_5a_roc_comparison.png)

## How to apply

Download the ChIP-seq BAM files (here GSM721294 and GSM721306 for the MB H3K27me3 dataset) and remove duplicated reads with samtools. Partition the genome into 1000 bp bins with getBins, count reads per bin with countBamInGRanges, and truncate read counts at the upper 0.1 percentile to avoid artifacts from extreme counts before HMM fitting. Fit the univariate HMM on the binned counts with run.univariate.hmm using EM-based mixture estimation and Baum-Welch transition learning, then score every bin with hmm.posterior. Run each competitor (MACS2 in broad mode, Zinba/fitzinba, SICER, BroadPeak, Rseg) on the same BAM files with the default parameters and thresholds recommended by each tool's authors, since those reflect what an end user would obtain. Sweep score thresholds for every method and compute per-base-pair sensitivity and specificity against the 197 qPCR-validated H3K27me3 regions to build the ROC curves, reproducing Figure 5a.

## Related tools

- **histoneHMM** (Univariate HMM caller whose posterior bin scores are swept to build the ROC curve being benchmarked) — https://github.com/matthiasheinig/histoneHMM
- **samtools** (Removes duplicated reads from the ChIP-seq BAM files before binning and HMM fitting)
- **MACS2** (Competing peak caller run in broad mode with author-recommended defaults)
- **Zinba** (Competing peak caller (fitzinba) run with author-recommended default parameters)
- **SICER** (Competing broad-domain caller run with author-recommended default parameters)
- **BroadPeak** (Competing broad-peak caller run with author-recommended default parameters)
- **Rseg** (Competing broad-domain segmentation tool run with author-recommended default parameters)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM"); bins <- getBins(genome, chr, bin.size=1000); counts <- countBamInGRanges(bam.file, bins); res <- run.univariate.hmm(fname, data)
```

## Evaluation signals

- histoneHMM's ROC curve should dominate the competing callers — higher per-base-pair sensitivity at matched specificity — consistent with published Figure 5a
- Read counts are truncated at the upper 0.1 percentile before HMM fitting; confirm no bin exceeds that value
- Genome partitioning uses 1000 bp bins (getBins) and read counts per bin come from countBamInGRanges; bin count should match the genome length divided by bin size per chromosome
- Competitors must be run with the default parameters and thresholds recommended by their authors, not re-tuned
- Each method's threshold sweep must cover its full score range so the ROC curve spans the achievable sensitivity-specificity trade-off

## Limitations

- No version numbers are given for Chipdiff, rseg, Diffreps, Pepr, Sicer, Broadpeak, and samtools, complicating exact reproduction
- histoneHMM is distributed via a project website/GitHub without a versioned archival DOI; the README notes version 1.7 works with Rcpp > 0.12 and version 1.6 removed the GNU scientific library dependency
- Benchmark validity depends entirely on the 197 qPCR-validated regions; methods may behave differently outside this truth set

## Evidence

- [results] In the sensitivity-specificity analysis on the qPCR-validated H3K27me3 dataset, histoneHMM outperformed the competing algorithms in detecting modified versus unmodified regions, as shown by the ROC curves in Figure 5a.: [evidence span withheld — anchor status: not_found]
- [discussion] Binning and counting steps: "we partition the genome into m equally sized bins (1000 bp by default)"
- [discussion] Read-count truncation filter: "we truncated read counts at the upper 0.1 percentile"
- [discussion] Duplicate removal: "and removed duplicated reads using samtools"
- [discussion] Author-default competitor settings: "We mostly used the default parameters and thresholds as recommended by the authors since these results are likely those that an end user would also obtain"
- [discussion] Dataset accessions: "(GEO accessions GSM721294, GSM721306)"
- [readme] Installation from README: "devtools::install_github("matthiasheinig/histoneHMM")"
