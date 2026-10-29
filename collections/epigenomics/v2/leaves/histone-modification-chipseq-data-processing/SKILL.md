---
name: histone-modification-chipseq-data-processing
description: Use when processing ChIP-Seq BED files of histone modifications (e.g.,
  H3K4me3, H3K27me3) to call enriched regions with HPeak and to assess whether different
  peak-calling algorithms produce overlapping peak sets.
when_to_use_negative:
- Input is already a processed peak set or feature table and the goal is downstream
  functional annotation rather than peak calling
- The data are comparative histone modification differential-analysis tasks — the
  article notes the ChIPDiff method is 'restricted to analyzing comparative histone
  modification data', a different problem
- 'Analysis requires distinguishing reads of different orientation within a peak:
  the current HPeak algorithm does not do this'
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: HPeak
  role: 'HMM-based peak-calling algorithm applied to histone mark BED files: HDF extension,
    25-bp binning, two-state HMM with GP/ZIP emissions'
  repo: https://github.com/cauyrd/HPeak
- name: Perl
  role: HPeak is implemented as a Perl software package; the HPeak.pl wrapper drives
    the pipeline
- name: C++
  role: HPeak implementation language alongside Perl for the peak-finding package
- name: UCSC genome browser
  role: Visualization of HPeak-called peaks via the emitted WIG format file
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:022a5484410bd6e3b9c0377258761bc74baaa44e8e03d4825806ccc29a84ba2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cauyrd/HPeak
  tools:
  - C++
  - HPeak
  - Perl
  - UCSC genome browser
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# histone-modification-chipseq-data-processing

## Summary

A ChIP-Seq peak-calling and cross-algorithm comparison workflow for histone mark data (e.g., H3K4me3 and H3K27me3): call read-enriched regions with the HMM-based HPeak, then quantify inter-algorithm peak overlap and average peak lengths. It exists because peak-caller agreement differs sharply between sharp marks like H3K4me3 and broad marks like H3K27me3.

## When to use

Use when processing ChIP-Seq BED files of histone modifications (e.g., H3K4me3, H3K27me3) to call enriched regions with HPeak and to assess whether different peak-calling algorithms produce overlapping peak sets.

## When NOT to use

- Input is already a processed peak set or feature table and the goal is downstream functional annotation rather than peak calling
- The data are comparative histone modification differential-analysis tasks — the article notes the ChIPDiff method is 'restricted to analyzing comparative histone modification data', a different problem
- Analysis requires distinguishing reads of different orientation within a peak: the current HPeak algorithm does not do this

## Inputs

- ChIP-Seq BED files for histone marks (treatment and control), single-end or pair-end
- Read length specification (e.g., -r 36) and insert size for pair-end data (e.g., -isize 200)

## Outputs

- Peak sets (peak regions with summit location and p-value) per algorithm per histone mark
- HPeak output files: WIG track for UCSC genome browser visualization, FASTA sequence file for motif analysis, and annotation file
- peak_overlap_and_lengths.csv containing pairwise overlap percentages and average peak lengths (738 bp H3K4me3, 183 bp H3K27me3)

## How to apply

Download H3K4me3 and H3K27me3 ChIP-Seq BED files (e.g., from http://dir.nhlbi.nih.gov/papers/lmi/epigenomes/hgtcell.aspx), then run HPeak via its Perl wrapper: extend each short read directionally from its start position into a hypothetical DNA fragment (HDF), partition the genome into fixed 25-bp bins, count HDFs per bin, and apply the two-state HMM with generalized Poisson (GP) emissions for enriched bins and zero-inflated Poisson (ZIP) emissions for background to classify bins. Use a novel unbalanced weighting scheme to gradually down-weight HDF coverage at fragment ends to account for fragment-length uncertainty, and estimate HMM parameters with the Viterbi algorithm seeded from summary statistics. Run the comparison peak-calling algorithms on the same BED files, then compute pairwise inter-algorithm overlap percentages of called peak regions per mark and the average peak length per algorithm. Correct application is judged by H3K4me3 overlap percentages mostly above 90% while H3K27me3 overlaps vary greatly due to differing numbers of called peaks, and by reported average peak lengths near 738 bp (H3K4me3) and 183 bp (H3K27me3), written to peak_overlap_and_lengths.csv.

## Related tools

- **HPeak** (HMM-based peak-calling algorithm applied to histone mark BED files: HDF extension, 25-bp binning, two-state HMM with GP/ZIP emissions) — https://github.com/cauyrd/HPeak
- **Perl** (HPeak is implemented as a Perl software package; the HPeak.pl wrapper drives the pipeline)
- **C++** (HPeak implementation language alongside Perl for the peak-finding package)
- **UCSC genome browser** (Visualization of HPeak-called peaks via the emitted WIG format file)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t H3K4me3.bed -c INPUT.bed -n H3K4me3_peaks -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- H3K4me3 pairwise inter-algorithm overlap percentages are mostly >90%
- H3K27me3 overlap percentages vary greatly because algorithms generate quite different numbers of peaks
- Average peak lengths match the expected values: 738 bp for H3K4me3 and 183 bp for H3K27me3 (peak_overlap_and_lengths.csv)
- HPeak emits the expected outputs: WIG visualization file, FASTA sequence file, and annotation file
- On benchmark data (e.g., NRSF ChIP-Seq), HPeak should show the best true positive rate among compared algorithms

## Limitations

- Little information on true positive and true negative loci makes criteria for evaluating peak-calling algorithms on experimental ChIP-Seq data challenging to determine, so performance results should be interpreted with caution
- The current HPeak algorithm does not distinguish reads of different orientation within a peak, which could otherwise inform peak summit placement (planned for future release)
- The method is not well-suited for quantifying false detection rate, which may disadvantage some compared methods in the overlap comparison

## Evidence

- [methods] Extend each short read directionally to form a hypothetical DNA fragment (HDF): "we followed the method used by Robertson et al. [11] to extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF)"
- [methods] Two-state HMM with GP/ZIP emissions: "HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background"
- [methods] 25-bp genome binning: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [methods] HPeak outputs WIG, FASTA, annotation files: "These include a wiggle (WIG) format file for visualization, a sequence file for subsequent motif analysis and an annotation file"
- [discussion] HPeak implementation in Perl and C++: "We have implemented HPeak in a software package using Perl and C++."
- [discussion] Peak-calling evaluation limitation: "it is challenging to determine criteria to evaluate the performance of various peak-calling algorithms on experimental ChIP-Seq data"
- [discussion] No read-orientation modeling: "The current HPeak algorithm does not distinguish reads of different orientation within a peak."
- [readme] HPeak command-line invocation: "perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles"
