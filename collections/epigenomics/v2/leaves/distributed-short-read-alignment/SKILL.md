---
name: distributed-short-read-alignment
description: Use when aligning bisulfite-treated sequencing reads (e.g., WGBS FASTQ
  from simulated Sherman data or GEO accessions) is computationally bottlenecked on
  a single node and a scale-out Spark/HDFS cluster is available.
when_to_use_negative:
- When the task is methylation array interrogation (Illumina 27K/450K/850K BeadChip)
  rather than bisulfite sequencing read alignment
- When no Spark/HDFS cluster or Spark >= 1.6 / HDFS 2.6 environment is available and
  the dataset fits single-node tools like Bismark
- When reads are not bisulfite-treated; a general-purpose aligner without three-letter/wildcard
  conversion is appropriate instead
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: BiSpark
  role: The Spark-based bisulfite aligner implementing the four-phase distributed
    alignment workflow
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed memory data-processing framework hosting the readRDD/transRDD/mapRDD/filteredRDD/methylRDD
    pipeline
  repo: https://spark.apache.org/
- name: Bowtie2
  role: Aligner invoked on the CT/GA-converted three-letter read variants against
    the four converted reference indexes; must be callable on all slave nodes
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: HDFS
  role: Shared file system holding the input reads, converted reference indexes, and
    outputs so users need not distribute files manually
  repo: https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
- name: Cutadapt
  role: Optional recommended pre-processing to remove adapter sequences before alignment
  repo: https://github.com/marcelm/cutadapt/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:50ae49f91b0853310b23175693fb5da0bf6f83444844ef0da77cd75a353c22d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - Apache Spark
  - BiSpark
  - Bowtie2
  - Cutadapt
  - HDFS
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# distributed-short-read-alignment

## Summary

A scale-out short-read alignment skill that parallelizes bisulfite 'three-letter' read mapping across an Apache Spark cluster (BiSpark), addressing the up-to-4x data expansion that makes bisulfite alignment a bottleneck of methylome analysis.

## When to use

Use when aligning bisulfite-treated sequencing reads (e.g., WGBS FASTQ from simulated Sherman data or GEO accessions) is computationally bottlenecked on a single node and a scale-out Spark/HDFS cluster is available.

## When NOT to use

- When the task is methylation array interrogation (Illumina 27K/450K/850K BeadChip) rather than bisulfite sequencing read alignment
- When no Spark/HDFS cluster or Spark >= 1.6 / HDFS 2.6 environment is available and the dataset fits single-node tools like Bismark
- When reads are not bisulfite-treated; a general-purpose aligner without three-letter/wildcard conversion is appropriate instead

## Inputs

- FASTQ/A bisulfite-treated sequencing reads (e.g., Sherman-simulated human chromosome 1, 95 bp, 1,000,000 reads, 0% error)
- Reference genome FASTA in HDFS (e.g., human Build 37 / hg19)
- Four converted and indexed reference genomes (CT/GA conversions for Bowtie2), built in the master node and shared via HDFS

## Outputs

- Aligned reads in SAM format (--local_save output)
- Per-site methylation estimates (methylRDD)
- Intermediate Spark RDDs serialized as artifacts: readRDD, transRDD, mapRDD, filteredRDD

## How to apply

Run BiSpark as four Spark phases: (1) load FASTQ/A reads into key-value readRDD tuples keyed by read identifier (with optional prior Cutadapt adapter trimming and quality-based discarding of poor reads); (2) transform each read into CT- and GA-converted 'three-letter' variants (transRDD split into CTtransRDD and GAtransRDD) and align each variant with Bowtie2 against four correspondingly converted and indexed reference genomes (e.g., hg19 Build 37 chromosome 1), yielding W-CT, W-GA, C-CT and C-GA mapRDDs; (3) union the four mapRDDs shuffle-free at the Spark context level, group by read id into combRDD, keep the uniquely smallest-mismatch alignment, and discard ambiguous multi-best hits and alignments above a user-defined mismatch cutoff (default 4) to obtain filteredRDD; (4) merge sequences and mapping information into mergedRDD and compare filteredRDD alignments against the broadcast original reference genome to estimate per-site methylation into methylRDD, written out in SAM format. Choose --testmode 'balancing' (portable_hash partitioning) or 'plain'; load balancing trades against shuffling cost, and the paper found no theoretical or uniform empirical winner, though BiSpark-balance was faster on large clusters.

## Related tools

- **BiSpark** (The Spark-based bisulfite aligner implementing the four-phase distributed alignment workflow) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed memory data-processing framework hosting the readRDD/transRDD/mapRDD/filteredRDD/methylRDD pipeline) — https://spark.apache.org/
- **Bowtie2** (Aligner invoked on the CT/GA-converted three-letter read variants against the four converted reference indexes; must be callable on all slave nodes) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **HDFS** (Shared file system holding the input reads, converted reference indexes, and outputs so users need not distribute files manually) — https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
- **Cutadapt** (Optional recommended pre-processing to remove adapter sequences before alignment) — https://github.com/marcelm/cutadapt/

## Examples

```
sh ./src/alignment/run.sh --input reads.fastq --ref hdfs:///index/hg19_chr1 --nodes 4 --num_mm 4 --testmode plain --local_save out.sam
```

## Evaluation signals

- On the 0%-error Sherman simulation (chr1, 95 bp, 1M reads), mappability should reach 0.9569 with precision 1.0, identical for BiSpark-plain and BiSpark-balance
- filteredRDD must contain only uniquely-best alignments: ambiguous multi-best hits and alignments exceeding the mismatch cutoff (default 4) are removed
- All four mapRDDs (W-CT, W-GA, C-CT, C-GA) are combined via a shuffle-free Spark context-level union
- Final methylRDD is emitted as a well-formed SAM file at --local_save
- Speed sanity check: BiSpark-plain on 10M reads took 617 s (vs. 14,783 s for CloudAligner); large deviations on comparable hardware warrant investigation

## Limitations

- Load balancing (portable_hash) and shuffling are a speed trade-off; the paper could not conclude theoretically when BiSpark-balance improves performance, and its speedup on big clusters remains speculative
- BiSpark-balance and BiSpark-plain give identical mapping results, so balancing only affects speed/scalability, not alignment quality
- Version numbers for BiSpark, Spark, and HDFS are not pinned by the paper; README requires Python 2.6/2.7, Spark >= 1.6, HDFS 2.6, and Bowtie2 callable on every slave node
- Poor input read quality degrades results; Cutadapt and FastQC pre-processing are only recommended, not enforced

## Evidence

- [readme] Analysis workflow within BiSpark consists of 4 processing phases: (1) Distributing the reads into key-value pairs, (2) Transforming reads into ‘three-letter’ reads and mapping: "Analysis workflow within BiSpark consists of 4 processing phases: (1) Distributing the reads into key-value pairs, (2) Transforming reads into ‘three-letter’ reads and mapping"
- [methods] the BiSpark also supports a user-defined mismatch cutoff to adjust the intensity of the restriction: "the BiSpark also supports a user-defined mismatch cutoff to adjust the intensity of the restriction"
- [methods] BiSpark-plain only took 617 s and this is around more than 20 times faster than CloudAligner that took 14,783 s: "BiSpark-plain only took 617 s and this is around more than 20 times faster than CloudAligner that took 14,783 s"
- [discussion] Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved: "Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not."
- [readme] Run Cutadapt (https://github.com/marcelm/cutadapt/) to remove adaptor sequence from the reads.: "Run Cutadapt (https://github.com/marcelm/cutadapt/) to remove adaptor sequence from the reads."
