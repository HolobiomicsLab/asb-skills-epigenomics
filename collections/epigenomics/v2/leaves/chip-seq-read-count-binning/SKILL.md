---
name: chip-seq-read-count-binning
description: Use when preparing raw ChIP-seq BAM files of broad-footprint histone
  modifications (e.g. H3K27me3) for histoneHMM differential or single-sample calling,
  where per-bin bivariate read counts must be computed before HMM fitting.
when_to_use_negative:
- The modification has sharp, peak-like footprints rather than broad diffuse marks
  — the article notes peak-searching methods are the wrong model for broad marks like
  H3K27me3, and conversely binning-based histoneHMM is aimed at broad footprints
- Input is already a binned read-count table (e.g. the distributed BN.txt/SHR.txt
  preprocessed data) — skip directly to HMM fitting
- A caller requiring per-base or peak-level resolution is needed, since aggregation
  into 1000 bp bins deliberately discards sub-bin resolution
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: R/Bioconductor package providing getBins, countBamInGRanges, countBedInGRanges,
    and preprocess.for.hmm for binning the genome and aggregating per-bin read counts
  repo: https://github.com/matthiasheinig/histoneHMM
- name: samtools
  role: Removes duplicated reads from BAM alignments before bin counting
- name: R
  role: Computing environment in which the histoneHMM binning and counting functions
    run
- name: Bioconductor
  role: Bioinformatics infrastructure (GenomicRanges/BAM handling) that histoneHMM
    integrates with
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:426bb8edc71cb5e3fff20b29a1b303d6b9abe8e96c5fa88fb84a43febba525a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matthiasheinig/histoneHMM
  tools:
  - Bioconductor
  - R
  - histoneHMM
  - samtools
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# chip-seq-read-count-binning

## Summary

Partition a genome into equally sized 1000 bp bins and aggregate ChIP-seq short-read counts per bin per sample, producing the binned bivariate read-count table that histoneHMM requires as input for its unsupervised HMM classification of broad histone marks like H3K27me3.

## When to use

Use when preparing raw ChIP-seq BAM files of broad-footprint histone modifications (e.g. H3K27me3) for histoneHMM differential or single-sample calling, where per-bin bivariate read counts must be computed before HMM fitting.

## When NOT to use

- The modification has sharp, peak-like footprints rather than broad diffuse marks — the article notes peak-searching methods are the wrong model for broad marks like H3K27me3, and conversely binning-based histoneHMM is aimed at broad footprints
- Input is already a binned read-count table (e.g. the distributed BN.txt/SHR.txt preprocessed data) — skip directly to HMM fitting
- A caller requiring per-base or peak-level resolution is needed, since aggregation into 1000 bp bins deliberately discards sub-bin resolution

## Inputs

- Deduplicated ChIP-seq alignment files (BAM with .bai index, e.g. BN.bam/SHR.bam) for each sample
- Chromosome-length table (e.g. chroms.txt) defining the reference genome bins (rat rn4)
- Optional BED-format alignments (via countBedInGRanges)
- histoneHMM package installed via devtools::install_github("matthiasheinig/histoneHMM")

## Outputs

- Genomic ranges object of 1000 bp bins covering the reference genome (from getBins)
- Per-sample binned read-count table (e.g. BN.txt / SHR.txt style) used as HMM input
- Truncated read counts capped at the upper 0.1 percentile

## How to apply

Remove duplicated reads from the aligned BAM files using samtools, then partition the reference genome (e.g. rat rn4) into m equally sized bins of 1000 bp — the histoneHMM default — using getBins with a chromosome-length table such as chroms.txt. Count reads falling into each bin for each sample with countBamInGRanges (or countBedInGRanges for BED input), invoked through the preprocess.for.hmm wrapper, honoring min.mapq and read.width parameters. To avoid artifacts from extreme read counts and numerical problems with very small emission probabilities, truncate all per-bin read counts greater than the upper 0.1 percentile to the value of that percentile. The resulting binned count tables (e.g. the packaged BN.txt/SHR.txt test data) become the bivariate input for downstream HMM fitting (run.bivariate.hmm or run.univariate.hmm); this aggregation over larger regions is what makes peak-style callers unsuitable and enables the unsupervised, tuning-parameter-free classification of broad marks.

## Related tools

- **histoneHMM** (R/Bioconductor package providing getBins, countBamInGRanges, countBedInGRanges, and preprocess.for.hmm for binning the genome and aggregating per-bin read counts) — https://github.com/matthiasheinig/histoneHMM
- **samtools** (Removes duplicated reads from BAM alignments before bin counting)
- **R** (Computing environment in which the histoneHMM binning and counting functions run)
- **Bioconductor** (Bioinformatics infrastructure (GenomicRanges/BAM handling) that histoneHMM integrates with)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM")
```

## Evaluation signals

- Bin table covers the genome in m equally sized bins of 1000 bp (default), consistent with the supplied chroms.txt chromosome lengths
- Per-bin counts exist for every sample and no bin has a count exceeding the upper 0.1 percentile truncation value
- Count table format matches the preprocessed test data (BN.txt/SHR.txt) so run.bivariate.hmm / preprocess.for.hmm downstream calls accept it without re-tuning
- Reproducing the pipeline on the packaged SHR/BN heart data yields a total differentially modified span consistent with the reported 24.96 Mb (0.9% of the rat genome)
- Duplicate reads were removed (samtools) prior to counting, so per-bin counts reflect deduplicated alignments

## Limitations

- Read counts greater than the upper 0.1 percentile must be truncated; without this, extreme-count regions cause artifacts and numerical problems with very small emission probabilities
- SNP-driven allele-specific effects require special handling — the article aligned SHR samples against a BN reference genome with known SNP positions substituted by SHR alleles
- The 1000 bp bin size is the default tuned for broad marks; the article does not validate binning parameters for peak-like modifications

## Evidence

- [discussion] we partition the genome into m equally sized bins (1000 bp by default): "we partition the genome into m equally sized bins (1000 bp by default)"
- [discussion] All read counts greater than the upper 0.1 percentile were set to the value of the upper 0.1 percentile: "All read counts greater than the upper 0.1 percentile were set to the value of the upper 0.1 percentile"
- [discussion] In order to avoid artifacts from regions with extreme read counts [40] and to avoid numerical problems with very small emission probabilities we truncated read counts at the upper 0.1 percentile: "to avoid artifacts from regions with extreme read counts [40] and to avoid numerical problems with very small emission probabilities we truncated read counts at the upper 0.1 percentile"
- [methods] preprocess.for.hmm(signal.bam, genes, bin.size, genome, chr, expr, offset, n.expr.bins, plot): "preprocess.for.hmm(signal.bam, genes, bin.size, genome, chr, expr, offset, n.expr.bins, plot)"
- [readme] We provide a small test data set to try out the package: [BN.txt](http://histonehmm.molgen.mpg.de/data/BN.txt): "We provide a small test data set to try out the package:"
