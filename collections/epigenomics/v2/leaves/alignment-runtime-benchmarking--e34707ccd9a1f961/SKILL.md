---
name: alignment-runtime-benchmarking
description: Use when comparing alignment runtimes of bisulfite sequencing aligners
  across read volumes (10M/50M/100M/200M reads) and cluster sizes, or when validating
  scale-out Spark aligners against MapReduce, MPI, or single-server baselines.
when_to_use_negative:
- Input is not bisulfite-treated sequencing data — the 'three-letter' CT/GA conversion
  and non-directional library mode assume WGBS/BS-seq reads
- Goal is accuracy/sensitivity assessment rather than runtime — use the Sherman simulation
  (human chromosome 1, 95bp, 1,000,000 reads) benchmark workflow instead
- Only a single-node, single-dataset alignment is needed — the distributed benchmark
  overhead (HDFS, multi-node setup) is unnecessary
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: BiSpark
  role: Spark-based 'three-letter' bisulfite aligner under test, run in plain and
    balancing (portable_hash) modes via build_index and alignment scripts
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed framework (>= 1.6) over which BiSpark is implemented; supplies
    portable_hash for the load-balancing mode
  repo: https://spark.apache.org/
- name: Bowtie2
  role: Underlying short-read aligner used by BiSpark's build_index step; must be
    callable on all slave nodes
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: CloudAligner
  role: Hadoop MapReduce baseline (v1.8) for runtime comparison
- name: Bison
  role: MPI baseline (v0.3.3) requiring 21 nodes for a 20-node cluster per the 4((N-1)/4)+1
    node restriction
- name: Bismark
  role: Single-server scale-up baseline (v0.18.1, 5-multicore) on a 24-core machine
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:98f663be0f1e6d21c99f8d2e8c328e49888e899b75f109db4814555e52b7a981
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# alignment-runtime-benchmarking

## Summary

Benchmark the runtime and scalability of distributed bisulfite read aligners by running BiSpark (plain and hash-balance modes) against CloudAligner, Bison, and Bismark on WGBS data (GSE80911) across read volumes (10M–200M) and cluster sizes (10/20/40 nodes), recording alignment seconds and reads-per-second throughput.

## When to use

Use when comparing alignment runtimes of bisulfite sequencing aligners across read volumes (10M/50M/100M/200M reads) and cluster sizes, or when validating scale-out Spark aligners against MapReduce, MPI, or single-server baselines.

## When NOT to use

- Input is not bisulfite-treated sequencing data — the 'three-letter' CT/GA conversion and non-directional library mode assume WGBS/BS-seq reads
- Goal is accuracy/sensitivity assessment rather than runtime — use the Sherman simulation (human chromosome 1, 95bp, 1,000,000 reads) benchmark workflow instead
- Only a single-node, single-dataset alignment is needed — the distributed benchmark overhead (HDFS, multi-node setup) is unnecessary

## Inputs

- WGBS FASTQ reads from GEO GSE80911 (Illumina HiSeq 2500, 95 bp) as 10M/50M/100M/200M read test sets
- Human reference genome Build 37 (hg19) FASTA in HDFS
- Spark cluster with 10/20/40 slave nodes; 24-core single server for Bismark

## Outputs

- alignment_runtime_comparison.csv with per-aligner elapsed seconds for 10M and 200M reads
- Throughput (reads/second) scalability table for 10/20/40 slave nodes on 100M-read input

## How to apply

