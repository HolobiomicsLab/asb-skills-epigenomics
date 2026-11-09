---
name: fastqc-sequence-quality-evaluation
description: Use when raw FASTQ files from ChIP-seq or DNase-seq experiments need
  sequence-quality screening, and when new samples must be compared against cumulative
  historical quality distributions (FastQC score >25, uniquely mapped ratio >0.5,
  PBC >0.8).
when_to_use_negative:
- Data is not ChIP-seq or DNase-seq (e.g. RNA-seq or ATAC-seq), since the atlas cutoffs
  and eight literature-based categories were compiled specifically from ChIP-seq/DNase-seq
  samples
- Only peak-level QC is needed (FRiP, fragment size, replicate overlap) — those are
  separate ChiLin steps 3/5/7, not the read-layer FastQC/mapping/PBC evaluation
- The user requires RNA-programming-free interactive QC only; the comparison rendering
  (id_bwa_compare) assumes R is available
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: ChiLin
  role: Unified pipeline that runs the FastQC (step 1), bwa mapping (step 2) and PBC
    (step 4) QC steps and generates QC reports with historical comparisons
  repo: https://github.com/cfce/chilin
- name: FastQC
  role: Step 1 sequence quality evaluation, GC contents evaluation and library contamination
    check
- name: bwa
  role: Step 2 default mapper producing the BAM files needed for mapping-ratio and
    PBC metrics
  repo: https://github.com/lh3/bwa
- name: samtools
  role: BAM handling supporting the mapping and PBC evaluation steps
- name: R
  role: Runs the id_bwa_compare script that renders new-sample vs. historical QC distribution
    plots
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:3d62e9632908d4e3ad4884a51786d2bb92b88b703a77303b697b036fbf3c334f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - FastQC
  - R
  - bwa
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# fastqc-sequence-quality-evaluation

## Summary

ChiLin's step-1 read-layer QC module: run FastQC sequence quality evaluation (plus GC content and library contamination checks) on ChIP-seq/DNase-seq FASTQ inputs and benchmark per-sample metrics against a compiled atlas of 23,677 public samples (11,265 datasets) to apply assay-insensitive cutoffs.

## When to use

Use when raw FASTQ files from ChIP-seq or DNase-seq experiments need sequence-quality screening, and when new samples must be compared against cumulative historical quality distributions (FastQC score >25, uniquely mapped ratio >0.5, PBC >0.8).

## When NOT to use

- Data is not ChIP-seq or DNase-seq (e.g. RNA-seq or ATAC-seq), since the atlas cutoffs and eight literature-based categories were compiled specifically from ChIP-seq/DNase-seq samples
- Only peak-level QC is needed (FRiP, fragment size, replicate overlap) — those are separate ChiLin steps 3/5/7, not the read-layer FastQC/mapping/PBC evaluation
- The user requires RNA-programming-free interactive QC only; the comparison rendering (id_bwa_compare) assumes R is available

## Inputs

- Raw FASTQ files from ChIP-seq or DNase-seq samples
- Reference genome index (e.g. hg19/hg38/mm9/mm10 tgz packages from cistrome.org) for the bwa mapping step
- Subsampled BAM files for PBC evaluation
- Compiled per-sample QC JSON records (fastqc evaluation, mapping ratio statistics, PBC score)

## Outputs

- Per-sample FastQC quality metrics (id_fastqc. outputs)
- Mapping ratio statistics (id_map. outputs) and PBC scores (id_pbc. outputs)
- QC comparison plot id_bwa_compare.pdf via the id_bwa_compare R script
- Aggregated category-level cutoff and median-depth table (e.g. qc_metric_summary.csv)

## How to apply

