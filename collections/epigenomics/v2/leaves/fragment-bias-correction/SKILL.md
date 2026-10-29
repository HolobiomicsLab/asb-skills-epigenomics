---
name: fragment-bias-correction
description: Use when estimating transcript expression from RNA-Seq alignments (SAM/BAM)
  whose library preparation (NSR priming, WT SOLiD, strand-specific protocols, single-
  or paired-end) induces non-uniform fragment distribution and quantification bias.
when_to_use_negative:
- Input is not fragment-level alignment data but already-computed expression estimates
  or a feature count table — bias correction operates on fragment alignments and reference
  sequences
- No reference transcriptome/genome FASTA or annotation is available — Cufflinks --bias-correction
  and Cuffdiff -b require the reference sequence, and eXpress-D estimation requires
  a transcript annotation
- The question is differential expression baseline calling where deliberately uncorrected
  (RH control) estimates are wanted for comparison — run the uncorrected mode instead
  of enabling correction
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3522
tools:
- name: eXpress-D
  role: Distributed engine performing likelihood-based simultaneous estimation of
    bias parameters and expression levels on mseq-formatted fragment alignments
  repo: github.com/adarob/eXpress-d
- name: Cufflinks
  role: Runs FPKM estimation per library with and without the --bias-correction flag
    to quantify replicate correlation improvements
- name: Cuffdiff
  role: Runs differential expression testing with and without fragment-bias correction
    (-b with reference genome FASTA) to measure the 46% decrease in called DE transcripts
- name: TopHat
  role: Splice-aware aligner producing BAM fragment alignments used as input to Cufflinks
    bias-corrected estimation
- name: Python
  role: Supplementary script that converts standard SAM and FASTA input into the mseq
    input format; also used with numpy/scipy to compute R2 between corrected and uncorrected
    estimates
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_004
  - task_006
  source_papers:
  - doi: 10.1186/gb-2011-12-3-r22
    title: Improving RNA-Seq expression estimates by correcting for fragment bias
  source_content_hash: sha256:ed738a969177488123498b5b565bbe45bdcd7ec80247efdf37810e85042201a5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cuffdiff
  - Cufflinks
  - Python
  - TopHat
  - eXpress-D
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3522
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# fragment-bias-correction

## Summary

Likelihood-based simultaneous estimation of RNA-Seq fragment bias parameters and transcript expression levels, as implemented in eXpress-D and Cufflinks/Cuffdiff, to correct sequence and positional bias from non-uniform cDNA fragmentation and priming.

## When to use

Use when estimating transcript expression from RNA-Seq alignments (SAM/BAM) whose library preparation (NSR priming, WT SOLiD, strand-specific protocols, single- or paired-end) induces non-uniform fragment distribution and quantification bias.

## When NOT to use

- Input is not fragment-level alignment data but already-computed expression estimates or a feature count table — bias correction operates on fragment alignments and reference sequences
- No reference transcriptome/genome FASTA or annotation is available — Cufflinks --bias-correction and Cuffdiff -b require the reference sequence, and eXpress-D estimation requires a transcript annotation
- The question is differential expression baseline calling where deliberately uncorrected (RH control) estimates are wanted for comparison — run the uncorrected mode instead of enabling correction

## Inputs

- SAM/BAM fragment alignments from single-end or paired-end RNA-Seq libraries (e.g. SRA020818 strand-specific yeast libraries, SRA008403/SRA010153 MAQC UHR libraries, SRA012427 Illumina MAQC, SOLiD WT-protocol BioScope alignments)
- FASTA reference genome or transcript sequences (required for Cufflinks --bias-correction and Cuffdiff -b)
- Transcript/gene annotation (e.g. GFF/GTF; Ensembl S. cerevisiae release 59 for the yeast analysis)
- Python conversion script to transform SAM + FASTA into mseq format for eXpress-D
- Optional independent qRT-PCR or NanoString expression measurements for validation

## Outputs

- Bias-corrected transcript-level expression estimates (FPKM or equivalent abundance values)
- Estimated bias parameters learned simultaneously with expression levels
- Corrected vs. uncorrected comparison tables (e.g. cross_platform_r2.csv, bias_correction_de_comparison.csv, replicate_correlations.csv)
- Fraction of NSR-vs-qRT-PCR discrepancy explained by correction (reported: 33.5%)
- Percent decrease in Cuffdiff differentially expressed transcripts after correction (reported: 46%)

## How to apply

