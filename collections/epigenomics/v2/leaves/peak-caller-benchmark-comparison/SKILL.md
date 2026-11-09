---
name: peak-caller-benchmark-comparison
description: Use when benchmarking ChIP-Seq peak callers on one shared treated+control
  dataset (e.g., NRSF GSE13047 or STAT1) to compare peak counts and TF-motif enrichment
  across HPeak, PeakFinder, FindPeaks, MACS, SISSRs, CisGenome, and ChIPseeqer.
when_to_use_negative:
- Data is not ChIP-Seq read data (e.g., already a peak BED file or feature count matrix)
  — this skill requires raw treated/control reads to run all callers on identical
  input.
- Comparative differential-binding analysis between two conditions (e.g., histone
  modification change detection as in ChIPDiff) rather than within-condition peak-caller
  benchmarking.
- Only one peak caller is of interest and no cross-algorithm comparison or motif-enrichment
  evaluation is needed — running a single caller suffices.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: HPeak
  role: HMM-based peak caller under evaluation; run in treated-only and treated+control
    modes with 25 bp bins, HDF extension, GP/ZIP emissions, and Viterbi decoding
  repo: github.com/cauyrd/HPeak
- name: PeakFinder
  role: Comparison peak caller run with published/default settings on the same NRSF
    data (reported 1,935 peaks)
- name: FindPeaks
  role: Comparison peak caller; its unique NRSF peaks form the Chi-square contrast
    group against HPeak-unique peaks (41,127 peaks on STAT1)
- name: MACS
  role: Comparison peak caller run with default parameters on the shared dataset
- name: SISSRs
  role: Comparison peak caller; upper bound of NRSF peak-count range (5,243 peaks)
- name: CisGenome
  role: Comparison peak caller run with published/default parameter settings
- name: ChIPseeqer
  role: Comparison peak caller; lower bound of STAT1 peak-count range (9,561 peaks)
- name: MatInspector
  role: Scans each caller's peak FASTA sequences for expected TF motifs with default
    settings against all 153 Genomatix MatBase 7.0 vertebrate matrix families
- name: Genomatix suite
  role: Platform in which MatInspector motif scans are executed
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:005e04fc203add981d26607d45346386506f28a693d66f8e85d25cd1dde102ce
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
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# peak-caller-benchmark-comparison

## Summary

A benchmarking workflow that runs multiple ChIP-Seq peak-calling algorithms (HPeak, PeakFinder, FindPeaks, MACS, SISSRs, CisGenome, ChIPseeqer) on the same treated/control data and compares them by reported peak counts and transcription-factor motif enrichment of called peaks, using MatInspector/Genomatix scans against Genomatix MatBase 7.0 matrices.

## When to use

Use when benchmarking ChIP-Seq peak callers on one shared treated+control dataset (e.g., NRSF GSE13047 or STAT1) to compare peak counts and TF-motif enrichment across HPeak, PeakFinder, FindPeaks, MACS, SISSRs, CisGenome, and ChIPseeqer.

## When NOT to use

- Data is not ChIP-Seq read data (e.g., already a peak BED file or feature count matrix) — this skill requires raw treated/control reads to run all callers on identical input.
- Comparative differential-binding analysis between two conditions (e.g., histone modification change detection as in ChIPDiff) rather than within-condition peak-caller benchmarking.
- Only one peak caller is of interest and no cross-algorithm comparison or motif-enrichment evaluation is needed — running a single caller suffices.

## Inputs

- ChIP-Seq treated sample reads (BED/ELAND formatted, e.g., NRSF GSE13047 or STAT1 Illumina data)
- Matched ChIP-Seq control (unstimulated/input) sample reads
- Peak-region FASTA sequences exported from each caller's output
- Genomatix MatBase 7.0 vertebrate motif matrix families (153 families) for MatInspector scans

## Outputs

- Per-caller peak counts table (e.g., peak_caller_comparison_results.csv)
- Peak-region FASTA files per caller
- Motif enrichment statistics (Chi-square) for caller-unique peak sets vs length-matched random controls
- WIG visualization files and peak annotation files from HPeak (-wig -ann)

## How to apply

