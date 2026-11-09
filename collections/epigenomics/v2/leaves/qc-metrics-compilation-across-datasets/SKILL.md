---
name: qc-metrics-compilation-across-datasets
description: Use when per-sample ChiLin QC JSON outputs (id_fastqc., id_map., id_frag.,
  id_pbc., id_frip., id_macs2., id_rep.) must be aggregated into a dataset-level summary
  table covering the 17 QC metrics across three QC layers for benchmarking or atlas
  building.
when_to_use_negative:
- Input is raw sequencing data needing only a single-sample QC run rather than cross-dataset
  compilation — use the per-sample ChiLin QC steps directly without the aggregation
  modules
- Data are not ChIP-seq or DNase-seq (e.g., RNA-seq or ATAC-seq) — ChiLin's QC metric
  set is designed specifically for ChIP-seq and DNase-seq experiments
- Only the final merged peak calls are needed and no quality assessment or historical
  comparison is intended
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0769
tools:
- name: ChiLin
  role: Pipeline that runs the 13-step QC workflow and emits per-sample QC JSON statistics;
    provides qc__summary / qc_summary_table aggregation modules and the compile_database.sh
    post-processing script
  repo: https://github.com/cfce/chilin
- name: FastQC
  role: 'Step 1: sequence quality evaluation, reads GC contents evaluation and library
    contamination, feeding the fastqc layer of QC metrics'
- name: bwa
  role: 'Step 2 (default mapper): produces the BAM files required by all subsequent
    QC metrics'
  repo: https://github.com/lh3/bwa
- name: MACS2
  role: 'Step 3: sub-samples BAM files and performs fragment size estimation; default
    peak caller for merged and replicate peak calling'
  repo: https://github.com/taoliu/MACS
- name: samtools
  role: BAM handling dependency installed via conda in the ChiLin environment
- name: bedtools
  role: Interval arithmetic dependency (conda install bedtools=2.17.0) supporting
    peak-based QC computations
- name: MDSeqPos
  role: 'Optional step 12: motif analysis on called peaks (skipped automatically for
    fewer than 200 peaks)'
  repo: https://bitbucket.org/cistrome/cistrome-applications-harvard
- name: seqtk
  role: FASTQ/FASTA processing dependency available to the pipeline
  repo: https://github.com/lh3/seqtk
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:92f8e56129879e0dd08c2994dc25276df1f0503382d00c3d7e00b49c8bbe1a37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - Cistrome
  - FastQC
  - MACS2
  - MDSeqPos
  - R
  - bedtools
  - bwa
  - samtools
  - seqtk
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# qc-metrics-compilation-across-datasets

## Summary

Aggregate per-sample ChIP-seq/DNase-seq quality-control statistics produced by the ChiLin pipeline (FastQC, mapping, fragment-size, PBC, replicate overlap, FRiP, MACS2, meta-region metrics) into a unified plain-text and tabular quality atlas across many samples and datasets, enabling comparison against historical reference metrics.

## When to use

Use when per-sample ChiLin QC JSON outputs (id_fastqc., id_map., id_frag., id_pbc., id_frip., id_macs2., id_rep.) must be aggregated into a dataset-level summary table covering the 17 QC metrics across three QC layers for benchmarking or atlas building.

## When NOT to use

- Input is raw sequencing data needing only a single-sample QC run rather than cross-dataset compilation — use the per-sample ChiLin QC steps directly without the aggregation modules
- Data are not ChIP-seq or DNase-seq (e.g., RNA-seq or ATAC-seq) — ChiLin's QC metric set is designed specifically for ChIP-seq and DNase-seq experiments
- Only the final merged peak calls are needed and no quality assessment or historical comparison is intended

## Inputs

- Single-end or paired-end ChIP-seq/DNase-seq FASTQ files (treatment and control) from GEO-deposited datasets
- Per-dataset ChiLin configuration files listed in a batch.conf file
- Species reference data (hg19/hg38/mm9/mm10 genome index, chrom_len, DHS/blacklist, geneTable) from cistrome.org/chilin/_downloads
- Per-sample QC JSON statistics (id_fastqc., id_map., id_frag., id_pbc., id_frip., id_macs2., id_rep.)

## Outputs

