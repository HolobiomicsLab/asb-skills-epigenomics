---
name: frip-score-thresholding
description: Use when a ChIP-seq or DNase-seq sample needs a per-sample enrichment
  QC score, or when deciding whether a 1% FRiP threshold is an appropriate pass/fail
  criterion for ChIP versus input control samples.
when_to_use_negative:
- The sample has no called peaks yet — FRiP requires peaks from the merged peak calling
  step and cannot be computed on raw reads alone
- The goal is fragment-size or PCR-duplicate QC — ChiLin handles those separately
  via macs2 fragment size estimation (step 3) and PBC evaluation (step 4), not via
  FRiP
- Peaks number fewer than 200 for downstream steps such as MDSeqPos motif analysis
  — ChiLin skips those steps automatically; FRiP is not a substitute for them
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: ChiLin
  role: Runs the QC pipeline whose step 7 calculates FRiP scores for each sample;
    supplies the post-processing script compile_database.sh for compiling FRiP JSON
    records into the QC database
  repo: https://github.com/cfce/chilin
- name: MACS2
  role: Default peak caller used to produce the peaks against which the fraction of
    reads in peaks (FRiP) is computed
  repo: https://github.com/taoliu/MACS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:720fa5788d217bb5157f606c3d31b7d327b77c48b35015797e826f28f4e840fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - MACS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# frip-score-thresholding

## Summary

Assesses ChIP-seq/DNase-seq sample enrichment quality by computing the fraction of reads in peaks (FRiP) and evaluating it against a 1% FRiP cutoff, which the ChiLin quality atlas validated as a proper ChIP enrichment reference.

## When to use

Use when a ChIP-seq or DNase-seq sample needs a per-sample enrichment QC score, or when deciding whether a 1% FRiP threshold is an appropriate pass/fail criterion for ChIP versus input control samples.

## When NOT to use

- The sample has no called peaks yet — FRiP requires peaks from the merged peak calling step and cannot be computed on raw reads alone
- The goal is fragment-size or PCR-duplicate QC — ChiLin handles those separately via macs2 fragment size estimation (step 3) and PBC evaluation (step 4), not via FRiP
- Peaks number fewer than 200 for downstream steps such as MDSeqPos motif analysis — ChiLin skips those steps automatically; FRiP is not a substitute for them

## Inputs

- BAM files of aligned ChIP-seq or DNase-seq reads (from bwa/bowtie/star mapping step)
- Called peak files (from the MACS2 peak calling step)
- Compiled FRiP JSON QC records (e.g. id_frip, aggregated via compile_database.sh)
- Sample class labels distinguishing ChIP (treatment) from input (control)

## Outputs

- Per-sample FRiP scores (step 7 output of ChiLin)
- frip_cutoff_summary.csv with per-class pass counts and proportions (expected: 12,032/14,866 = 80.9% ChIP; 4,289/8,811 = 48.6% input)

## How to apply

Run ChiLin's QC pipeline so that peaks are first called (MACS2 is the default peak caller, pinned at macs2=2.1.0 in the reference conda install), then execute step 7 to calculate the FRiP score for each sample: the fraction of sequencing reads falling inside called peaks. Load FRiP JSON records (e.g. id_frip) compiled by the compile_database.sh post-processing script across the 23,677 public ChIP-seq and DNase-seq samples, and stratify records into ChIP (treatment) versus input (control) classes. Apply the threshold by counting a sample as passing if its FRiP score is greater than 0.01, then compute pass counts and proportions per class and write frip_cutoff_summary.csv. The rationale for the 1% cutoff is empirical: in the compiled atlas it passes 80.9% (12,032/14,866) of ChIP samples but only 48.6% (4,289/8,811) of input controls, cleanly separating enriched treatments from controls.

## Related tools

- **ChiLin** (Runs the QC pipeline whose step 7 calculates FRiP scores for each sample; supplies the post-processing script compile_database.sh for compiling FRiP JSON records into the QC database) — https://github.com/cfce/chilin
- **MACS2** (Default peak caller used to produce the peaks against which the fraction of reads in peaks (FRiP) is computed) — https://github.com/taoliu/MACS

## Examples

```
chilin -h
```

## Evaluation signals

- ChIP class pass proportion at FRiP > 0.01 equals 12,032/14,866 = 80.9% within rounding (0.809)
- Input class pass proportion equals 4,289/8,811 = 48.6% within rounding (0.486)
- Class totals equal 14,866 ChIP samples and 8,811 input samples, summing to the 23,677-sample quality atlas cohort
- Every sample record in id_frip JSON has a parseable FRiP score and is assigned to exactly one class (ChIP or input)
- frip_cutoff_summary.csv contains one row per class with counts and proportions matching the reported values

## Limitations

- The 1% threshold was calibrated on the 23,677-sample public atlas (hg19/hg38/mm9/mm10 era data); it may not transfer directly to assays with very different peak widths or read depths
- FRiP is a threshold metric only: a sample with FRiP <= 0.01 may still be usable, and a passing sample is not guaranteed to be biologically meaningful
- ChiLin omits the FRiP step's dependence on peak quality — poorly called peaks (bad MACS2 settings) will deflate FRiP regardless of library enrichment

## Evidence

- [methods] FRiP step is pipeline step 7: "step 7: calculate FRiP scores for each sample."
- [other] 1% FRiP threshold validates ChIP enrichment: [evidence span withheld — anchor status: not_found]
- [intro] compile_database.sh aggregates FRiP records: "post-processing script for compiling QC database is provided at https://github.com/cfce/chilin/blob/master/demo/compile_database.sh"
- [abstract] ChiLin automates ChIP-seq/DNase-seq QC: "ChiLin is a computational pipeline that automates the quality control and data analyses of ChIP-seq and DNase-seq data."
- [intro] MACS2 is the default peak caller used by ChiLin: "ChiLin uses MACS2 (https://github.com/taoliu/MACS/) as the default peak caller"