Download the WGBS FASTQ data for GEO accession GSE80911 (Illumina HiSeq 2500, 95 bp reads) and construct test sets of 10M, 50M, 100M, and 200M reads per Table 1. Prepare the hg19 (Build 37) chromosome 1 reference with the BiSpark build_index step, which performs the four CT/GA conversions of the 'three-letter' algorithm, indexes with Bowtie2, and stores the result in HDFS. Run BiSpark-plain on the 20-node Spark cluster in non-directional library mode for each data size (excluding reference pre-processing time from measured elapsed time), then repeat with BiSpark-balance (portable_hash partitioning enabled via --testmode balancing). Run CloudAligner v1.8 (Hadoop MapReduce), Bison v0.3.3 (MPI, 21 nodes for the 20-cluster under the 4((N-1)/4)+1 node restriction), and Bismark v0.18.1 with the 5-multicore option on the same datasets. For cluster-size scaling, compute throughput as processed reads per second on 100M-read input over 10/20/40 slave nodes and assemble all runtimes into a summary table mirroring Fig. 2a and 2b.

## Related tools

- **BiSpark** (Spark-based 'three-letter' bisulfite aligner under test, run in plain and balancing (portable_hash) modes via build_index and alignment scripts) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed framework (>= 1.6) over which BiSpark is implemented; supplies portable_hash for the load-balancing mode) — https://spark.apache.org/
- **Bowtie2** (Underlying short-read aligner used by BiSpark's build_index step; must be callable on all slave nodes) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **CloudAligner** (Hadoop MapReduce baseline (v1.8) for runtime comparison)
- **Bison** (MPI baseline (v0.3.3) requiring 21 nodes for a 20-node cluster per the 4((N-1)/4)+1 node restriction)
- **Bismark** (Single-server scale-up baseline (v0.18.1, 5-multicore) on a 24-core machine)

## Examples

```
./src/alignment/run.sh --input /hdfs/GSE80911_10M.fastq --ref /hdfs/hg19_chr1_index --nodes 20 --num_mm 4 --testmode plain --local_save out.sam
```

## Evaluation signals

- BiSpark-plain on 10M reads with 20 nodes takes approximately 617 s, versus CloudAligner's 14,783 s (over 20x speedup)
- BiSpark-plain is around 2.5x faster than Bismark at 10M reads and 3.5x at 200M reads
- BiSpark-plain achieves roughly 22% faster alignment than Bison (v0.3.3) on 200M reads
- All measured runtimes recorded in alignment_runtime_comparison.csv, with reference pre-processing (build_index) time excluded from elapsed-time measurements
- Throughput (reads/second) for 10/20/40 slave nodes on 100M reads scales near-linearly, matching Fig. 2a/2b trends

## Limitations

- Trade-off between load balancing and shuffling makes it impossible to conclude theoretically whether BiSpark-balance improves performance; the reason it is faster on large clusters is speculative
- No software version numbers are stated for BiSpark, Spark, or HDFS, complicating reproducibility
- Bison's MPI implementation imposes a node-count restriction (4((N-1)/4)+1), requiring 21 nodes to match a 20-node comparison cluster
- Reference genome conversion and indexing are performed once on the master node before moving to HDFS, so pre-processing time must be excluded to keep runtime comparisons fair

## Evidence

- [methods] BiSpark-plain took 617 s on 10M reads, more than 20 times faster than CloudAligner (14,783 s): "BiSpark-plain only took 617 s and this is around more than 20 times faster than CloudAligner that took 14,783 s"
- [methods] 22% improvement over Bison on 200M reads: "the BiSpark-plain achieved around 22% performance improvement on 200M reads"
- [methods] WGBS dataset and accession: "a whole genome bisulfite sequencing (WGBS) dataset obtained from Gene Expression Omnibus (GEO) repository whose series accession number is GSE80911"
- [methods] Reference preparation in HDFS: "all four reference genome conversion and indexing are performed in the master node first and moved to HDFS for multi-node sharing"
- [discussion] Load balancing trade-off: "Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not."
- [readme] BiSpark alignment script parameters: "--nodes: The number of executors. = (# of nodes of cluster) * (# of spark executors in each node)"
- [readme] Testmode switch for benchmark modes: "--testmode: Switching testmode for experiment. Should be one of 'balancing' or 'plain'."
