---
name: roc-evaluation-against-qpcr-validated-regions
description: Use when a modified-vs-unmodified region caller on broad histone marks
  (e.g. H3K27me3) must be ranked against competitors using an independent qPCR-validated
  ground-truth region set rather than simulated data.
when_to_use_negative:
- The task is differential calling between two samples (the bivariate histoneHMM use
  case) rather than single-sample modified-vs-unmodified detection benchmarking
- No independent qPCR-validated or otherwise experimentally validated region set exists,
  so sensitivity/specificity cannot be computed against ground truth (the article
  used simulation studies, Additional file 1, in that setting)
- The histone mark of interest forms sharp peak-like features rather than broad diffuse
  footprints like H3K27me3/H3K9me3, where peak-style callers are appropriate and this
  broad-mark benchmark does not apply
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0160
tools:
- name: histoneHMM
  role: Method under evaluation; its univariate HMM (run.univariate.hmm, hmm.posterior)
    produces per-bin posterior scores that are threshold-swept for the ROC analysis
  repo: https://github.com/matthiasheinig/histoneHMM
- name: MACS2
  role: Competing peak caller run in broad mode on the same BAMs with author-recommended
    defaults, to generate comparison ROC curves
- name: Zinba
  role: Competing peak caller (fitzinba) run with author-recommended defaults for
    the ROC comparison
- name: SICER
  role: Competing broad-mark caller run with author-recommended defaults for the ROC
    comparison
- name: BroadPeak
  role: Competing broad-mark caller run with author-recommended defaults for the ROC
    comparison
- name: Rseg
  role: Competing broad-mark caller run with author-recommended defaults for the ROC
    comparison
- name: samtools
  role: Removes duplicated reads from the GSM721294/GSM721306 BAMs before binning
    and calling
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:dac98ceda2e15f348127ca05347d492da38dd792a93b902f05030f843cec013e
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
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# roc-evaluation-against-qpcr-validated-regions

## Summary

Benchmark a ChIP-seq peak caller or HMM-based caller by sweeping its score thresholds and building per-base-pair ROC curves of sensitivity versus specificity against an independent qPCR-validated region set (here 197 H3K27me3 regions from GSM721294/GSM721306), as done to show histoneHMM outperforms MACS2 broad, Zinba, SICER, BroadPeak and Rseg in Figure 5a.

## When to use

Use when a modified-vs-unmodified region caller on broad histone marks (e.g. H3K27me3) must be ranked against competitors using an independent qPCR-validated ground-truth region set rather than simulated data.

## When NOT to use

- The task is differential calling between two samples (the bivariate histoneHMM use case) rather than single-sample modified-vs-unmodified detection benchmarking
- No independent qPCR-validated or otherwise experimentally validated region set exists, so sensitivity/specificity cannot be computed against ground truth (the article used simulation studies, Additional file 1, in that setting)
- The histone mark of interest forms sharp peak-like features rather than broad diffuse footprints like H3K27me3/H3K9me3, where peak-style callers are appropriate and this broad-mark benchmark does not apply

## Inputs

- ChIP-seq BAM files for the qPCR-validated H3K27me3 dataset (GEO GSM721294 and GSM721306)
- Set of 197 qPCR-validated H3K27me3 regions used as ground truth (positive class)
- Per-bin or per-region scores from each caller: histoneHMM hmm.posterior probabilities, plus MACS2 broad, Zinba, SICER, BroadPeak and Rseg outputs on the same BAMs

## Outputs

- Sweep of (sensitivity, specificity) pairs per method at per-base-pair resolution
- ROC curves comparing histoneHMM against all competing callers, saved as figure_5a_roc_comparison.png

## How to apply

