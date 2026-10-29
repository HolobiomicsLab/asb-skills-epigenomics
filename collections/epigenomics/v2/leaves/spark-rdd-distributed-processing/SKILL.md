---
name: spark-rdd-distributed-processing
description: Use when aligning bisulfite-sequencing (WGBS) reads with the 'three-letter'
  approach where data volume (up to 4× genome expansion) makes single-node aligners
  like Bismark or CloudAligner too slow, and a Spark cluster with HDFS is available.
when_to_use_negative:
- When the analysis is not bisulfite-treated sequencing alignment (e.g., standard
  DNA-seq or RNA-seq mapping) — the four-genome 'three-letter' expansion and CT/GA
  transformations are specific to BS-seq.
- When no Spark cluster or HDFS is available and the dataset is small enough for a
  single-node aligner like Bismark.
- When strict methylation profiling is needed without access to the original reference
  genome sequence for comparing filteredRDD reads back.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: BiSpark
  role: The aligner implementing the Spark RDD pipeline (readRDD, transRDD, mapRDD,
    combRDD, filteredRDD) for bisulfite read alignment
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed data processing framework over which BiSpark's key-value RDD transformations
    and hash-partitioned load balancing are implemented
- name: Bowtie2
  role: Read alignment engine invoked on each 'three-letter' transformed read set
    against its corresponding customized genome index
- name: HDFS
  role: Distributed file system used to stage indexed reference genomes and share
    them across executors
- name: Sherman
  role: Simulator used to generate bisulfite FASTQ reads with known original positions
    for computing mappability, sensitivity, precision, and accuracy
- name: Cutadapt
  role: Optional recommended preprocessing to remove adapter sequence from reads before
    the main BiSpark phases
  repo: https://github.com/marcelm/cutadapt/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:a8d9b17eb23de9ff81c8ff687e7e23d83495f7a7e7ef479944491ed2ad079ed6
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
  - Sherman
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spark-rdd-distributed-processing

## Summary

Design a bisulfite sequencing read alignment workflow as a chain of Spark RDD transformations (readRDD → transRDD → mapRDD → combRDD → filteredRDD), so that the compute-heavy 'three-letter' mapping scales out across a cluster instead of running single-node. This skill reproduces the BiSpark approach of expressing alignment as parallelizable key-value operations on Apache Spark with HDFS-staged reference genomes.

## When to use

Use when aligning bisulfite-sequencing (WGBS) reads with the 'three-letter' approach where data volume (up to 4× genome expansion) makes single-node aligners like Bismark or CloudAligner too slow, and a Spark cluster with HDFS is available.

## When NOT to use

- When the analysis is not bisulfite-treated sequencing alignment (e.g., standard DNA-seq or RNA-seq mapping) — the four-genome 'three-letter' expansion and CT/GA transformations are specific to BS-seq.
- When no Spark cluster or HDFS is available and the dataset is small enough for a single-node aligner like Bismark.
- When strict methylation profiling is needed without access to the original reference genome sequence for comparing filteredRDD reads back.

## Inputs

- FASTQ/FASTA raw bisulfite-treated sequencing reads (e.g., 95bp reads, 1,000,000 reads simulated on human chromosome 1 with Sherman at 0%, 1%, 2% error rates)
- Reference genome FASTA (human Build 37, hg19), staged in HDFS
- Bowtie2 index directories for the four 'three-letter' genomes (W-CT, W-GA, C-CT, C-GA)

## Outputs

- SAM file of aligned reads (local_save path) with best alignments per read id
- filteredRDD of unambiguous, best-alignment reads
- Downstream methylRDD methylation calls (Phase 4 output of the BiSpark workflow)
- Alignment-quality metrics per error rate: mappability (mapped/all reads), precision TP/(TP+FP), sensitivity TP/(TP+FN), accuracy TP/(TP+FP+FN)

## How to apply