Download the ChIP-Seq dataset with both ChIP-enriched (treated) and control runs, then run each peak caller with published/default parameters on identical input, recording the number of peaks each reports (on NRSF data the counts spanned 1,935 for PeakFinder to 5,243 for SISSRs; on STAT1, 9,561 for ChIPseeqer to 41,127 for FindPeaks). Run HPeak both treated-only and treated+control, extending short reads directionally into hypothetical DNA fragments (HDFs) with end-weight down-weighting, partitioning the genome into 25 bp bins, and applying a two-state HMM with GP (enriched) and ZIP (background) emissions via the Viterbi algorithm. Export peak-region sequences as FASTA (HPeak emits a sequence file for subsequent motif analysis) and scan all callers' peak FASTAs with MatInspector in the Genomatix suite using all 153 families of vertebrate motif matrices from Genomatix MatBase 7.0 with default settings. Quantify expected-TF-motif occurrence in peak versus length-matched random control sequences, and use Chi-square statistics to test whether the TF motif distinguishes one caller's unique peaks from another's (e.g., HPeak-unique vs FindPeaks-unique peaks on NRSF). Assemble caller peak counts and Chi-square results into a comparison table (peak_caller_comparison_results.csv).

## Related tools

- **HPeak** (HMM-based peak caller under evaluation; run in treated-only and treated+control modes with 25 bp bins, HDF extension, GP/ZIP emissions, and Viterbi decoding) — github.com/cauyrd/HPeak
- **PeakFinder** (Comparison peak caller run with published/default settings on the same NRSF data (reported 1,935 peaks))
- **FindPeaks** (Comparison peak caller; its unique NRSF peaks form the Chi-square contrast group against HPeak-unique peaks (41,127 peaks on STAT1))
- **MACS** (Comparison peak caller run with default parameters on the shared dataset)
- **SISSRs** (Comparison peak caller; upper bound of NRSF peak-count range (5,243 peaks))
- **CisGenome** (Comparison peak caller run with published/default parameter settings)
- **ChIPseeqer** (Comparison peak caller; lower bound of STAT1 peak-count range (9,561 peaks))
- **MatInspector** (Scans each caller's peak FASTA sequences for expected TF motifs with default settings against all 153 Genomatix MatBase 7.0 vertebrate matrix families)
- **Genomatix suite** (Platform in which MatInspector motif scans are executed)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- Reproduced NRSF peak counts for all seven callers fall within the reported range of 1,935 peaks (PeakFinder) to 5,243 peaks (SISSRs).
- HPeak STAT1 peak counts match the reported 24,394 (treated+control) and 43,440 (treated-only) values.
- STAT1 competitor peak counts fall within the 9,561 (ChIPseeqer) to 41,127 (FindPeaks) range.
- Computed Chi-square statistics for NRSF motif enrichment in HPeak-unique vs FindPeaks-unique peaks match the values in the paper's comparison tables.
- Motif-enrichment pattern reproduces Figure 2B: higher enrichment of the expected TF motif when the control sample is included in HPeak peak calling than in treated-only mode.

## Limitations

- Very little information on true positive and true negative loci is available for experimental ChIP-Seq data, making criteria for evaluating peak-calling algorithms challenging to determine; performance evaluation results should be interpreted with caution.
- The method is not well-suited for quantifying false detection rate, which may put some compared methods at a disadvantage.
- Many motif patterns stored in databases may not be accurate and novel motifs may not exist in motif databases, so motif-enrichment-based comparison may misrank callers.
- The current HPeak algorithm does not distinguish reads of different orientation within a peak, which has been shown to be informative for pinpointing peak summits.

## Evidence

- [discussion] Motif scan of the test and the control sequences was then performed using MatInspector in the Genomatix suite: "Motif scan of the test and the control sequences was then performed using MatInspector [52] in the Genomatix suite"
- [discussion] 25 bp bins and HDF counting: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] End-weight down-weighting rationale: "we gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF"
- [discussion] Two-state HMM with GP and ZIP emissions: "HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background"
- [discussion] HPeak outputs include a sequence file for motif analysis: "These include a wiggle (WIG) format file for visualization, a sequence file for subsequent motif analysis and an annotation file"
- [readme] HPeak README description: "HPeak is a hidden Markov model-based approach that can accurately pinpoint regions to where significantly more sequence reads map."
