---
name: transcript-expression-quantification
description: Use when RNA-Seq alignments (BAM/SAM, e.g. from SRA012427 or MAQC HBR
  libraries) must be converted into per-transcript expression estimates whose non-uniform
  fragment bias has been corrected before correlation or differential-expression analysis.
when_to_use_negative:
- When expression estimates have already been bias-corrected upstream (re-correcting
  the same alignments risks double-modeling fragment bias)
- When no reference genome FASTA is available for the alignments, since the -b fragment-bias
  correction cannot be applied
- When the input is not fragment-level alignment data (e.g. already-aggregated count
  or FPKM tables), since the likelihood approach operates on fragment alignments
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3522
tools:
- name: Cufflinks
  role: Estimates transcript expression from mapped BAM/SAM alignments, run with and
    without fragment-bias correction (-b genome FASTA) to produce corrected and uncorrected
    FPKM estimates
- name: Cuffdiff
  role: Tests differential expression on Illumina/SOLiD MAQC sample pairs with and
    without -b bias correction; DE counts at q 0.05 quantify the correction's effect
- name: Python
  role: Supplementary script converts standard SAM and FASTA input into the mseq input
    format; also used to compute R2 correlations and permutation tests
- name: NanoString
  role: Independent NanoString expression measurements used as supplementary validation
    data for corrected expression estimates
- name: eXpress-D
  role: Distributed ambiguous fragment assignment implementation of the eXpress likelihood
    framework for fragment-bias-aware expression estimation
  repo: github.com/adarob/eXpress-d
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1186/gb-2011-12-3-r22
    title: Improving RNA-Seq expression estimates by correcting for fragment bias
  source_content_hash: sha256:9d6c357b81993f0c5e7ce9be6361d7b56ce143993e69426acd58dca223a685a3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cuffdiff
  - Cufflinks
  - NanoString
  - Python
  - eXpress-D
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3522
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# transcript-expression-quantification

## Summary

Estimate transcript-level expression from RNA-Seq alignments with a likelihood-based approach that simultaneously models fragment bias (non-uniform cDNA fragment distribution) and abundance, yielding corrected expression estimates and improved cross-library replicability.

## When to use

Use when RNA-Seq alignments (BAM/SAM, e.g. from SRA012427 or MAQC HBR libraries) must be converted into per-transcript expression estimates whose non-uniform fragment bias has been corrected before correlation or differential-expression analysis.

## When NOT to use

- When expression estimates have already been bias-corrected upstream (re-correcting the same alignments risks double-modeling fragment bias)
- When no reference genome FASTA is available for the alignments, since the -b fragment-bias correction cannot be applied
- When the input is not fragment-level alignment data (e.g. already-aggregated count or FPKM tables), since the likelihood approach operates on fragment alignments

## Inputs

- Mapped RNA-Seq alignments in BAM/SAM format (e.g. SRA012427 alignments, Illumina and SOLiD MAQC libraries)
- Reference transcript annotation (GTF/GFF)
- Reference genome FASTA (required for fragment-bias correction via -b)
- Independent qRT-PCR measurements with uniquely mapping TaqMan probes (907 transcripts on MAQC HBR)

## Outputs

- Per-transcript expression estimates (uncorrected and bias-corrected)
- qpcr_correlation_results.csv with uncorrected R2, corrected R2, and permutation-test P-value
- bias_correction_de_comparison.csv with DE transcript counts per platform and percent decrease
- Set of significantly differentially expressed transcripts at q-value 0.05 per run

## How to apply

Load mapped alignments (BAM/SAM) against a reference transcript annotation into Cufflinks and run one pass in default mode (no bias correction) and a second pass with fragment-bias correction enabled, supplying the reference genome FASTA via the -b option. The method uses a likelihood framework to jointly estimate bias parameters and expression levels, accounting for the non-uniform distribution of cDNA fragments. Validate correction quality by correlating estimates against independently measured qRT-PCR (e.g. MAQC Human Brain Reference, retaining only the 907 transcripts with uniquely mapping TaqMan probes) and computing squared Pearson R2 for uncorrected versus corrected estimates, testing the improvement by permutation test. For differential analysis, run Cuffdiff on the same sample pairs (e.g. Illumina vs SOLiD MAQC) with and without -b, counting significantly DE transcripts at q-value 0.05 in each run and computing the percent decrease. Write uncorrected R2, corrected R2, and permutation P to a results CSV alongside per-platform DE counts.

## Related tools

- **Cufflinks** (Estimates transcript expression from mapped BAM/SAM alignments, run with and without fragment-bias correction (-b genome FASTA) to produce corrected and uncorrected FPKM estimates)
- **Cuffdiff** (Tests differential expression on Illumina/SOLiD MAQC sample pairs with and without -b bias correction; DE counts at q 0.05 quantify the correction's effect)
- **Python** (Supplementary script converts standard SAM and FASTA input into the mseq input format; also used to compute R2 correlations and permutation tests)
- **NanoString** (Independent NanoString expression measurements used as supplementary validation data for corrected expression estimates)
- **eXpress-D** (Distributed ambiguous fragment assignment implementation of the eXpress likelihood framework for fragment-bias-aware expression estimation) — github.com/adarob/eXpress-d

## Examples

```
cufflinks -b genome.fa alignments.bam; cuffdiff -b genome.fa -o corrected_out condition1.bam condition2.bam
```

## Evaluation signals

- On MAQC HBR, bias-corrected R2 with qRT-PCR must exceed uncorrected R2 (0.807 > 0.753) over the same 907 uniquely probed transcripts
- Permutation test comparing corrected vs uncorrected correlations must yield P = 0.0007 (highly significant improvement)
- Cuffdiff with bias correction on Illumina and SOLiD MAQC pairs must show ~46% fewer significantly DE transcripts (q 0.05) than the default run
- Correlation must be computed over an identical transcript set for corrected and uncorrected estimates so the comparison is paired
- Output CSVs must contain the specified fields: uncorrected R2, corrected R2, P-value (qpcr_correlation_results.csv) and counts plus percent decrease (bias_correction_de_comparison.csv)

## Limitations

- The method must remain robust across single- and paired-end reads, strand-specific and non-specific protocols, and a variety of priming and fragmentation methods; bias correction may not transfer cleanly to new library-preparation regimes
- Bias estimation and expression estimation are fundamentally linked and must be solved together — correcting expression without jointly inferring bias parameters from fragment alignments is not valid
- Bias correction reduces the number of reported differentially expressed transcripts (~46%), so downstream DE lists change substantially and should not be compared across corrected/uncorrected runs without noting this

## Evidence

- [abstract] This non-uniformity must be accounted for when estimating expression levels, and we show how to perform the needed corrections using a likelihood based approach: "This non-uniformity must be accounted for when estimating expression levels, and we show how to perform the needed corrections using a likelihood based approach"
- [abstract] We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR: "We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR"
- [abstract] correction of bias leads to improved replicability of results across libraries and sequencing technologies: "correction of bias leads to improved replicability of results across libraries and sequencing technologies"
- [intro] Our main result is the development of a likelihood based approach for simultaneous estimation of bias parameters and expression levels: "Our main result is the development of a likelihood based approach for simultaneous estimation of bias parameters and expression levels using the likelihood framework of [6]."
- [readme] Distributed ambiguous fragment assignment using eXpress and Spark.: "Distributed ambiguous fragment assignment using [eXpress](http://bio.math.berkeley.edu/eXpress) and [Spark](http://spark.incubator.apache.org/)."
