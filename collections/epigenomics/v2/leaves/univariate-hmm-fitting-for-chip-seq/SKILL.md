---
name: univariate-hmm-fitting-for-chip-seq
description: Use when you have duplicated-read-removed ChIP-seq BAM files for a single
  sample of a broad histone mark (e.g. H3K27me3) and need genome-wide probabilistic
  calls of modified versus unmodified regions rather than peak-like summits.
when_to_use_negative:
- Input ChIP-seq targets peak-like transcription factor binding rather than broad
  diffuse marks like H3K27me3 or H3K9me3, where peak callers are appropriate
- The analysis goal is comparing two samples to find differentially modified regions
  — use histoneHMM's bivariate model instead of the univariate fit
- Read counts have not been deduplicated or truncated at the upper 0.1 percentile,
  which the authors state causes artifacts and numerical problems
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0102
tools:
- name: histoneHMM
  role: Provides getBins, countBamInGRanges, preprocess.for.hmm, run.univariate.hmm,
    hmm.baumwelch, hmm.posterior and hmm.viterbi for the binned univariate HMM fit
    and scoring
  repo: http://histonehmm.molgen.mpg.de
- name: samtools
  role: Removes duplicated reads from BAM files before binning and counting
- name: R
  role: Computing environment in which the C++-backed histoneHMM package is run
  repo: http://histonehmm.molgen.mpg.de
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:270edac1c864b93b4b444a291f0760a761307f463b027e618121d9ca87e68d22
schema_version: 0.2.0
metadata:
  license_tier: open
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# univariate-hmm-fitting-for-chip-seq

## Summary

Fit a single-sample (univariate) Hidden Markov Model over 1000 bp genomic bins of ChIP-seq read counts to classify bins as modified vs. unmodified for broad histone marks like H3K27me3, using histoneHMM's EM + Baum-Welch estimation and forward-backward posterior scoring.

## When to use

Use when you have duplicated-read-removed ChIP-seq BAM files for a single sample of a broad histone mark (e.g. H3K27me3) and need genome-wide probabilistic calls of modified versus unmodified regions rather than peak-like summits.

## When NOT to use

- Input ChIP-seq targets peak-like transcription factor binding rather than broad diffuse marks like H3K27me3 or H3K9me3, where peak callers are appropriate
- The analysis goal is comparing two samples to find differentially modified regions — use histoneHMM's bivariate model instead of the univariate fit
- Read counts have not been deduplicated or truncated at the upper 0.1 percentile, which the authors state causes artifacts and numerical problems

## Inputs

- Deduplicated ChIP-seq BAM file (with .bai index) for a single sample, e.g. GSM721294 or GSM721306 H3K27me3 MB data
- Genome definition / chromosome length table (e.g. chroms.txt) and genome assembly identifier (e.g. hg19 or rn4)
- Optional gene annotation in GFF format (e.g. ensembl59-genes.gff) and RNA-seq expression table (e.g. expression.txt) for expression-aware binning

## Outputs

- Per-bin enrichment posterior probabilities from hmm.posterior
- Per-bin state calls (modified vs. unmodified) via hmm.viterbi or posterior thresholding
- Diagnostic preprocessing/HMM fit plots produced by the plot option of preprocess.for.hmm and run.univariate.hmm

## How to apply

Remove duplicated reads with samtools, then partition the genome into equally sized bins (1000 bp by default) using getBins and count reads per bin with countBamInGRanges. To avoid numerical problems with very small emission probabilities, truncate all read counts greater than the upper 0.1 percentile to that value via preprocess.for.hmm. Fit the model with run.univariate.hmm, which estimates emission mixture parameters via the EM algorithm and transition probabilities between states via the Baum-Welch algorithm (hmm.baumwelch); set n.expr.bins to hold out expression-associated bins from the mixture fit if using the expression-aware mode. Score each bin's enrichment probability with hmm.posterior (forward-backward), or assign hard states with hmm.viterbi, and threshold posterior scores to derive modified-region calls. The rationale is that aggregating short reads over larger regions captures the diffuse, broad footprints of marks like H3K27me3 where peak-calling methods generate false positives and negatives.

## Related tools

- **histoneHMM** (Provides getBins, countBamInGRanges, preprocess.for.hmm, run.univariate.hmm, hmm.baumwelch, hmm.posterior and hmm.viterbi for the binned univariate HMM fit and scoring) — http://histonehmm.molgen.mpg.de
- **samtools** (Removes duplicated reads from BAM files before binning and counting)
- **R** (Computing environment in which the C++-backed histoneHMM package is run) — http://histonehmm.molgen.mpg.de

## Examples

```
library(histoneHMM); bins <- getBins(genome, chr, bin.size=1000); counts <- countBamInGRanges("sample.bam", bins); fit <- run.univariate.hmm(fname, counts, n.expr.bins, em=TRUE, baum.welch=TRUE)
```

## Evaluation signals

- Preprocessed read counts must be capped: all counts above the upper 0.1 percentile equal the value of the 0.1 percentile after preprocess.for.hmm
- Binning uses 1000 bp equally sized genome-wide bins (getBins default), so the count matrix length matches the expected number of bins for the chromosome length table
- EM and Baum-Welch iterations should converge within max.it at tolerance eps before hmm.posterior scores are computed
- Posterior probabilities are well-calibrated probabilities per bin; sweeping thresholds on the qPCR-validated H3K27me3 set (197 regions) should yield ROC curves that dominate competing callers (MACS2 broad, Zinba, SICER, BroadPeak, Rseg), consistent with published Figure 5a

## Limitations

- Truncation at the upper 0.1 percentile is required to avoid artifacts from extreme read counts and numerical problems with very small emission probabilities
- Model quality depends on the mixture/transition parameter setting; the authors state simulation studies in Additional file 1 justified the setting, but its contents are not included in the document
- histoneHMM is distributed without a stated version number or archival DOI in the article text (README lists versions up to 1.7, which requires Rcpp > 0.12)

## Evidence

- [discussion] we partition the genome into m equally sized bins (1000 bp by default): "we partition the genome into m equally sized bins (1000 bp by default)"
- [discussion] Truncation of read counts: "we truncated read counts at the upper 0.1 percentile"
- [discussion] EM parameter estimation: "Parameter estimates are obtained via the EM algorithm [43]"
- [discussion] Baum-Welch transition learning: "We use the Baum-Welch algorithm [44] to determine the transition probabilities between states"
- [discussion] Forward-backward posterior scoring: "calculate the probability of enrichment for each bin in the genome using the forward-backward algorithm [45]"
- [methods] Univariate HMM entry point signature: "run.univariate.hmm(fname, data, n.expr.bins, em, chrom, maxq, redo, model.constructor, baum.welch, plot)"
- [methods] Read counting into bins: "countBamInGRanges(bam.file, granges, min.mapq, read.width)"
- [readme] Single-sample calling scope from README: "It allows for calling modified regions in single samples as well as for calling differentially modified regions in a comparison of two samples."
- [readme] Package installation from README: "devtools::install_github("matthiasheinig/histoneHMM")"
