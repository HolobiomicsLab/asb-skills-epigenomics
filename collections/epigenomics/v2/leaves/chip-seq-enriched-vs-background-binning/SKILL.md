---
name: chip-seq-enriched-vs-background-binning
description: Use when ChIP-Seq short-read data (e.g. BED/ELAND single-end or pair-end)
  must be binned into enriched vs. background regions to model read-count distributions
  or call protein-DNA binding peaks.
when_to_use_negative:
- Data are comparative histone modification ChIP-Seq (ChIPDiff's scope) rather than
  transcription-factor ChIP-Seq peak calling
- Reads of different orientation within a peak must be distinguished to pinpoint the
  summit — the current HPeak algorithm does not use orientation information
- Input already consists of classified enriched/background regions or a feature table,
  not raw aligned reads
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: HPeak
  role: HMM-based peak-finder that bins the genome into 25 bp intervals, counts HDFs
    per bin, and classifies bins as enriched or background via a two-state HMM with
    GP/ZIP emissions
  repo: https://github.com/cauyrd/HPeak
- name: Perl
  role: Implementation language of the HPeak software package driver (HPeak.pl)
- name: C++
  role: Implementation language of the HPeak software package core routines
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:65fd14184a58ff861a0a36ee0f4b81ad329cef546531f694bfcf81ae9abd7257
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cauyrd/HPeak
  tools:
  - C++
  - HPeak
  - Perl
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# chip-seq-enriched-vs-background-binning

## Summary

Partitions a ChIP-Seq genome into fixed 25 bp bins, counts hypothetical DNA fragments (HDFs) per bin, and uses HPeak's two-state HMM with GP and ZIP emissions to classify bins as ChIP-enriched or background for downstream distribution fitting and peak calling.

## When to use

Use when ChIP-Seq short-read data (e.g. BED/ELAND single-end or pair-end) must be binned into enriched vs. background regions to model read-count distributions or call protein-DNA binding peaks.

## When NOT to use

- Data are comparative histone modification ChIP-Seq (ChIPDiff's scope) rather than transcription-factor ChIP-Seq peak calling
- Reads of different orientation within a peak must be distinguished to pinpoint the summit — the current HPeak algorithm does not use orientation information
- Input already consists of classified enriched/background regions or a feature table, not raw aligned reads

## Inputs

- ChIP-Seq treatment read file (BED or ELAND format, single-end with read length, e.g. -r 36, or pair-end with insert size, e.g. -isize 200)
- Optional control ChIP-Seq read file in the same formats
- Genome specification (HUMAN or MOUSE; other genomes can be added)
- Fragment length range for single-end extension (e.g. -fmin 100 -fmax 300)

## Outputs

- Binary classification of genome bins into ChIP-enriched vs. background HDF coverage blocks
- Peak list with summit location and p-value
- WIG format coverage file for genome browser visualization
- FASTA sequence file of enriched regions for downstream motif analysis
- Annotation file with enriched gene symbols, refseq IDs, summit distance, conservation score

## How to apply

Extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF), following the Robertson et al. method, and gradually down-weight HDF coverage in bins covered by the end portion of each fragment to account for length uncertainty. Partition the entire genome into 25 bp bins and count the HDFs falling in each bin to build an HDF coverage profile. Apply HPeak's two-state HMM, using GP emissions for enriched bins and ZIP emissions for background, estimating parameters with the Viterbi algorithm seeded by summary statistics, to classify consecutive blocks of bins as ChIP-enriched or background. The rationale is to explicitly model noise in the sequencing data so enriched regions are defined by rigorous statistical inference rather than ad-hoc thresholds; an unbalanced weighting scheme handles uncertainty in actual fragment lengths. The choice of GP vs. ZIP emissions is grounded in the authors' distribution fitting: GP best fits enriched-bin HDF counts and ZIP best fits background-bin counts.

## Related tools

- **HPeak** (HMM-based peak-finder that bins the genome into 25 bp intervals, counts HDFs per bin, and classifies bins as enriched or background via a two-state HMM with GP/ZIP emissions) — https://github.com/cauyrd/HPeak
- **Perl** (Implementation language of the HPeak software package driver (HPeak.pl))
- **C++** (Implementation language of the HPeak software package core routines)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- Enriched sequences identified by the HMM show higher prevalence of the expected transcription factor binding motifs (e.g. via MatInspector against Genomatix MatBase 7.0 matrices) than control sequences and than peaks from competing algorithms
- On the NRSF ChIP-Seq benchmark (Laajala et al.), the binning/peak classification achieves the best true positive rate, closest to the optimum
- HDF-count distribution fits match expectation: GP best fits enriched-bin counts (much better than Poisson, slightly better than NB), and ZIP slightly better fits background-bin counts than Poisson and NB
- Output package contains all three expected artifacts: WIG file, FASTA sequence file, and annotation file
- For pair-end input, both ends are treated together as ONE fragment (insert size respected in bin counting)

## Limitations

- Little true positive/negative ground truth exists for experimental ChIP-Seq, so performance evaluation criteria are challenging and results should be interpreted with caution
- The method is not well-suited to quantifying false detection rate, which may disadvantage some compared methods
- The current HPeak algorithm does not distinguish reads of different orientation within a peak, limiting summit pinpointing; orientation support is planned for a future release
- The background-region ZIP fit result was reported as 'data not shown', so the ZIP-vs-Poisson/NB ranking for background bins is not fully documented in the article

## Evidence

- [discussion] HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] Two-state HMM classification of bins: "HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background"
- [discussion] Down-weighting of fragment-end coverage: "we gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF"
- [discussion] GP fits enriched counts best: "the GP distribution shows a much better fit than the Poisson distribution and a slightly better fit than the NB distributions"
- [readme] README command showing binning/peak-calling invocation: "perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles"
