---
name: kmer-variability-deviation-analysis
description: 'Use when scATAC-seq/DNase counts in peaks vary between cells or samples
  and de novo motif discovery is the goal: rank bias-corrected k-mer deviations by
  covariance, assemble PWMs with assembleKmers, then match to JASPAR via pwmDistance.'
when_to_use_negative:
- The task is scoring a curated known-motif collection (e.g., getJasparMotifs/human_pwms_v1)
  rather than discovering de novo motifs — matchMotifs + computeDeviations is the
  appropriate path.
- The primary goal is cell clustering or embedding — the README notes SnapATAC outperforms
  chromVAR for the clustering tasks evaluated; k-mer deviations annotate motif variability
  instead.
- Input has no per-cell/per-sample count variability over peaks (e.g., a bare peak
  FASTA without fragment counts) — the deviation framework requires variability between
  individual cells or samples.
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: chromVAR
  role: Core package supplying addGCBias, filterSamples/filterPeaks, matchKmers, computeDeviations,
    deviationsCovariability, plotKmerMismatch, and assembleKmers for the de novo k-mer
    deviation workflow
  repo: https://github.com/GreenleafLab/chromVAR
- name: motifmatchr
  role: Fast motif/k-mer matching backend (MOODS C++ code) underlying matching of
    6-mers to the filtered peak set
  repo: https://github.com/GreenleafLab/motifmatchr
- name: TFBSTools
  role: toPWM/PWM utilities used to orient the closest known JASPAR motif (reverse
    complement) for comparison with assembled de novo PWMs
- name: ggmotif
  role: Sequence-logo visualization of assembled de novo PWMs and their closest known
    motif matches
  repo: https://github.com/AliciaSchep/ggmotif
- name: BiocParallel
  role: Registers the parallel backend (e.g., register(MulticoreParam(8))) before
    running computeDeviations/assembleKmers
  repo: https://bioconductor.org/packages/release/bioc/html/BiocParallel.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nmeth.4401
    title: 'chromVAR: inferring transcription-factor-associated accessibility from
      single-cell epigenomic data'
  source_content_hash: sha256:ca1f08651c684fd168f803822446a045f7cbd9b1174ecd1e4e0d2137ae1472e7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GreenleafLab/chromVAR
  tools:
  - BiocParallel
  - R
  - TFBSTools
  - chromVAR
  - ggmotif
  - motifmatchr
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# kmer-variability-deviation-analysis

## Summary

Reconstructs chromVAR's de novo motif-discovery module: bias-corrected 6-mer deviations (matchKmers + computeDeviations) are ranked by deviationsCovariability, assembled into PWMs with assembleKmers, and compared to JASPAR motifs via pwmDistance to characterize de novo motifs linked to accessibility variability between cells or samples.

## When to use

Use when scATAC-seq/DNase counts in peaks vary between cells or samples and de novo motif discovery is the goal: rank bias-corrected k-mer deviations by covariance, assemble PWMs with assembleKmers, then match to JASPAR via pwmDistance.

## When NOT to use

- The task is scoring a curated known-motif collection (e.g., getJasparMotifs/human_pwms_v1) rather than discovering de novo motifs — matchMotifs + computeDeviations is the appropriate path.
- The primary goal is cell clustering or embedding — the README notes SnapATAC outperforms chromVAR for the clustering tasks evaluated; k-mer deviations annotate motif variability instead.
- Input has no per-cell/per-sample count variability over peaks (e.g., a bare peak FASTA without fragment counts) — the deviation framework requires variability between individual cells or samples.

## Inputs

- RangedSummarizedExperiment of scATAC-seq/DNase fragment counts in peaks (chromVAR example_counts or getCounts output)
- Peak set with GC bias annotation added via addGCBias against BSgenome.Hsapiens.UCSC.hg19
- Reference genome for k-mer matching: BSgenome.Hsapiens.UCSC.hg19 (hg19)
- Known PWM collection for comparison, e.g., JASPAR motifs via getJasparMotifs or chromVARmotifs human_pwms_v1
- Seed k-mer string for mismatch inspection (workflow uses 'CATTCC')

## Outputs

- K-mer deviation object from computeDeviations over all matched 6-mers
- Normalized k-mer covariance matrix from deviationsCovariability
- Assembled de novo PWM objects from assembleKmers, serialized as de_novo_motifs.rds
- pwmDistance result with 'dist', 'strand', and 'offset' matrices mapping de novo motifs to closest JASPAR motifs
- ggmotif plots of the top de novo motif and its closest known match (including reverse-complement orientation)

