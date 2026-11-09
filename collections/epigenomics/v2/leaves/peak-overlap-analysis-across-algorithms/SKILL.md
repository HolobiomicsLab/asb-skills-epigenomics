---
name: peak-overlap-analysis-across-algorithms
description: Use when you need to evaluate agreement between different ChIP-Seq peak-calling
  algorithms on the same dataset, e.g. comparing called peak regions for histone marks
  like H3K4me3 or H3K27me3 by pairwise overlap percentages.
when_to_use_negative:
- The question is about a single algorithm's accuracy against known true positives
  (motif enrichment or TP rate on data like NRSF), not cross-algorithm consistency.
- Input is already peak-level data with no common raw BED input, so callers cannot
  be re-run on identical reads for a fair comparison.
- Data are transcription-factor ChIP-Seq with sharp punctate peaks where orientation/summit
  information matters; the HPeak workflow here does not distinguish reads of different
  orientation within a peak.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: HPeak
  role: Primary HMM-based peak caller applied to each BED file; extends short reads
    directionally into HDFs, uses 25-bp bins and a two-state HMM with GP/ZIP emissions
    to define ChIP-enriched regions
  repo: https://github.com/cauyrd/HPeak
- name: Perl
  role: Scripting language in which HPeak is implemented and driven via HPeak.pl
- name: C++
  role: Implementation language for the HPeak software package alongside Perl
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:de7114facdc297292392ab2c9566419df34822d4339c1a957e9d71fa9b1ded2e
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
yes_rate: 0.6666666666666666
n_claims: 12
---

# peak-overlap-analysis-across-algorithms

## Summary

A comparative analysis skill that runs multiple ChIP-Seq peak-calling algorithms (HPeak plus alternatives) on the same input BED files and quantifies pairwise inter-algorithm peak overlap percentages and average peak lengths, to assess whether peak callers yield consistent peak sets.

## When to use

Use when you need to evaluate agreement between different ChIP-Seq peak-calling algorithms on the same dataset, e.g. comparing called peak regions for histone marks like H3K4me3 or H3K27me3 by pairwise overlap percentages.

## When NOT to use

- The question is about a single algorithm's accuracy against known true positives (motif enrichment or TP rate on data like NRSF), not cross-algorithm consistency.
- Input is already peak-level data with no common raw BED input, so callers cannot be re-run on identical reads for a fair comparison.
- Data are transcription-factor ChIP-Seq with sharp punctate peaks where orientation/summit information matters; the HPeak workflow here does not distinguish reads of different orientation within a peak.

## Inputs

- ChIP-Seq BED files for treatment and control samples (e.g. H3K4me3 and H3K27me3)
- Peak sets called by each peak-calling algorithm (genomic interval lists)
- Read length specification for peak calling (e.g. -r 36)

## Outputs

- Pairwise inter-algorithm peak overlap percentage table per histone mark
- Average peak length per algorithm per dataset (e.g. 738 bp H3K4me3, 183 bp H3K27me3)
- CSV summary file, e.g. peak_overlap_and_lengths.csv

## How to apply

Download the ChIP-Seq BED files (e.g. H3K4me3 and H3K27me3 from the H1 cell epigenome resource) and run HPeak on each, extending reads directionally into hypothetical DNA fragments (HDFs), binning the genome into 25-bp bins, counting HDFs per bin, and applying the two-state HMM with GP/ZIP emissions. Run each comparison algorithm on the identical BED files to obtain its peak set. For each histone mark, compute pairwise inter-algorithm overlap percentages between the called peak regions and the average peak length per algorithm. Interpret consistency by mark type: H3K4me3 peaks mostly show >90% overlap between algorithms, whereas H3K27me3 overlaps vary greatly because callers produce very different peak counts (average lengths 738 bp for H3K4me3 vs 183 bp for H3K27me3 in the reference study). Write the overlap tables and average lengths to a CSV such as peak_overlap_and_lengths.csv.

## Related tools

- **HPeak** (Primary HMM-based peak caller applied to each BED file; extends short reads directionally into HDFs, uses 25-bp bins and a two-state HMM with GP/ZIP emissions to define ChIP-enriched regions) — https://github.com/cauyrd/HPeak
- **Perl** (Scripting language in which HPeak is implemented and driven via HPeak.pl)
- **C++** (Implementation language for the HPeak software package alongside Perl)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t H3K4me3.bed -c control.bed -n h3k4me3_hpeak -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- H3K4me3 pairwise overlap percentages between algorithms should be mostly >90%; values far below that indicate a pipeline or file mismatch.
- Average peak lengths should fall in the ranges reported (738 bp for H3K4me3, 183 bp for H3K27me3); large deviations suggest wrong fragment-length (-fmin/-fmax) or binning settings.
- All algorithms must be run on the identical input BED files; overlap computed from different inputs is invalid.
- H3K27me3 overlap values are expected to vary greatly across algorithm pairs because callers produce very different peak numbers — uniform agreement would be suspicious.
- Output CSV must contain both overlap tables and per-algorithm average peak lengths, matching peak_overlap_and_lengths.csv schema.

## Limitations

- Little information on true positive and true negative loci makes criteria for evaluating peak-calling algorithms on experimental ChIP-Seq data challenging to determine; performance results should be interpreted with caution.
- The method is not well-suited for quantifying false detection rate, which may disadvantage some compared methods.
- The current HPeak algorithm does not distinguish reads of different orientation within a peak, which could affect peak boundaries used in overlap comparisons.
- Overlap statistics are dataset-dependent: broad marks like H3K27me3 yield highly variable overlap because algorithms generate quite different numbers of peaks.

## Evidence

- [other] Results from most of the peak-calling algorithms tested are very similar for H3K4me3 (most overlap percentages are greater than 90%): [evidence span withheld — anchor status: not_found]
- [discussion] HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] we followed the method used by Robertson et al. to extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF): "we followed the method used by Robertson et al. [11] to extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF)"
- [discussion] We have implemented HPeak in a software package using Perl and C++.: "We have implemented HPeak in a software package using Perl and C++."
- [readme] perl HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles: "perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles"
- [discussion] The current HPeak algorithm does not distinguish reads of different orientation within a peak.: "The current HPeak algorithm does not distinguish reads of different orientation within a peak."
