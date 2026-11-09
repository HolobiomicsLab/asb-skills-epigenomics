---
name: sequencing-depth-downsampling-simulation
description: Decision support for whether a ChIP-seq/DNase-seq QC metric like FRiP
  is a genuine quality readout or merely an artifact of sequencing depth, especially
  when benchmarking shallow libraries or choosing downsampling conventions.
when_to_use_negative:
- Input data is not ChIP-seq/DNase-seq with callable peaks — FRiP is undefined without
  a peak set
- The goal is to measure absolute library quality rather than depth-dependence of
  a QC metric; ChiLin's standard one-shot FRiP step suffices
- Peak regions are not comparable across samples (e.g. mixing broad and narrow peak
  types, when MACS2 narrow is the ChiLin TF default)
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3169
tools:
- name: BWA
  role: Maps raw FASTQ files to hg19 to produce the sorted BAM files that are subsequently
    down-sampled
  repo: https://github.com/lh3/bwa
- name: seqtk
  role: Sub-sampling utility with a fixed random seed; usable to down-sample FASTQ
    reads before mapping at controlled depths
  repo: https://github.com/lh3/seqtk
- name: MACS2
  role: Narrow peak caller used both per down-sampled BAM (Setting A) and once on
    the full-depth BAM (Setting B)
  repo: https://github.com/taoliu/MACS
- name: ChiLin
  role: Defines the sub-sampling convention (sub-sample BAM files for macs2 fragment
    size estimation) and computes FRiP scores (step 7, id_frip./*.frip outputs)
  repo: https://github.com/cfce/chilin
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:0a126b179980940d89ee84801afe648a03bea54f2ab424f5153bd582f62c73e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/bwa
  tools:
  - BWA
  - ChiLin
  - MACS2
  - seqtk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# sequencing-depth-downsampling-simulation

## Summary

Simulates sequencing-depth variation by sub-sampling BAM (or FASTQ) files at stepwise depths and comparing QC metrics — such as FRiP scores — computed against peaks called per-depth versus peaks called once from the full-depth BAM, exposing how metric stability depends on peak-calling strategy.

## When to use

Decision support for whether a ChIP-seq/DNase-seq QC metric like FRiP is a genuine quality readout or merely an artifact of sequencing depth, especially when benchmarking shallow libraries or choosing downsampling conventions.

## When NOT to use

- Input data is not ChIP-seq/DNase-seq with callable peaks — FRiP is undefined without a peak set
- The goal is to measure absolute library quality rather than depth-dependence of a QC metric; ChiLin's standard one-shot FRiP step suffices
- Peak regions are not comparable across samples (e.g. mixing broad and narrow peak types, when MACS2 narrow is the ChiLin TF default)

## Inputs

- Raw FASTQ files of selected low/mid/high-quality ChIP-seq or DNase-seq samples
- Sorted BAM files aligned to hg19 (BWA mapping)
- Sub-sampled BAM files at stepwise depths (e.g. 1M, ..., 4M reads, ChiLin convention)
- MACS2 narrow-peak BED files from Setting A (per-depth) and Setting B (full-depth, fixed)

## Outputs

- FRiP score table per sample per depth per setting (id_frip. / *.frip style outputs)
- Stability curves of FRiP versus sequencing depth per quality class, overlaying Setting A and Setting B

## How to apply

Map raw FASTQ files to a reference (e.g. hg19) with BWA to obtain sorted BAM files, then sub-sample each BAM stepwise starting at 1M reads, following the ChiLin sub-sampling convention (the 4M-read subsampled BAMs it produces in its fragment-size-estimation step). Run two settings: Setting A calls narrow peaks with MACS2 independently on each down-sampled BAM; Setting B calls peaks once on the full-depth BAM and holds those peak regions fixed across all depths. Compute FRiP scores at each depth under both settings (fraction of reads falling in peaks, as in ChiLin's step 7, emitted in the style of id_frip./*.frip outputs). Rationale: reads from deeper sequencing will trivially fall inside a fixed peak set, so Setting B isolates depth effects on FRiP that Setting A conflates with peak-set shrinkage. Plot FRiP versus depth as stability curves per sample quality class (low/mid/high), overlaying both settings.

## Related tools

- **BWA** (Maps raw FASTQ files to hg19 to produce the sorted BAM files that are subsequently down-sampled) — https://github.com/lh3/bwa
- **seqtk** (Sub-sampling utility with a fixed random seed; usable to down-sample FASTQ reads before mapping at controlled depths) — https://github.com/lh3/seqtk
- **MACS2** (Narrow peak caller used both per down-sampled BAM (Setting A) and once on the full-depth BAM (Setting B)) — https://github.com/taoliu/MACS
- **ChiLin** (Defines the sub-sampling convention (sub-sample BAM files for macs2 fragment size estimation) and computes FRiP scores (step 7, id_frip./*.frip outputs)) — https://github.com/cfce/chilin

## Examples

```
seqtk sample -s100 in.fq.gz 1000000 > sub_1M.fq
```

## Evaluation signals

- Setting B FRiP curves are approximately flat across sequencing depths for all quality classes (reproducing Fig. 2e)
- Setting A FRiP degrades at shallow depths relative to Setting B (reproducing Fig. 2d), confirming the peak-set artifact
- Sub-sampled BAMs contain the expected read counts at each stepwise depth (e.g. 1M through 4M per the ChiLin convention)
- FRiP values lie in [0, 1] and equal the fraction of reads falling inside the held peak regions when recomputed with bedtools/ChiLin step 7
- Full-depth Setting B FRiP matches the standard one-shot FRiP from ChiLin step 7 for the same sample

## Limitations

- The article states reproducibility as a claim but provides no concrete commands, environment specification, or output artifacts in the discussion section, so exact reproduction requires assembling the pipeline from ChiLin/MACS2/BWA/seqtk documentation
- No software version is stated for ChiLin, so sub-sampling conventions may drift across versions
- MACS2 peak calling on very shallow sub-samples may fail or produce unstable fragment-size models, degrading Setting A at exactly the depths of interest

## Evidence

- [methods] step 3: sub-sample bam files and do macs2 fragment size estimation.: "step 3: sub-sample bam files and do macs2 fragment size estimation."
- [methods] step 7: calculate FRiP scores for each sample.: "step 7: calculate FRiP scores for each sample."
- [intro] ChiLin uses MACS2 as the default peak caller: "ChiLin uses MACS2 (https://github.com/taoliu/MACS/) as the default peak caller"
- [readme] Subsample read pairs from FASTQ with seqtk using a fixed seed: [evidence span withheld — anchor status: external_doc]
- [methods] The full-depth mapping step produces the BAM files that feed all downstream QC: "step 2: bwa(default), bowtie or star mapping, this step cannot be skipped, because this provided necessary BAM files."
