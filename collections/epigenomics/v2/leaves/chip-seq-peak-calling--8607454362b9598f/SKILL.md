---
name: chip-seq-peak-calling
description: Use when the input is raw ChIP-Seq short reads (BED or ELAND format,
  human or mouse) or BED files for transcription-factor or histone-mark datasets,
  and the goal is to identify read-enriched regions and compare peak callers.
when_to_use_negative:
- The analysis goal is comparative histone modification (differential) data — the
  article notes ChIPDiff is the method restricted to that setting, not HPeak.
- A rigorous false detection rate estimate is required — the authors state 'our method
  is not well-suited for quantifying false detection rate', which may disadvantage
  HPeak in FDR-based comparisons.
- Peak summits require strand/orientation information — the current HPeak algorithm
  does not distinguish reads of different orientation within a peak.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: HPeak
  role: 'HMM-based peak-caller: extends reads into HDFs, bins the genome at 25 bp,
    and applies the two-state HMM with GP/ZIP emissions via the Viterbi algorithm;
    implemented in Perl and C++'
  repo: https://github.com/cauyrd/HPeak
- name: MatInspector
  role: Scans exported peak FASTA sequences with default settings against all 153
    vertebrate motif matrix families in Genomatix MatBase 7.0 to quantify motif enrichment
- name: Genomatix suite
  role: Suite in which MatInspector motif scans of test and control sequences are
    performed
- name: UCSC genome browser
  role: Visualization of called peaks via the WIG output file
- name: PeakFinder / FindPeaks / MACS / SISSRs / CisGenome / ChIPseeqer
  role: Comparison peak-callers run with published/default parameters on the same
    datasets to benchmark HPeak peak counts and overlap
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:2ee4bbeb226e376a5d0e7649f4cda31e922136cfb0a34b522fafa3758ca05724
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cauyrd/HPeak
  tools:
  - C++
  - ChIPseeqer
  - CisGenome
  - FindPeaks
  - Genomatix suite
  - HPeak
  - MACS
  - MatInspector
  - PeakFinder
  - PeakFinder / FindPeaks / MACS / SISSRs / CisGenome / ChIPseeqer
  - Perl
  - SISSRs
  - UCSC genome browser
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# chip-seq-peak-calling

## Summary

Calls ChIP-enriched genomic regions from short-read ChIP-Seq data using HPeak, a two-state HMM that partitions the genome into 25 bp bins, extends reads directionally into hypothetical DNA fragments (HDFs), and models enriched-bin counts with a generalized Poisson (GP) emission and background counts with a zero-inflated Poisson (ZIP) emission.

## When to use

Use when the input is raw ChIP-Seq short reads (BED or ELAND format, human or mouse) or BED files for transcription-factor or histone-mark datasets, and the goal is to identify read-enriched regions and compare peak callers.

## When NOT to use

- The analysis goal is comparative histone modification (differential) data — the article notes ChIPDiff is the method restricted to that setting, not HPeak.
- A rigorous false detection rate estimate is required — the authors state 'our method is not well-suited for quantifying false detection rate', which may disadvantage HPeak in FDR-based comparisons.
- Peak summits require strand/orientation information — the current HPeak algorithm does not distinguish reads of different orientation within a peak.

## Inputs

- Treatment ChIP-Seq short reads in BED or ELAND format (single-end, -t)
- Paired-end treatment reads (-pe) with insert size (-isize) and optional paired-end control (-pec)
- Control (unstimulated/mock) ChIP-Seq sample in the same format (-c)
- Read length and species (HUMAN/MOUSE) parameters
- Histone mark BED files (e.g., H3K4me3, H3K27me3) or GEO datasets (e.g., NRSF GSE13047, STAT1)

## Outputs

- Peak list with summit location and p-value per called region
- WIG-format coverage file for genome-browser visualization
- FASTA sequence file of peak regions for downstream motif analysis
- Peak annotation file (enriched gene symbol, RefSeq ID, summit distance, conservation score)
- Intermediate files for downstream analyses such as enrichment curves and heatmaps

