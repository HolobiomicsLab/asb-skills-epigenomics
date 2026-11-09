---
name: hmm-based-enriched-region-detection
description: Use when calling protein-DNA binding sites from ChIP-Seq BED or ELAND
  read files (single-end or paired-end), especially when reproducing HPeak peak counts
  or comparing peak callers on datasets such as STAT1, NRSF, H3K4me3, or H3K27me3.
when_to_use_negative:
- Data are comparative histone-modification differences only — the article notes ChIPDiff
  is the method restricted to that comparative setting, not HPeak.
- You need strand/orientation-aware summit refinement — the current HPeak algorithm
  does not distinguish reads of different orientation within a peak.
- Your primary goal is a well-calibrated false detection rate — the paper states HPeak's
  model is not well-suited for quantifying FDR.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: HPeak
  role: HMM-based peak caller that extends reads to HDFs, bins the genome into 25
    bp bins, and classifies bins as enriched or background; implemented in Perl and
    C++
  repo: http://www.sph.umich.edu/csg/qin/HPeak
- name: MatInspector
  role: Motif scanner used downstream to test expected transcription factor binding
    motif enrichment in HPeak peak sequences versus control sequences
- name: Genomatix suite
  role: Suite in which MatInspector motif scans are run with default settings against
    all 153 vertebrate motif matrix families in Genomatix MatBase 7.0
- name: UCSC genome browser
  role: Visualization target for HPeak WIG output peaks
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:8cc0fefb9451cc1d272dbdbde739374785a5a605f1239f16228173fed355daba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - Genomatix suite
  - HPeak
  - MatInspector
  - Perl
  - UCSC genome browser
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# hmm-based-enriched-region-detection

## Summary

Detect ChIP-enriched genomic regions from ChIP-Seq short-read data with HPeak, a two-state Hidden Markov Model that models read-enriched blocks against background using GP and ZIP emission distributions applied to a 25-bp binned HDF coverage profile.

## When to use

Use when calling protein-DNA binding sites from ChIP-Seq BED or ELAND read files (single-end or paired-end), especially when reproducing HPeak peak counts or comparing peak callers on datasets such as STAT1, NRSF, H3K4me3, or H3K27me3.

## When NOT to use

- Data are comparative histone-modification differences only — the article notes ChIPDiff is the method restricted to that comparative setting, not HPeak.
- You need strand/orientation-aware summit refinement — the current HPeak algorithm does not distinguish reads of different orientation within a peak.
- Your primary goal is a well-calibrated false detection rate — the paper states HPeak's model is not well-suited for quantifying FDR.

## Inputs

- ChIP-Seq treatment reads in BED format (-t) or paired-end reads (-pe with -isize)
- Optional ChIP-Seq control reads in BED format (-c)
- Species genome specification (HUMAN or MOUSE) and read length (-r)
- Fragment-length range parameters (-fmin/-fmax for single-end, e.g. 100-300 bp)

## Outputs

- Peak list with summit location and p-value per peak
- WIG format coverage file for genome-browser visualization
- FASTA sequence file of peaks for downstream motif analysis
- Annotation file with enriched gene symbol, RefSeq ID, summit distance
- Intermediate files (-interfiles) supporting enrichment-curve and heatmap analysis

## How to apply

Extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF), following the Robertson et al. approach, and gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF to account for length uncertainty. Partition the genome into fixed-length 25 bp bins and count HDFs per bin to build a coverage profile. Fit and apply a two-state HMM on this profile to distinguish consecutive ChIP-enriched bins from background, using the GP distribution for enriched bins (which fits HDF counts better than Poisson and slightly better than NB) and the ZIP distribution for background bins; estimate parameters with the Viterbi algorithm, initializing with quick summary-statistics estimates. Run with both treatment and control inputs (-t and -c) to obtain the sensitivity gain from control data, or treatment-only for comparison. Emit WIG, FASTA sequence, and annotation output files for downstream visualization, motif analysis (e.g., MatInspector against Genomatix MatBase 7.0), and browser viewing.

## Related tools

- **HPeak** (HMM-based peak caller that extends reads to HDFs, bins the genome into 25 bp bins, and classifies bins as enriched or background; implemented in Perl and C++) — http://www.sph.umich.edu/csg/qin/HPeak
- **MatInspector** (Motif scanner used downstream to test expected transcription factor binding motif enrichment in HPeak peak sequences versus control sequences)
- **Genomatix suite** (Suite in which MatInspector motif scans are run with default settings against all 153 vertebrate motif matrix families in Genomatix MatBase 7.0)
- **UCSC genome browser** (Visualization target for HPeak WIG output peaks)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- Peak counts fall within the ranges the paper reports for reference datasets: STAT1 treated+control 24,394 peaks and treated-only 43,440 peaks; competitor range 9,561 (ChIPseeqer) to 41,127 (FindPeaks).
- Average peak length approximates the reported values: 738 bp on H3K4me3 and 183 bp on H3K27me3.
- Inter-algorithm overlap of H3K4me3 peak sets is >90% for most callers; H3K27me3 overlap varies greatly.
- Motif enrichment of peak sequences over control sequences (MatInspector, Genomatix MatBase 7.0 default settings) is higher when control samples are included in the run, reproducing Figure 2B.
- Output package includes all three expected file types (WIG, FASTA sequence, annotation) per the article's output description.

## Limitations

- The current HPeak algorithm does not distinguish reads of different orientation within a peak, though such information is informative for pinpointing peak summits; orientation support is planned for a future release.
- Little information on true positive and true negative loci exists for experimental ChIP-Seq data, making performance-evaluation criteria and comparisons with caution necessary.
- The method is not well-suited for quantifying false detection rate, which may disadvantage some compared methods.
- Motif patterns stored in databases may be inaccurate and novel motifs may not exist in motif databases, so motif-enrichment validation has caveats.

## Evidence

- [results] Two-state HMM on 25 bp bins: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [results] HMM classification of enriched vs background bins: "HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background"
- [results] HDF extension and end down-weighting: "we gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF"
- [results] Viterbi parameter estimation: "For HMM parameter estimation, we use the well-established Viterbi algorithm [50]. In the initial step, we use summary statistics to get a quick and rough estimate"
- [results] Output file types: "These include a wiggle (WIG) format file for visualization, a sequence file for subsequent motif analysis and an annotation file"
- [readme] HPeak CLI from README: "perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36"
