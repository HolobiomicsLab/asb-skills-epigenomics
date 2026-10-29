---
name: peak-calling-quality-assessment
description: Use when processing raw ChIP-seq or DNase-seq FASTQ files (treatment
  and control) and you need per-sample quality metrics to judge whether peak calls
  are reliable before downstream analysis.
when_to_use_negative:
- Input is not raw FASTQ/BAM ChIP-seq or DNase-seq data (e.g., already-called peak
  sets without reads) — the pipeline's mapping and fragment-size steps cannot run
- Peak number is less than 200, where ChiLin automatically skips the MDSeqPos motif
  step and motif-related QC interpretation is not available
- You need only motif or conservation analysis without QC — the skipped/optional steps
  (PhastCons, regulatory potential, MDSeqPos) are not the core of this skill
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0085
tools:
- name: ChiLin
  role: Unified 13-step QC pipeline that orchestrates all quality-control steps and
    emits per-sample JSON statistics
  repo: https://github.com/cfce/chilin
- name: bwa
  role: Default mapper producing the BAM files required by all downstream QC steps
  repo: https://github.com/lh3/bwa
- name: FastQC
  role: Step 1 sequence quality evaluation, reads GC contents and library contamination
    assessment
- name: MACS2
  role: Sub-sample BAM fragment-size estimation and default peak caller
  repo: https://github.com/taoliu/MACS
- name: samtools
  role: BAM handling during sub-sampling and QC evaluation
- name: bedtools
  role: Interval arithmetic for peak-overlap and meta-region QC calculations
- name: seqtk
  role: FASTQ/FASTA pre-processing dependency installed in the ChiLin environment
  repo: https://github.com/lh3/seqtk
- name: MDSeqPos
  role: Optional step 12 motif analysis on called peaks (skipped automatically for
    <200 peaks)
  repo: https://bitbucket.org/cistrome/cistrome-applications-harvard
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:3ddef8410e4ce10d23910d7cde8e2d39e7d6421c72f505d0a49745b09076d305
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# peak-calling-quality-assessment

## Summary

Assess the quality of ChIP-seq/DNase-seq peak calling using ChiLin's 13-step QC workflow, covering FastQC read quality, bwa mapping, MACS2 fragment-size estimation, PBC, replicate peak overlap/correlation, merged peak calling, FRiP scores, and meta-region annotation across 17 QC metrics in three layers.

## When to use

Use when processing raw ChIP-seq or DNase-seq FASTQ files (treatment and control) and you need per-sample quality metrics to judge whether peak calls are reliable before downstream analysis.

## When NOT to use

- Input is not raw FASTQ/BAM ChIP-seq or DNase-seq data (e.g., already-called peak sets without reads) — the pipeline's mapping and fragment-size steps cannot run
- Peak number is less than 200, where ChiLin automatically skips the MDSeqPos motif step and motif-related QC interpretation is not available
- You need only motif or conservation analysis without QC — the skipped/optional steps (PhastCons, regulatory potential, MDSeqPos) are not the core of this skill

## Inputs

- Single-end or paired-end ChIP-seq/DNase-seq FASTQ files (treatment and control) from GEO-deposited datasets
- Per-dataset ChiLin configuration file (from 'chilin gen') with species, FASTQ paths, and narrow/broad peak type
- Species reference data: genome index, chrom_len, DHS/blacklist, geneTable (hg19/hg38/mm9/mm10 tarballs)
- batch.conf file listing per-dataset configs

## Outputs

- Per-sample QC JSON statistics (id_fastqc., id_map., id_frag., id_pbc., id_frip., id_macs2., id_rep.)
- Aggregated plain-text and tabular summary of 17 QC metrics across three QC layers per sample/dataset (qc__summary / qc_summary_table)
- Merged peak calls and replicate overlap/correlation results

## How to apply