- Plain-text and tabular summary of 17 QC metrics across three QC layers for every sample and dataset
- Compiled quality atlas table comparable to Additional file 2: Table S1 (e.g., 23,677 samples, 11,265 datasets, eight categories)

## How to apply

Run ChiLin per dataset via 'chilin gen' to write a config (species, treatment/control FASTQ paths, narrow/broad peak type, reference genome index, chrom_len, DHS/blacklist, geneTable), then launch batches with 'chilin batch -b batch.conf', which executes the 13-step workflow (FastQC quality/GC/contamination, bwa mapping, macs2 fragment-size estimation, PBC, replicate peak overlap/correlation, merged peak calling, FRiP, bedAnnotate.py meta-region annotation, optional PhastCons, regulatory potential and MDSeqPos steps). Parse the per-sample QC JSON statistics emitted to the / directory (id_fastqc., id_map., id_frag., id_pbc., id_frip., id_macs2., id_rep., etc.). Aggregate the parsed metrics with the summary modules (qc__summary / qc_summary_table) into a plain-text and tabular summary covering the 17 QC metrics across the three QC layers for every sample and dataset. Validation compares the compiled table against the reference atlas of 23,677 public samples from 11,265 datasets across eight literature-based categories (Additional file 2: Table S1).

## Related tools

- **ChiLin** (Pipeline that runs the 13-step QC workflow and emits per-sample QC JSON statistics; provides qc__summary / qc_summary_table aggregation modules and the compile_database.sh post-processing script) — https://github.com/cfce/chilin
- **FastQC** (Step 1: sequence quality evaluation, reads GC contents evaluation and library contamination, feeding the fastqc layer of QC metrics)
- **bwa** (Step 2 (default mapper): produces the BAM files required by all subsequent QC metrics) — https://github.com/lh3/bwa
- **MACS2** (Step 3: sub-samples BAM files and performs fragment size estimation; default peak caller for merged and replicate peak calling) — https://github.com/taoliu/MACS
- **samtools** (BAM handling dependency installed via conda in the ChiLin environment)
- **bedtools** (Interval arithmetic dependency (conda install bedtools=2.17.0) supporting peak-based QC computations)
- **MDSeqPos** (Optional step 12: motif analysis on called peaks (skipped automatically for fewer than 200 peaks)) — https://bitbucket.org/cistrome/cistrome-applications-harvard
- **seqtk** (FASTQ/FASTA processing dependency available to the pipeline) — https://github.com/lh3/seqtk

## Examples

```
chilin batch -b batch.conf && bash compile_database.sh
```

## Evaluation signals

- The compiled table contains 17 QC metrics across three QC layers for every sample and dataset, matching the expected schema
- Row and column counts reconcile with the number of input samples and datasets (reference scale: 23,677 samples from 11,265 datasets across eight classified categories)
- Per-sample QC JSON files (id_fastqc., id_map., id_frag., id_pbc., id_frip., id_macs2., id_rep.) exist for every processed sample before aggregation
- Aggregated FRiP, PBC, fragment-size, and replicate overlap/correlation values fall within plausible historical ranges relative to the compiled public atlas
- The summary is emitted in both plain-text and tabular form by the qc__summary / qc_summary_table modules without missing per-sample entries

## Limitations

- No software version is stated for ChiLin in the discussion, complicating exact reproduction of the compiled database
- Reproducibility is claimed with 'only a simple command', but no concrete commands, environment specification, or output artifacts are provided for the compilation step in the article; the compile_database.sh script at github.com/cfce/chilin/blob/master/demo/ is the reference
- Several pipeline steps are skippable or conditional (FastQC step 1, PBC step 4 if step 3 ran, PhastCons step 10, regulatory potential step 11, MDSeqPos step 12 skipped for <200 peaks), so the set of metrics present in the compiled table can vary across datasets

## Evidence

- [full_text] batch workflow and JSON outputs: [evidence span withheld — anchor status: not_found]
- [abstract] atlas scale: "comparisons with historical data derived from over 23,677 public ChIP-seq and DNase-seq samples (11,265 datasets) from eight literature-based classified categories"
- [intro] compilation script: "post-processing script for compiling QC database is provided at https://github.com/cfce/chilin/blob/master/demo/compile_database.sh"
- [methods] workflow steps: "step 1(*can skip*): FastQC sequence quality evaluation, reads GC contents evaluation and library contamination"
- [readme] installation commands: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0"