Convert the reference (e.g., hg19 chromosome 1 FASTA) into four 'three-letter' genomes (W-CT, W-GA, C-CT, C-GA), index them with Bowtie2 in the master node, and stage the indexed genomes in HDFS for multi-node sharing. Load FASTQ reads into a readRDD of <read id, sequence> key-value tuples, transform into CT and GA transRDDs, and align each against the corresponding customized genome with Bowtie2 under non-directional library mode, respecting a user-defined mismatch cutoff (default 4). Union the resulting four mapRDDs without shuffling, combine by read id into combRDD, and keep only reads with a uniquely smallest mismatch count as the best alignment, discarding ambiguous alignments to form filteredRDD. For large clusters, apply Spark's portable_hash partitioning to balance load across executors, trading extra shuffling for even distribution.

## Related tools

- **BiSpark** (The aligner implementing the Spark RDD pipeline (readRDD, transRDD, mapRDD, combRDD, filteredRDD) for bisulfite read alignment) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed data processing framework over which BiSpark's key-value RDD transformations and hash-partitioned load balancing are implemented)
- **Bowtie2** (Read alignment engine invoked on each 'three-letter' transformed read set against its corresponding customized genome index)
- **HDFS** (Distributed file system used to stage indexed reference genomes and share them across executors)
- **Sherman** (Simulator used to generate bisulfite FASTQ reads with known original positions for computing mappability, sensitivity, precision, and accuracy)
- **Cutadapt** (Optional recommended preprocessing to remove adapter sequence from reads before the main BiSpark phases) — https://github.com/marcelm/cutadapt/

## Examples

```
./src/alignment/run.sh --input <HDFS_FASTQ> --output <HDFS_OUT> --ref <HDFS_IDX> --nodes 4 --num_mm 4 --local_save aligned.sam --testmode plain --appname BiSpark
```

## Evaluation signals

- Reproduce the paper's Table 3 BiSpark metrics on Sherman-simulated chromosome 1 data: 0% error → mappability/sensitivity 0.9569, precision 1.0, accuracy 0.9569; 1% error → mappability 0.9494, sensitivity 0.9489, accuracy 0.9392; 2% error → mappability 0.9422, sensitivity 0.9411, accuracy 0.9234.
- Verify BiSpark-plain and BiSpark-balance produce identical alignment metrics (differences should only be in runtime, not mapping quality).
- Confirm the four mapRDDs correspond to the four customized genomes (W-CT, W-GA, C-CT, C-GA) and are unioned without a shuffle.
- Check that combRDD aggregates by read id and filteredRDD contains only reads with a uniquely smallest mismatch count (≤ the user-defined cutoff, default 4).
- Compare runtime against single-node baselines: on 10M reads BiSpark-plain should take on the order of hundreds of seconds (617 s reported), not the ~14,783 s CloudAligner requires.

## Limitations

- Load balancing via portable_hash causes shuffling, and the trade-off between balancing and shuffling speed is not theoretically resolved — balance mode may not always improve performance.
- BiSpark-balance's speedup on large clusters is speculative (attributed to more nodes waiting on the slowest node) and not definitively established.
- Requires Bowtie2 callable on every slave node, plus matching Spark (≥1.6) and HDFS (2.6) versions; Python 2.6/2.7 only.
- Ambiguous alignments (no uniquely best hit) are discarded, which can reduce mappability on repetitive or high-error datasets.

## Evidence

- [readme] Four-phase Spark RDD pipeline description: "BiSpark is designed based on the Apache Spark distributed framework and shows highly efficient scalability."
- [methods] Distributed key-value RDD structure: "the BiSpark accepts raw sequencing data files, FASTQ/A format, as inputs and converts them into list of key-value structured tuples"
- [methods] Three-letter transform step: "the BiSpark transforms readRDDs to transRDD which consists of <read id, transformed read sequence > tuples"
- [methods] Best alignment and ambiguity filtering: "the next step of the analysis is figuring out the best alignment among these results"
- [methods] Load balancing via hash partitioning: "the BiSpark utilizes portable_hash function, supported by Spark framework, to determine which partition the data should be placed"
- [methods] Speed advantage over CloudAligner: "BiSpark-plain only took 617 s and this is around more than 20 times faster than CloudAligner that took 14,783 s"