Preprocess the ChIP-seq BAMs (GSM721294/GSM721306) by removing duplicated reads with samtools, then run each method — histoneHMM's univariate HMM and the competitors MACS2 (broad mode), Zinba (fitzinba), SICER, BroadPeak and Rseg — with the default parameters and thresholds recommended by each tool's authors. For histoneHMM, partition the genome into 1000 bp bins via getBins, count reads with countBamInGRanges, truncate read counts at the upper 0.1 percentile, and score bins with hmm.posterior after run.univariate.hmm (EM-based mixture estimation, Baum-Welch transition learning). For every method, sweep the score threshold across its full range and, at each threshold, compute per-base-pair sensitivity and specificity against the 197 qPCR-validated H3K27me3 regions, treating validated regions as positives. Plot the resulting ROC curves of all methods together (reproducing Figure 5a) and judge correctness by whether the evaluated method's curve dominates or matches the published ordering — histoneHMM should show higher per-base-pair sensitivity at matched specificity than the competing callers.

## Related tools

- **histoneHMM** (Method under evaluation; its univariate HMM (run.univariate.hmm, hmm.posterior) produces per-bin posterior scores that are threshold-swept for the ROC analysis) — https://github.com/matthiasheinig/histoneHMM
- **MACS2** (Competing peak caller run in broad mode on the same BAMs with author-recommended defaults, to generate comparison ROC curves)
- **Zinba** (Competing peak caller (fitzinba) run with author-recommended defaults for the ROC comparison)
- **SICER** (Competing broad-mark caller run with author-recommended defaults for the ROC comparison)
- **BroadPeak** (Competing broad-mark caller run with author-recommended defaults for the ROC comparison)
- **Rseg** (Competing broad-mark caller run with author-recommended defaults for the ROC comparison)
- **samtools** (Removes duplicated reads from the GSM721294/GSM721306 BAMs before binning and calling)

## Examples

```
Rscript -e 'devtools::install_github("matthiasheinig/histoneHMM"); bins <- getBins(genome, chr, bin.size=1000); counts <- countBamInGRanges("GSM721294.bam", bins); fit <- run.univariate.hmm("hmm", counts, em=TRUE, baum.welch=TRUE); post <- hmm.posterior(fit)'
```

## Evaluation signals

- histoneHMM's ROC curve dominates the competitors: higher per-base-pair sensitivity at matched specificity, reproducing the published Figure 5a ordering
- All methods are evaluated on the same 197 qPCR-validated H3K27me3 regions from GSM721294/GSM721306 after duplicate removal, ensuring a fair shared ground truth
- Competing tools (MACS2 broad, Zinba, SICER, BroadPeak, Rseg) are run with the default parameters and thresholds recommended by their authors, matching the end-user scenario
- Threshold sweep spans each method's full score range so the ROC curve is not truncated before its extremes
- Read counts are truncated at the upper 0.1 percentile before HMM scoring, avoiding artifacts from extreme-count bins that would distort the posterior sweep

## Limitations

- Competing tool versions (Chipdiff, rseg, Diffreps, Pepr, Sicer, Broadpeak, samtools) are not stated; parameters were mostly the authors' recommended defaults, so results may vary with newer releases
- Validation depends entirely on the 197 qPCR-validated regions, which cover only a limited set of loci and may not represent genome-wide performance
- Methods designed for peak-like features can generate many false positive or false negative calls on broad diffuse marks, so including them in the benchmark mainly illustrates the broad-footprint failure mode rather than tuning them fairly

## Evidence

- [results] histoneHMM outperformed the competing algorithms in detecting modified versus unmodified regions, as shown by the ROC curves in Figure 5a: [evidence span withheld — anchor status: not_found]
- [discussion] we partition the genome into m equally sized bins (1000 bp by default): "we partition the genome into m equally sized bins (1000 bp by default)"
- [discussion] All read counts greater than the upper 0.1 percentile were set to the value of the upper 0.1 percentile: "All read counts greater than the upper 0.1 percentile were set to the value of the upper 0.1 percentile"
- [discussion] removed duplicated reads using samtools: "and removed duplicated reads using samtools"
- [discussion] We mostly used the default parameters and thresholds as recommended by the authors since these results are likely those that an end user would also obtain: "We mostly used the default parameters and thresholds as recommended by the authors since these results are likely those that an end user would also obtain"
- [discussion] GEO accessions GSM721294, GSM721306: "(GEO accessions GSM721294, GSM721306)"
- [methods] devtools::install_github("matthiasheinig/histoneHMM"): "devtools::install_github("matthiasheinig/histoneHMM")"