Generate a per-dataset config with 'chilin gen', filling the [basics] section with species, treatment/control FASTQ paths, and narrow/broad peak type, and confirm species reference data (genome index, chrom_len, DHS/blacklist, geneTable for hg19/hg38/mm9/mm10). List configs in a batch.conf and run 'chilin batch -b batch.conf' to execute the 13-step workflow: FastQC quality/GC/contamination evaluation, bwa mapping (default; bowtie or star alternatives), MACS2 fragment-size estimation on sub-sampled BAMs, PBC evaluation, replicate peak overlap/correlation, merged peak calling (never skipped, as it supplies peaks for annotation), FRiP score calculation, and bedAnnotate.py meta-region annotation. Parse the per-sample QC JSON files (id_fastqc., id_map., id_frag., id_pbc., id_frip., id_macs2., id_rep.) and aggregate with the qc__summary/qc_summary_table modules into a 17-metric, three-layer tabular summary. Correct application is judged by the compiled table matching the reference atlas structure — 17 metrics across three QC layers per sample, as in Additional file 2: Table S1 (23,677 samples, 11,265 datasets).

## Related tools

- **ChiLin** (Unified 13-step QC pipeline that orchestrates all quality-control steps and emits per-sample JSON statistics) — https://github.com/cfce/chilin
- **bwa** (Default mapper producing the BAM files required by all downstream QC steps) — https://github.com/lh3/bwa
- **FastQC** (Step 1 sequence quality evaluation, reads GC contents and library contamination assessment)
- **MACS2** (Sub-sample BAM fragment-size estimation and default peak caller) — https://github.com/taoliu/MACS
- **samtools** (BAM handling during sub-sampling and QC evaluation)
- **bedtools** (Interval arithmetic for peak-overlap and meta-region QC calculations)
- **seqtk** (FASTQ/FASTA pre-processing dependency installed in the ChiLin environment) — https://github.com/lh3/seqtk
- **MDSeqPos** (Optional step 12 motif analysis on called peaks (skipped automatically for <200 peaks)) — https://bitbucket.org/cistrome/cistrome-applications-harvard

## Examples

```
chilin batch -b batch.conf
```

## Evaluation signals

- The aggregated table contains 17 metrics across three QC layers for every sample and dataset, matching the structure of Additional file 2: Table S1 (reference atlas: 23,677 samples, 11,265 datasets, eight categories)
- All non-skippable steps completed: mapping (step 2) and merged peak calling (step 6) BAM/peak outputs exist, since these 'cannot be skipped'
- Per-sample JSON files are present for each emitted metric prefix (id_fastqc., id_map., id_frag., id_pbc., id_pbc., id_frip., id_macs2., id_rep.)
- FRiP scores and PBC values are finite and recorded for each sub-sampled BAM; fragment-size estimation is skipped correctly for PBC when step 3 already ran
- QC reports include comparisons against the historical atlas (23,677 public samples), so each sample's metrics can be benchmarked against historical distributions

## Limitations

- MDSeqPos motif analysis (step 12) is automatically skipped when peak number is less than 200
- Steps 1 (FastQC), 10 (PhastCons), and 11 (regulatory potential) are optional and can be skipped, producing partial metric coverage
- PBC evaluation and reads-ratio steps are skipped if step 3 (fragment-size estimation) already sub-sampled the BAM files
- No software version is stated for ChiLin in the discussion, complicating exact environment reproduction
- The reproducibility claim is not backed by concrete environment specification or output artifacts in the article text

## Evidence

- [methods] step 2: bwa(default), bowtie or star mapping, this step cannot be skipped, because this provided necessary BAM files.: "step 2: bwa(default), bowtie or star mapping, this step cannot be skipped, because this provided necessary BAM files."
- [methods] step 7: calculate FRiP scores for each sample.: "step 7: calculate FRiP scores for each sample."
- [methods] step 5: call peak for replicates samples, and do replicates peak overlap/correlation analysis: "step 5: call peak for replicates samples, and do replicates peak overlap/correlation analysis"
- [abstract] We have successfully compiled 23,677 profiles into a comprehensive quality atlas with fine classification for users.: "We have successfully compiled 23,677 profiles into a comprehensive quality atlas with fine classification for users."
- [readme] conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0"
- [methods] use MDSeqPos to perform motif analysis, for peaks number less than 200, ChiLin skip this step automatically: "use MDSeqPos to perform motif analysis, for peaks number less than 200, ChiLin skip this step automatically"
