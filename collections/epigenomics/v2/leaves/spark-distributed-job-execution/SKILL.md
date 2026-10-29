---
name: spark-distributed-job-execution
description: Use when aligning large WGBS bisulfite sequencing datasets (tens to hundreds
  of millions of 95 bp reads, e.g. FASTQ from GSE80911) where single-server aligners
  like Bismark are too slow and a Spark/HDFS cluster is available.
when_to_use_negative:
- Input is small-scale or a methylation array (e.g. Infinium 27K/450K/850K BeadChip
  data) rather than high-volume bisulfite sequencing reads
- No Spark >=1.6 / HDFS 2.6 cluster or Bowtie2 on all slave nodes is available — single-node
  aligners like Bismark with multicore (-5) are the fallback
- Directional library-specific workflows or tools requiring features BiSpark lacks
  (user should verify functionality coverage against Bismark/BSSeeker alternatives)
edam_operation: http://edamontology.org/operation_3186
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: BiSpark
  role: The Spark-based bisulfite aligner whose build_index and alignment scripts
    are executed as the distributed job
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed data processing framework hosting the BiSpark RDD pipeline (key-value
    distribution, three-letter mapping, filtering, methylation profiling)
  repo: https://spark.apache.org/
- name: Bowtie2
  role: Underlying short-read aligner used by BiSpark on the converted 'three-letter'
    reference; must be callable on all slave nodes
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: Cutadapt
  role: Optional pre-processing step run before the Spark job to remove adaptor sequences
    from reads
  repo: https://github.com/marcelm/cutadapt/
- name: CloudAligner
  role: Hadoop MapReduce comparison baseline (v1.8) used to benchmark the Spark job's
    runtime
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:e1ddd8d28abe94715cb0171990b117efc3213243331477c1679d3292fb2aa821
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - Apache Spark
  - BiSpark
  - Bismark
  - Bison
  - Bowtie2
  - CloudAligner
  - Cutadapt
  edam_operation: http://edamontology.org/operation_3186
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# spark-distributed-job-execution

## Summary

Executes the BiSpark bisulfite aligner as a distributed Apache Spark job over HDFS, scaling 'three-letter' bisulfite read alignment across multi-node clusters (10–40 slaves) instead of single-node scale-up tools like Bismark.

## When to use

Use when aligning large WGBS bisulfite sequencing datasets (tens to hundreds of millions of 95 bp reads, e.g. FASTQ from GSE80911) where single-server aligners like Bismark are too slow and a Spark/HDFS cluster is available.

## When NOT to use

- Input is small-scale or a methylation array (e.g. Infinium 27K/450K/850K BeadChip data) rather than high-volume bisulfite sequencing reads
- No Spark >=1.6 / HDFS 2.6 cluster or Bowtie2 on all slave nodes is available — single-node aligners like Bismark with multicore (-5) are the fallback
- Directional library-specific workflows or tools requiring features BiSpark lacks (user should verify functionality coverage against Bismark/BSSeeker alternatives)

## Inputs

- WGBS FASTQ/FASTA read files (e.g. Illumina HiSeq 2500, 95 bp reads, GSE80911) placed in HDFS
- Reference genome FASTA (human Build 37 / hg19) in HDFS
- Cluster configuration: number of nodes, Spark executors per node

## Outputs

- Indexed bisulfite-converted reference (four CT/GA 'three-letter' conversions, Bowtie2 index) in HDFS
- Alignment SAM file (via --local_save) plus methylation-profiled alignments per read

## How to apply

Install Apache Spark (>=1.6), HDFS (2.6), and Bowtie2 (callable on all slave nodes), plus Python 2.6/2.7 as required by the BiSpark README. First run the build_index step on the master node: it performs the four CT/GA conversions of the 'three-letter' algorithm on the hg19 (Build 37) FASTA, indexes with Bowtie2, and stores the result in HDFS for multi-node sharing; exclude this pre-processing time from measured alignment runtimes. Then run the alignment job (./src/alignment/run.sh) with --ref pointing at the HDFS index, --nodes set to (# of nodes) * (# of Spark executors per node), and --num_mm for the user-defined mismatch cutoff (default 4); choose --testmode plain or balancing (portable_hash partitioning) — BiSpark-plain was benchmarked in non-directional library mode on a 20-node cluster. Optionally pre-process with Cutadapt for adapter removal and FastQC before alignment to improve mappability. Rationale: distributing reads into key-value RDD tuples and mapping 'three-letter' reads in parallel scales almost linearly, so 10M reads align in ~617 s on 20 nodes versus 14,783 s for CloudAligner.

## Related tools

- **BiSpark** (The Spark-based bisulfite aligner whose build_index and alignment scripts are executed as the distributed job) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed data processing framework hosting the BiSpark RDD pipeline (key-value distribution, three-letter mapping, filtering, methylation profiling)) — https://spark.apache.org/
- **Bowtie2** (Underlying short-read aligner used by BiSpark on the converted 'three-letter' reference; must be callable on all slave nodes) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **Cutadapt** (Optional pre-processing step run before the Spark job to remove adaptor sequences from reads) — https://github.com/marcelm/cutadapt/
- **CloudAligner** (Hadoop MapReduce comparison baseline (v1.8) used to benchmark the Spark job's runtime)

## Examples

```
bash ./src/alignment/run.sh --input reads.fastq --ref hdfs:///index/hg19_bisulfite --nodes 20 --num_mm 4 --testmode plain --local_save aligned.sam
```

## Evaluation signals

- BiSpark-plain on 10M reads with 20 nodes completes in approximately 617 s versus CloudAligner's 14,783 s (>20x speedup)
- BiSpark-plain is ~2.5x faster than Bismark at 10M reads and ~3.5x faster at 200M reads
- BiSpark-plain achieves roughly 22% performance improvement over Bison (v0.3.3, MPI, 21 nodes) at 200M reads
- Throughput (reads/second) scales near-linearly in the cluster-size experiment (10/20/40 slave nodes, 100M-read input)
- Mapping results remain consistent and comparable to reference aligners, with best performance on all four metrics (mappability, precision, sensitivity, accuracy) on the 0% error dataset

## Limitations

- Load balancing (BiSpark-balance via portable_hash) trades off against shuffling overhead; no theoretical or conclusive empirical determination of when it improves performance
- Any speedup of BiSpark-balance on large clusters is speculative — larger clusters wait longer for the slowest node
- No version numbers are stated for BiSpark, Spark, or HDFS, and reference genome preparation (four conversions + Bowtie2 indexing) runs on the master node first and its time is excluded from alignment benchmarks
- Bison (MPI) requires a node-count restriction of 4((N-1)/4)+1 nodes (21 nodes for a 20-node cluster comparison)

## Evidence

- [readme] requirements from README: "Python: 2.6, 2.7 * Apache Spark: >= 1.6 * HDFS: 2.6"
- [methods] index sharing: "all four reference genome conversion and indexing are performed in the master node first and moved to HDFS for multi-node sharing"
- [methods] benchmark result: "BiSpark-plain only took 617 s and this is around more than 20 times faster than CloudAligner that took 14,783 s"
- [discussion] load balancing trade-off: "Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not."
- [readme] workflow phases: "Analysis workflow within BiSpark consists of 4 processing phases: (1) Distributing the reads into key-value pairs, (2) Transforming reads into ‘three-letter’ reads"
