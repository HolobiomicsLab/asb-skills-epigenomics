---
name: ngs-error-model-parameterization
description: Use when an aligner-evaluation experiment needs synthetic bisulfite-seq
  FASTQ read sets with a controlled per-base error rate (e.g. 0%, 1%, 2%) and known
  genomic origin, so mapping quality can be scored against ground truth.
when_to_use_negative:
- The evaluation requires real experimental read data (e.g. a WGBS dataset like GEO
  GSE80911) rather than simulated reads with a known ground truth.
- The aligner under test is not a bisulfite-seq aligner — Sherman generates bisulfite-treated
  reads that would not exercise normal aligner behavior.
- Only alignment throughput/scalability is being measured and error-rate-dependent
  accuracy is irrelevant; a single simulated or real read set suffices.
edam_operation: http://edamontology.org/operation_3580
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: Sherman
  role: Bisulfite-treated read FastQ simulator; generates the 95 bp, 1,000,000-read
    sets at 0%, 1%, and 2% error rates from human chromosome 1 (hg19)
- name: BiSpark
  role: Downstream bisulfite aligner whose mapping quality is evaluated on the simulated
    read sets
  repo: https://github.com/bhi-kimlab/BiSpark/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:232c3956ff62001ee924ed39289c0590eb670434618940744b86fee7525081ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - BiSpark
  - Sherman
  edam_operation: http://edamontology.org/operation_3580
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# ngs-error-model-parameterization

## Summary

Parameterize an NGS sequencing-error model by simulating bisulfite-treated reads with Sherman at controlled error rates (0%, 1%, 2%) so that aligner accuracy (mappability, precision, sensitivity, accuracy) can be evaluated against a known ground truth. In the BiSpark study this produced three simulated read sets from human chromosome 1 (hg19) that served as shared input for mapping-quality and alignment-time reproductions.

## When to use

Use when an aligner-evaluation experiment needs synthetic bisulfite-seq FASTQ read sets with a controlled per-base error rate (e.g. 0%, 1%, 2%) and known genomic origin, so mapping quality can be scored against ground truth.

## When NOT to use

- The evaluation requires real experimental read data (e.g. a WGBS dataset like GEO GSE80911) rather than simulated reads with a known ground truth.
- The aligner under test is not a bisulfite-seq aligner — Sherman generates bisulfite-treated reads that would not exercise normal aligner behavior.
- Only alignment throughput/scalability is being measured and error-rate-dependent accuracy is irrelevant; a single simulated or real read set suffices.

## Inputs

- Human chromosome 1 sequence from reference genome Build 37 (hg19), FASTA
- Sherman error-ratio settings: 0%, 1%, 2% (mean per-bp error rate with exponential decay curve)

## Outputs

- sim_0err.fq — 1,000,000 simulated bisulfite-treated reads, 95 bp, 0% error rate
- sim_1err.fq — 1,000,000 simulated bisulfite-treated reads, 95 bp, 1% error rate
- sim_2err.fq — 1,000,000 simulated bisulfite-treated reads, 95 bp, 2% error rate

## How to apply

Obtain the target reference sequence — in the BiSpark study, human chromosome 1 from Build 37 (hg19). Run Sherman (bisulfite-treated Read FastQ Simulator) once per error condition, setting the read length to 95 bp and the number of reads to 1,000,000 per run, and varying the error ratio parameter across runs (0%, 1%, 2%); Sherman treats this ratio as a mean per-bp error rate with an exponential decay error curve. Write each run to a separate FASTQ file (sim_0err.fq, sim_1err.fq, sim_2err.fq) so the same simulated sets can be shared across downstream mapping-quality and alignment-time evaluations. Validate each output against the expected simulation statistics before using it as aligner input.

## Related tools

- **Sherman** (Bisulfite-treated read FastQ simulator; generates the 95 bp, 1,000,000-read sets at 0%, 1%, and 2% error rates from human chromosome 1 (hg19))
- **BiSpark** (Downstream bisulfite aligner whose mapping quality is evaluated on the simulated read sets) — https://github.com/bhi-kimlab/BiSpark/

## Evaluation signals

- Each output FASTQ contains exactly 1,000,000 reads of 95 bp length.
- Each simulated set totals approximately 122 MB, matching the simulation data statistics reported in Table 1.
- Three distinct error conditions (0%, 1%, 2%) are produced as separate FASTQ files, not collapsed into one set.
- Reads derive only from the specified reference region (human chromosome 1 of hg19/Build 37).
- As a behavioral sanity check, the aligner under test should achieve its best scores on all four mapping metrics with the 0% error dataset.

## Limitations

- Sherman treats the error ratio as a mean per-bp error rate with an exponential decay error curve, so errors are not uniformly distributed across read positions.
- Simulated sets only cover a single read length (95 bp) and a single reference region (chromosome 1), which may not transfer to other library designs.
- Simulated reads do not include real-library artifacts such as adapter contamination; if desired, follow-up pre-processing steps (Cutadapt, FastQC) are recommended in the BiSpark README for quality control.

## Evidence

- [methods] setting up with human chromosome 1, read length to 95bp, and the number of reads to 1,000,000: "setting up with human chromosome 1, read length to 95bp, and the number of reads to 1,000,000"
- [methods] the BiSpark shows the best performance on all four metrics with the 0% error dataset: "the BiSpark shows the best performance on all four metrics with the 0% error dataset"
- [readme] To improve the mappability and alignment accuracy, snitizing the poor reads before the main BiSpark phase is highly recommended.: "To improve the mappability and alignment accuracy, snitizing the poor reads before the main BiSpark phase is highly recommended."
