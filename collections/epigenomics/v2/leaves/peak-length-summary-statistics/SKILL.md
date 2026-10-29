---
name: peak-length-summary-statistics
description: Use when comparing peak sets from multiple ChIP-seq peak-calling algorithms
  on histone mark datasets (H3K4me3, H3K27me3) and you need overlap percentages and
  average peak-length summaries.
when_to_use_negative:
- The analysis targets transcription-factor ChIP-seq requiring motif-enrichment validation
  rather than histone-mark peak-set comparison
- Only one peak caller's output is available, so no pairwise inter-algorithm overlap
  statistics can be computed
- Input is raw reads needing alignment/quality filtering rather than already-called
  peak interval sets to summarize
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: HPeak
  role: HMM-based peak caller applied to H3K4me3 and H3K27me3 BED files (directional
    HDF extension, 25-bp binning, two-state HMM with GP/ZIP emissions) to generate
    one peak set for the overlap and length statistics
  repo: https://github.com/cauyrd/HPeak
- name: Perl
  role: Part of the HPeak software package used to run the peak-calling step feeding
    the summary statistics
- name: C++
  role: Implementation language (with Perl) of the HPeak package used for peak calling
    prior to computing overlap/length summaries
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:25ac53bb0e7d672759a7bfd9803cceb98ede5463b99a3a4dac10e9b380c4d205
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
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# peak-length-summary-statistics

## Summary

Reproduce the HPeak paper's histone-mark benchmark statistics: pairwise inter-algorithm ChIP-seq peak overlap percentages and average peak lengths (e.g., 738 bp for H3K4me3, 183 bp for H3K27me3) to test consistency of peak callers.

## When to use

Use when comparing peak sets from multiple ChIP-seq peak-calling algorithms on histone mark datasets (H3K4me3, H3K27me3) and you need overlap percentages and average peak-length summaries.

## When NOT to use

- The analysis targets transcription-factor ChIP-seq requiring motif-enrichment validation rather than histone-mark peak-set comparison
- Only one peak caller's output is available, so no pairwise inter-algorithm overlap statistics can be computed
- Input is raw reads needing alignment/quality filtering rather than already-called peak interval sets to summarize

## Inputs

- H3K4me3 ChIP-Seq BED file of short sequencing reads
- H3K27me3 ChIP-Seq BED file of short sequencing reads
- Peak sets (genomic intervals) produced by HPeak and comparison peak-calling algorithms on the same BED inputs

## Outputs

- Pairwise inter-algorithm peak overlap percentage table per histone mark (peak_overlap_and_lengths.csv)
- Average peak length per algorithm per dataset (e.g., 738 bp H3K4me3, 183 bp H3K27me3)

## How to apply

Download the H3K4me3 and H3K27me3 ChIP-Seq BED files (human CD4+ T cells, dir.nhlbi.nih.gov/papers/lmi/epigenomes/hgtcell.aspx). Run HPeak on each BED file: extend each short read directionally into hypothetical DNA fragments (HDFs), partition the genome into fixed 25-bp bins, count HDFs per bin, and apply the two-state HMM with GP/ZIP emissions to call ChIP-enriched regions; run the comparison peak callers on the same BED inputs. For each histone mark, compute pairwise inter-algorithm overlap percentages between the called peak regions and the average peak length per algorithm per dataset. Expect mostly >90% overlap for H3K4me3 and highly variable overlap for H3K27me3 (because algorithms generate quite different numbers of peaks), with average peak lengths around 738 bp (H3K4me3) and 183 bp (H3K27me3). Write the overlap tables and average lengths to peak_overlap_and_lengths.csv.

## Related tools

- **HPeak** (HMM-based peak caller applied to H3K4me3 and H3K27me3 BED files (directional HDF extension, 25-bp binning, two-state HMM with GP/ZIP emissions) to generate one peak set for the overlap and length statistics) — https://github.com/cauyrd/HPeak
- **Perl** (Part of the HPeak software package used to run the peak-calling step feeding the summary statistics)
- **C++** (Implementation language (with Perl) of the HPeak package used for peak calling prior to computing overlap/length summaries)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t H3K4me3.bed -n H3K4me3_hpeak -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- Average H3K4me3 peak length across algorithms is on the order of 738 bp; H3K27me3 around 183 bp
- Pairwise H3K4me3 overlap percentages between most peak callers exceed 90%
- H3K27me3 overlap percentages vary greatly across algorithm pairs, reflecting algorithm-dependent peak counts
- peak_overlap_and_lengths.csv contains both the overlap table and the per-algorithm average peak lengths
- All peak sets were generated from the same source BED files (same download, same HPeak parameters: 25-bp bins, HDF extension)

## Limitations

- H3K27me3 overlap is highly variable because the algorithms generate quite different numbers of peaks, making inter-algorithm agreement hard to interpret
- Little information on true positive and true negative loci makes criteria for evaluating peak-calling algorithms on experimental ChIP-Seq data challenging to determine; results should be interpreted with caution
- The HPeak method is not well-suited for quantifying false detection rate, which may disadvantage some compared methods in the overlap comparison

## Evidence

- [discussion] HMM peak calling on binned HDF coverage: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] two-state HMM classification of bins: "HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background"
- [discussion] directional read extension into HDFs: "we followed the method used by Robertson et al. [11] to extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF)"
- [discussion] HPeak implemented in Perl and C++: "We have implemented HPeak in a software package using Perl and C++."
- [readme] HPeak CLI invocation from README: "perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles"