## How to apply

Extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF), gradually down-weighting coverage contributed by the end portion of each HDF to account for fragment-length uncertainty. Partition the genome into 25 bp bins and count HDFs per bin, then apply a two-state HMM (Viterbi algorithm, initial estimates from summary statistics) with GP emissions for ChIP-enriched bins and ZIP emissions for background bins to classify consecutive enriched blocks as peaks. Run HPeak in treated-only mode and in treated+control mode (-c flag) — including the control sample refines peak calling and improves motif enrichment (e.g., STAT1: 43,440 peaks treated-only vs 24,394 with control). Request annotation (-ann), WIG visualization (-wig), FASTA sequence output (-seq), and intermediate files (-interfiles) so peak sequences can be scanned with MatInspector against the 153 vertebrate motif matrix families in Genomatix MatBase 7.0 to validate enrichment of the expected transcription-factor motif. Specify read length explicitly (-r) as the README notes this is important from practical experience.

## Related tools

- **HPeak** (HMM-based peak-caller: extends reads into HDFs, bins the genome at 25 bp, and applies the two-state HMM with GP/ZIP emissions via the Viterbi algorithm; implemented in Perl and C++) — https://github.com/cauyrd/HPeak
- **MatInspector** (Scans exported peak FASTA sequences with default settings against all 153 vertebrate motif matrix families in Genomatix MatBase 7.0 to quantify motif enrichment)
- **Genomatix suite** (Suite in which MatInspector motif scans of test and control sequences are performed)
- **UCSC genome browser** (Visualization of called peaks via the WIG output file)
- **PeakFinder / FindPeaks / MACS / SISSRs / CisGenome / ChIPseeqer** (Comparison peak-callers run with published/default parameters on the same datasets to benchmark HPeak peak counts and overlap)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- Peak counts fall within the reported competitor range on benchmark data — e.g., NRSF: 1,935 peaks (PeakFinder) to 5,243 peaks (SISSRs); STAT1: 9,561 (ChIPseeqer) to 41,127 (FindPeaks).
- Reproduced HPeak counts match reported values: 43,440 peaks treated-only and 24,394 peaks treated+control on STAT1.
- MatInspector scan shows the expected transcription factor motif (e.g., NRSF or STAT family) is the most enriched in HPeak peaks relative to length-matched random controls, and enrichment is higher in treated+control runs than treated-only runs (Figure 2B pattern).
- Pairwise inter-algorithm peak overlap exceeds 90% for H3K4me3, with average peak lengths near 738 bp (H3K4me3) and 183 bp (H3K27me3), and highly variable overlap for H3K27me3.
- Output files exist as expected: -ann produces the annotation file, -wig the WIG file, -seq the FASTA sequence file, and -interfiles the intermediate files.

## Limitations

- The method is not well-suited for quantifying false detection rate, which may put some compared methods at a disadvantage; performance-evaluation results should be interpreted with caution because little true positive/negative information is available for experimental ChIP-Seq data.
- The current HPeak algorithm does not distinguish reads of different orientation within a peak, though such information is informative for pinpointing peak summits (planned for a future release).
- Motif-based validation depends on motif databases: stored patterns may be inaccurate, novel motifs may be absent, and the effect of inaccurate patterns on comparison results remains to be seen.
- Read length must be supplied explicitly (-r); the README notes this is important from experience, and omitting it risks incorrect HDF extension.

## Evidence

- [discussion] HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background: "HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background"
- [discussion] Motif scan of the test and the control sequences was then performed using MatInspector [52] in the Genomatix suite: "Motif scan of the test and the control sequences was then performed using MatInspector [52] in the Genomatix suite"
- [readme] perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles: "perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles"
- [readme] –r specifies read length (this is import through my experience).: "–r specifies read length (this is import through my experience)."