Convert standard SAM alignments and a FASTA reference into the mseq input format with the provided Python conversion script, then run eXpress-D's likelihood-based simultaneous estimation of bias parameters and expression levels against a transcript annotation (e.g. Ensembl S. cerevisiae release 59). Run a companion estimation without bias correction (RH control setting) on the same library to serve as an uncorrected baseline. The decision point is joining corrected and uncorrected estimates and comparing both against independent qRT-PCR measurements (e.g. on the MAQC HBR sample) to quantify the fraction of discrepancy explained — the paper reports 33.5% for the NSR-vs-qRT-PCR case. For replicate and cross-platform analyses, re-run Cufflinks with --bias-correction (or Cuffdiff with -b and the genome FASTA) and evaluate pairwise R2 between replicates (target >0.96 within SRA010153) or between platforms (0.74 uncorrected rising to 0.94 with both platforms corrected). Rationale: the problems of bias estimation and expression estimation are fundamentally linked and must be solved together, so bias parameters are inferred indirectly from fragment alignments rather than fixed a priori.

## Related tools

- **eXpress-D** (Distributed engine performing likelihood-based simultaneous estimation of bias parameters and expression levels on mseq-formatted fragment alignments) — github.com/adarob/eXpress-d
- **Cufflinks** (Runs FPKM estimation per library with and without the --bias-correction flag to quantify replicate correlation improvements)
- **Cuffdiff** (Runs differential expression testing with and without fragment-bias correction (-b with reference genome FASTA) to measure the 46% decrease in called DE transcripts)
- **TopHat** (Splice-aware aligner producing BAM fragment alignments used as input to Cufflinks bias-corrected estimation)
- **Python** (Supplementary script that converts standard SAM and FASTA input into the mseq input format; also used with numpy/scipy to compute R2 between corrected and uncorrected estimates)

## Examples

```
cufflinks --bias-correction reference.fa alignments.bam  # and for DE: cuffdiff -b reference.fa cond1.sam cond2.sam
```

## Evaluation signals

- Corrected estimates show improved correlation with independently performed qRT-PCR measurements (e.g. the NSR-primed MAQC HBR comparison where correction explained 33.5% of the discrepancy)
- Average within-replicate pairwise R2 across the four SRA010153 paired-end library preps exceeds 0.96 after --bias-correction
- Cross-platform Illumina-vs-SOLiD R2 reproduces the reported 0.74 (uncorrected), 0.88 (Illumina-only), 0.85 (SOLiD-only), and 0.94 (both corrected) within ±0.02
- Enabling Cuffdiff bias correction reduces significantly DE transcripts (q < 0.05) by ~46% relative to the uncorrected run
- Corrected-vs-uncorrected run comparison (RH control setting) shows corrected estimates moving closer to the qRT-PCR standard in both replicate sets

## Limitations

- Correction must be robust to library protocol variety — single- vs paired-end reads, strand-specific vs non-specific protocols, and different priming/fragmentation methods each change the bias model; results are protocol-specific (e.g. NSR-primed libraries show larger discrepancies than WT-protocol libraries)
- Only a fraction of discrepancy with qRT-PCR is explained (33.5% for NSR vs qRT-PCR), so residual disagreement with the qRT-PCR standard persists after correction
- Bias parameters are estimated indirectly from fragment alignments in the same experiment; very low-depth libraries may not support reliable indirect estimation
- eXpress-D documentation lives in the repository wiki; the README itself provides no runnable usage instructions

## Evidence

- [abstract] This non-uniformity must be accounted for when estimating expression levels, and we show how to perform the needed corrections using a likelihood based approach: "This non-uniformity must be accounted for when estimating expression levels, and we show how to perform the needed corrections using a likelihood based approach"
- [intro] Our main result is the development of a likelihood based approach for simultaneous estimation of bias parameters and expression levels using the likelihood framework of [6].: "Our main result is the development of a likelihood based approach for simultaneous estimation of bias parameters and expression levels using the likelihood framework of [6]."
- [abstract] We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR: "We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR"
- [supplementary] Python script that converts standard SAM and FASTA input into the mseq input format.: "Python script that converts standard SAM and FASTA input into the mseq input format."
- [readme] Distributed ambiguous fragment assignment using eXpress and Spark.: "Distributed ambiguous fragment assignment using [eXpress](http://bio.math.berkeley.edu/eXpress) and [Spark](http://spark.incubator.apache.org/)."
