---
name: hmm-coverage-profile-modeling
description: Use when calling protein-DNA binding peaks from short-read ChIP-Seq data
  with explicit statistical modeling of sequencing noise, or when reproducing HPeak
  peak counts and motif-enrichment comparisons against other peak-callers on datasets
  like the NRSF ChIP-Seq data.
when_to_use_negative:
- Analyzing comparative histone modification ChIP data — the article notes ChIPDiff
  is designed specifically for that comparative setting, not standard binding-site
  peak calling.
- Quantifying false detection rate is a primary goal — the article states 'our method
  is not well-suited for quantifying false detection rate', so comparisons may disadvantage
  some methods.
- The input is already a peak list or motif enrichment table rather than raw short-read
  alignments.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: HPeak
  role: 'Core HMM-based peak-caller: extends reads to HDFs, bins coverage at 25 bp,
    and applies the two-state GP/ZIP HMM to call enriched regions'
  repo: github.com/cauyrd/HPeak
- name: MatInspector
  role: Motif scanning of exported peak FASTA sequences against Genomatix MatBase
    7.0 vertebrate motif matrix families to assess enrichment
- name: PeakFinder
  role: Comparison peak-caller used to define HPeak-unique peaks (e.g., the 2,323
    peaks significant by HPeak but not PeakFinder on NRSF data)
- name: MACS
  role: Comparison peak-caller run with published/default settings on the same NRSF
    data for peak-count benchmarking
- name: FindPeaks
  role: Comparison peak-caller whose unique peaks serve as the contrast group in NRSF
    motif-enrichment Chi-square tests
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:88bbcf9e43eb9c714bed139fe4c3ac145232124d92c68df911241fc963e3582f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChIPseeqer
  - CisGenome
  - FindPeaks
  - Genomatix suite
  - HPeak
  - MACS
  - MatInspector
  - PeakFinder
  - SISSRs
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# hmm-coverage-profile-modeling

## Summary

Model ChIP-Seq read coverage as a two-state Hidden Markov Model over fixed-length genomic bins to statistically distinguish ChIP-enriched regions from background noise, using a Generalized Poisson emission for enriched bins and a Zero-Inflated Poisson emission for background bins (as implemented in HPeak).

## When to use

Use when calling protein-DNA binding peaks from short-read ChIP-Seq data with explicit statistical modeling of sequencing noise, or when reproducing HPeak peak counts and motif-enrichment comparisons against other peak-callers on datasets like the NRSF ChIP-Seq data.

## When NOT to use

- Analyzing comparative histone modification ChIP data — the article notes ChIPDiff is designed specifically for that comparative setting, not standard binding-site peak calling.
- Quantifying false detection rate is a primary goal — the article states 'our method is not well-suited for quantifying false detection rate', so comparisons may disadvantage some methods.
- The input is already a peak list or motif enrichment table rather than raw short-read alignments.

## Inputs

- ChIP-enriched (treated) short-read sequencing data in BED or ELAND format
- Optional matched control sequencing sample for treated+control peak-calling mode
- Read length (e.g., -r 36) and expected fragment size range (e.g., -fmin 100 -fmax 300 for single-end; -isize 200 for pair-end)
- Reference genome specification (HUMAN or MOUSE)

## Outputs

- Set of significant read-enriched regions (peaks) with summit location and p-value
- WIG (wiggle) format coverage file for UCSC genome browser visualization
- FASTA sequence file of peak regions for downstream motif analysis
- Annotation file with enriched gene symbols, RefSeq IDs, summit distances, and conservation scores

## How to apply

First extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF), following the Robertson et al. approach. Partition the genome into fixed-length 25 bp bins and count HDFs per bin, gradually down-weighting the HDF coverage contribution at fragment ends to account for uncertainty in actual DNA fragment lengths. Fit a two-state HMM on this HDF coverage profile, using a Generalized Poisson (GP) emission for ChIP-enriched bins (which fits better than Poisson and slightly better than Negative Binomial) and a Zero-Inflated Poisson (ZIP) emission for background bins (which fits slightly better than Poisson and NB); the treated-only mode captures enrichment while a treated+control mode uses the control sample to refine peak calling. Estimate HMM parameters with the Viterbi algorithm, seeding initial estimates from summary statistics, then classify consecutive enriched bins as peaks and export WIG, FASTA sequence, and annotation files. Validate peak counts against the expected range (e.g., 1,935 peaks from PeakFinder to 5,243 from SISSRs on NRSF data) and confirm the expected transcription factor motif (e.g., NRSF motif via MatInspector against Genomatix MatBase 7.0 matrices) is enriched in called peaks relative to length-matched random controls.

## Related tools

- **HPeak** (Core HMM-based peak-caller: extends reads to HDFs, bins coverage at 25 bp, and applies the two-state GP/ZIP HMM to call enriched regions) — github.com/cauyrd/HPeak
- **MatInspector** (Motif scanning of exported peak FASTA sequences against Genomatix MatBase 7.0 vertebrate motif matrix families to assess enrichment)
- **PeakFinder** (Comparison peak-caller used to define HPeak-unique peaks (e.g., the 2,323 peaks significant by HPeak but not PeakFinder on NRSF data))
- **MACS** (Comparison peak-caller run with published/default settings on the same NRSF data for peak-count benchmarking)
- **FindPeaks** (Comparison peak-caller whose unique peaks serve as the contrast group in NRSF motif-enrichment Chi-square tests)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t treated.bed -c control.bed -n nrsf_hpeak -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- Reproduced peak counts fall within the reported range of 1,935 peaks (PeakFinder) to 5,243 peaks (SISSRs) on the NRSF dataset (GSE13047).
- GP emission fits HDF counts in ChIP-enriched regions better than Poisson and slightly better than NB; ZIP fits background counts slightly better than Poisson and NB.
- The expected transcription factor motif (NRSF motif) is the most enriched in called peaks relative to length-matched random control sequences when scanned with MatInspector against all 153 vertebrate motif matrix families of Genomatix MatBase 7.0.
- Chi-square statistics comparing NRSF motif enrichment in HPeak-unique versus FindPeaks-unique peaks match the values reported in the paper's comparison tables.
- Output artifacts include a valid WIG file visualizable in the UCSC genome browser and a FASTA sequence file suitable for motif analysis.

## Limitations

- The current HPeak algorithm does not distinguish reads of different orientation within a peak, though such strand information has been shown to be informative in pinpointing the peak summit.
- Very little information on true positive and true negative loci is available for experimental ChIP-Seq data, making performance evaluation criteria challenging; results should be interpreted with caution.
- Motif patterns stored in databases may be inaccurate, and novel motifs may not exist in motif databases, which can affect motif-enrichment-based validation.
- The method is not well-suited for quantifying false detection rate, potentially disadvantaging some compared methods.

## Evidence

- [discussion] HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] GP vs Poisson vs NB fit in enriched regions: "the GP distribution shows a much better fit than the Poisson distribution and a slightly better fit than the NB distributions"
- [discussion] ZIP background fit: "the ZIP distribution produces a slightly better fit than both Poisson and NB distributions"
- [discussion] HMM parameter estimation: "For HMM parameter estimation, we use the well-established Viterbi algorithm [50]. In the initial step, we use summary statistics to get a quick and rough estimate"
- [readme] README invocation with key parameters: "perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles"
- [discussion] Output artifacts for downstream analysis: "These include a wiggle (WIG) format file for visualization, a sequence file for subsequent motif analysis and an annotation file"
- [discussion] Strand-orientation limitation: "The current HPeak algorithm does not distinguish reads of different orientation within a peak. Such information has been shown to be informative in pinpointing the summit of the peak"
