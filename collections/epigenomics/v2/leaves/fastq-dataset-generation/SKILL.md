---
name: fastq-dataset-generation
description: Use when controlled synthetic FASTQ inputs are needed to benchmark bisulfite
  aligners under known error rates, and ground-truth read counts and lengths must
  be exactly reproducible.
when_to_use_negative:
- The evaluation target is real sequencing data (e.g. the WGBS dataset GEO GSE80911
  used elsewhere in the BiSpark paper) rather than simulated reads with known parameters.
- A different reference region, read length, or read count is required, since the
  skill is grounded in the specific chromosome 1 / 95 bp / 1M-read configuration.
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Sherman
  role: Bisulfite-treated Read FastQ Simulator used to generate the three simulated
    read sets with varying error rates
- name: BiSpark
  role: 'Downstream consumer: the simulated FASTQ sets serve as the shared input for
    BiSpark''s mapping-quality and alignment-time evaluations'
  repo: https://github.com/bhi-kimlab/BiSpark/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:e8f567ce492016454ca7ab79209f1aaa007fcc63de0cfecccbaaf06280a1b4aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - BiSpark
  - Sherman
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# fastq-dataset-generation

## Summary

Simulates bisulfite-treated sequencing FASTQ read sets with Sherman using fixed reference, read-length, and read-count parameters (human chromosome 1 of hg19, 95 bp, 1,000,000 reads) while varying the error ratio (0%, 1%, 2%), producing controlled synthetic inputs for aligner benchmarking such as BiSpark's mapping-quality evaluation.

## When to use

Use when controlled synthetic FASTQ inputs are needed to benchmark bisulfite aligners under known error rates, and ground-truth read counts and lengths must be exactly reproducible.

## When NOT to use

- The evaluation target is real sequencing data (e.g. the WGBS dataset GEO GSE80911 used elsewhere in the BiSpark paper) rather than simulated reads with known parameters.
- A different reference region, read length, or read count is required, since the skill is grounded in the specific chromosome 1 / 95 bp / 1M-read configuration.

## Inputs

- Human chromosome 1 sequence in FASTA format from reference genome Build 37 (hg19)
- Sherman simulation parameters: read length (95 bp), number of reads (1,000,000), error ratio (0%, 1%, or 2%)

## Outputs

- sim_0err.fq — FASTQ with 1,000,000 bisulfite-converted 95 bp reads at 0% error (~122 MB)
- sim_1err.fq — FASTQ with 1,000,000 bisulfite-converted 95 bp reads at 1% error (~122 MB)
- sim_2err.fq — FASTQ with 1,000,000 bisulfite-converted 95 bp reads at 2% error (~122 MB)

## How to apply

Extract the target reference sequence (human chromosome 1 from Build 37, hg19) and run Sherman (bisulfite-treated Read FastQ Simulator) on it three times, each configured with read length 95 bp and 1,000,000 reads. Vary only the error ratio parameter across runs — 0%, 1%, and 2% — which Sherman interprets as a mean per-base error rate with an exponential decay error curve. Write each run to a separate FASTQ file (e.g. sim_0err.fq, sim_1err.fq, sim_2err.fq) so error rate becomes the single controlled variable for downstream aligner comparison. Rationale: the original BiSpark mapping-quality evaluation used exactly these simulated read sets, and the 0% error set is the condition on which BiSpark showed the best performance on all four evaluation metrics.

## Related tools

- **Sherman** (Bisulfite-treated Read FastQ Simulator used to generate the three simulated read sets with varying error rates)
- **BiSpark** (Downstream consumer: the simulated FASTQ sets serve as the shared input for BiSpark's mapping-quality and alignment-time evaluations) — https://github.com/bhi-kimlab/BiSpark/

## Evaluation signals

- Each output FASTQ contains exactly 1,000,000 reads.
- Every read in each FASTQ is exactly 95 bp long.
- Each file is approximately 122 MB, matching the simulation data statistics reported in Table 1.
- Exactly three files are produced, one per error rate (0%, 1%, 2%), differing only in the error ratio parameter.
- All three runs are generated from the same chromosome 1 (hg19) reference, so read sets are directly comparable across error conditions.

## Limitations

- The article provides no software version numbers for the simulation or alignment tools, so bit-for-bit reproduction may require version discovery.
- Sherman's error model (exponential decay around a mean per-bp error rate) is a simulator approximation of sequencing errors, not an empirical error profile.
- Only chromosome 1 is simulated, so results do not capture whole-genome scale behavior beyond the dedicated 10M–200M read scalability tests.

## Evidence

- [methods] Sherman simulation configuration: "setting up with human chromosome 1, read length to 95bp, and the number of reads to 1,000,000"
- [methods] BiSpark best at 0% error: "the BiSpark shows the best performance on all four metrics with the 0% error dataset"
- [readme] Read quality preprocessing context: "To improve the mappability and alignment accuracy, snitizing the poor reads before the main BiSpark phase is highly recommended."
- [abstract] BiSpark scalability claim: "BiSpark significantly outperforms other state-of-the-art bisulfite sequencing aligners in terms of alignment speed and scalability"