## How to apply

Start from a RangedSummarizedExperiment of per-cell fragment counts in peaks (e.g., chromVAR's example_counts), add GC bias via addGCBias with BSgenome.Hsapiens.UCSC.hg19, then apply filterSamples(min_depth = 1500, min_in_peaks = 0.15) and filterPeaks to remove low-depth cells and weak, overlapping peaks. Match all 6-mers to the filtered peaks with matchKmers(6, ..., genome = BSgenome.Hsapiens.UCSC.hg19) and run computeDeviations to obtain bias-corrected k-mer deviations that estimate gain or loss of accessibility while controlling for technical biases. Rank k-mer variability with deviationsCovariability (normalized covariance between k-mer deviations) and inspect mismatch coverage for a seed k-mer such as 'CATTCC' with plotKmerMismatch before assembly. Iteratively assemble de novo PWMs from the most variable k-mers with assembleKmers(kmer_dev, progress = FALSE), weighting each nucleotide by the normalized covariance of bias-corrected deviations for closely related k-mers. Compare assembled PWMs to known motifs with pwmDistance, which returns 'dist', 'strand', and 'offset' matrices — select the closest JASPAR match via which.min — and visualize de novo and known motifs with ggmotif, using TFBSTools toPWM to orient reverse-complement matches. Persist the PWM objects as de_novo_motifs.rds for reuse.

## Related tools

- **chromVAR** (Core package supplying addGCBias, filterSamples/filterPeaks, matchKmers, computeDeviations, deviationsCovariability, plotKmerMismatch, and assembleKmers for the de novo k-mer deviation workflow) — https://github.com/GreenleafLab/chromVAR
- **motifmatchr** (Fast motif/k-mer matching backend (MOODS C++ code) underlying matching of 6-mers to the filtered peak set) — https://github.com/GreenleafLab/motifmatchr
- **TFBSTools** (toPWM/PWM utilities used to orient the closest known JASPAR motif (reverse complement) for comparison with assembled de novo PWMs)
- **ggmotif** (Sequence-logo visualization of assembled de novo PWMs and their closest known motif matches) — https://github.com/AliciaSchep/ggmotif
- **BiocParallel** (Registers the parallel backend (e.g., register(MulticoreParam(8))) before running computeDeviations/assembleKmers) — https://bioconductor.org/packages/release/bioc/html/BiocParallel.html

## Examples

```
kmer_dev <- computeDeviations(counts_filtered, matchKmers(6, counts_filtered, genome = BSgenome.Hsapiens.UCSC.hg19)); de_novo <- assembleKmers(kmer_dev, progress = FALSE)
```

## Evaluation signals

- counts_filtered reflects the filterSamples thresholds min_depth = 1500 and min_in_peaks = 0.15, and filterPeaks leaves a non-overlapping peak set
- deviationsCovariability yields a normalized covariance matrix whose most variable k-mers cluster around the seed 'CATTCC' and its closely related 6-mers
- assembleKmers returns PWM objects that serialize to de_novo_motifs.rds and reload as valid PWM objects
- pwmDistance output contains all three matrices ('dist', 'strand', 'offset'), and which.min(dist) selects a JASPAR motif whose ggmotif logo matches the de novo motif up to reverse-complement orientation
- A BiocParallel backend is explicitly registered (e.g., register(MulticoreParam(8)) or SerialParam) before running the deviation pipeline

## Limitations

- K-mer deviations are for annotating motif/annotation variability, not clustering: the README notes SnapATAC outperforms chromVAR for the clustering tasks evaluated
- Deviations control technical biases via addGCBias; skipping GC-bias correction confounds k-mer variability with sequence composition
- filterPeaks reduces the peak set to non-overlapping peaks, so assembly operates on a reduced peak universe rather than the full peak file
- The workflow matches 6-mers (matchKmers(6, ...)); deviating from k = 6 changes the k-mer space and is not validated in the article
- Installation of chromVAR/motifmatchr dependencies requires the gsl system library, and some users report GO.db installation issues

## Evidence

- [readme] Before running chromVAR functions, it is advisable to use the `register` function from BiocParallel to specify your preferred method of parallelization.: [evidence span withheld — anchor status: external_doc]
- [intro] Using kmers + PCA appears to be the best variant of chromVAR for clustering, but newer methods such as [SnapATAC] outperform chromVAR for the clustering tasks evaluated: [evidence span withheld — anchor status: not_found]