Run the ChiLin pipeline QC steps on each sample: step 1 FastQC sequence quality evaluation (can be skipped, but is the read-quality entry point), step 2 bwa (default) mapping to produce BAM files, and step 4 PBC evaluation on subsampled BAM files, generating per-sample id_fastqc., id_map., and id_pbc. outputs. Parse the per-sample QC JSON records (FastQC quality, mapping ratio, PBC) with Python, assign samples to the eight literature-based categories, and compute distributions plus cumulative pass fractions against the atlas cutoffs. The paper reports >85% of the 23,677 public samples have sequence quality score >25, uniquely mapped read ratio >0.5 and PBC >0.8, and median read depths of 25M (ChIP-seq) and 40M (DNase-seq); these serve as the comparison baselines. Render the historical-distribution comparison with the id_bwa_compare R script to produce id_bwa_compare.pdf, and write aggregated cutoff/depth tables (e.g. qc_metric_summary.csv). Validate by confirming the reproduced distributions match the reported cumulative cutoffs and flagging categories or samples that deviate.

## Related tools

- **ChiLin** (Unified pipeline that runs the FastQC (step 1), bwa mapping (step 2) and PBC (step 4) QC steps and generates QC reports with historical comparisons) — https://github.com/cfce/chilin
- **FastQC** (Step 1 sequence quality evaluation, GC contents evaluation and library contamination check)
- **bwa** (Step 2 default mapper producing the BAM files needed for mapping-ratio and PBC metrics) — https://github.com/lh3/bwa
- **samtools** (BAM handling supporting the mapping and PBC evaluation steps)
- **R** (Runs the id_bwa_compare script that renders new-sample vs. historical QC distribution plots)

## Examples

```
conda create -n chilin_env python=2 && conda activate chilin_env && conda install bwa=0.7.13 samtools=0.1.19 fastqc macs2=2.1.0 bioconductor-seqlogo && python setup.py install && chilin -h
```

## Evaluation signals

- More than 85% of the analyzed samples show FastQC sequence quality score >25, uniquely mapped read ratio >0.5, and PBC >0.8
- Category-level median read depths match the atlas values of 25M for ChIP-seq and 40M for DNase-seq
- Per-sample outputs id_fastqc., id_map. and id_pbc. exist for every sample that passed step 2 (bwa mapping cannot be skipped)
- id_bwa_compare.R runs without error and produces id_bwa_compare.pdf; aggregated cutoff table qc_metric_summary.csv is written with the expected columns
- Any category whose cumulative distributions deviate from the reported cutoffs is explicitly flagged

## Limitations

- Step 1 (FastQC) is optional (can be skipped) in ChiLin, so quality evaluation may be absent if only downstream peak steps are run
- The historical cutoffs come from a single compiled atlas of 23,677 samples in eight categories and may not transfer to new assay types or read lengths
- No software version for ChiLin is stated, and the reproducibility claim is not backed by concrete environment specifications in the paper
- ChiLin's Python environment requires python 2 (conda create -n chilin_env python=2) with pinned versions such as bwa=0.7.13, samtools=0.1.19, bedtools=2.17.0, macs2=2.1.0, which may be hard to reproduce on modern systems

## Evidence

- [other] >85% of samples have sequence quality score >25, uniquely mapped read ratio >0.5 and PBC >0.8, suggesting these as usable QC cutoffs for ChIP-seq data: [evidence span withheld — anchor status: not_found]
- [methods] step 1(*can skip*): FastQC sequence quality evaluation, reads GC contents evaluation and library contamination: "step 1(*can skip*): FastQC sequence quality evaluation, reads GC contents evaluation and library contamination"
- [methods] step 4: sub-sample bam files(if step 3 is run, skip) and do PBC evaluation: "step 4: sub-sample bam files(if step 3 is run, skip) and do PBC evaluation"
- [abstract] comparisons with historical data derived from over 23,677 public ChIP-seq and DNase-seq samples (11,265 datasets) from eight literature-based classified categories: "comparisons with historical data derived from over 23,677 public ChIP-seq and DNase-seq samples (11,265 datasets) from eight literature-based classified categories"
- [readme] conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0"
