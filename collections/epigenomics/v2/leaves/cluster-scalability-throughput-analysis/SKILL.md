---
name: cluster-scalability-throughput-analysis
description: Use when you must quantify how a distributed aligner's runtime and throughput
  scale with read volume and number of cluster nodes, compared against alternative
  aligners on the same WGBS dataset.
when_to_use_negative:
- The goal is mapping-quality or methylation-call accuracy comparison rather than
  runtime/scalability benchmarking
- The input is a small dataset where single-node aligners suffice and distributed
  overhead dominates
- The aligner under test is a scale-up (single-server) tool only, with no multi-node
  comparison possible
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: BiSpark
  role: Distributed bisulfite aligner under benchmark (plain and portable_hash load-balanced
    modes)
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed framework on which BiSpark runs; nodes/executors set via --nodes
- name: CloudAligner
  role: Hadoop MapReduce comparator aligner (v1.8)
- name: Bison
  role: MPI comparator aligner (v0.3.3), run on 21 nodes for a 20-node cluster
- name: Bismark
  role: Single-server comparator aligner (v0.18.1) run with the 5-multicore option
    on 24 cores
- name: Bowtie2
  role: Alignment engine used inside BiSpark's build_index/mapping; must be callable
    on all slave nodes
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:1d69ae91cfadc7a5c9f9f56466c1f47d1a608f90e922f756cb8b5346bdc14a45
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
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cluster-scalability-throughput-analysis

## Summary

Benchmark a distributed bisulfite aligner (BiSpark) against CloudAligner, Bison, and Bismark on real WGBS data (GSE80911 vs hg19), measuring alignment runtimes, throughput (reads/second), and scaling across read volumes (10M–200M) and cluster sizes (10/20/40 slave nodes).

## When to use

Use when you must quantify how a distributed aligner's runtime and throughput scale with read volume and number of cluster nodes, compared against alternative aligners on the same WGBS dataset.

## When NOT to use

- The goal is mapping-quality or methylation-call accuracy comparison rather than runtime/scalability benchmarking
- The input is a small dataset where single-node aligners suffice and distributed overhead dominates
- The aligner under test is a scale-up (single-server) tool only, with no multi-node comparison possible

## Inputs

- WGBS FASTQ reads from GEO accession GSE80911 (Illumina HiSeq 2500, 95 bp), subset to 10M/50M/100M/200M reads
- Human reference genome Build 37 (hg19) FASTA for BiSpark build_index
- Spark cluster with 10/20/40 slave nodes and HDFS 2.6
- Single 24-core server for the Bismark comparison

## Outputs

- alignment_runtime_comparison.csv with per-tool, per-dataset, per-cluster-size elapsed times
- Throughput values (processed reads per second) for the cluster-size scaling experiment
- Summary table of speedups (e.g., BiSpark-plain vs CloudAligner, Bismark, Bison) mirroring Fig. 2a/2b

## How to apply

Download the WGBS FASTQ data for GEO accession GSE80911 (Illumina HiSeq 2500, 95 bp reads) and construct test sets of 10M, 50M, 100M, and 200M reads. Build the hg19 (Build 37) index with BiSpark's build_index step, which performs the four CT/GA 'three-letter' conversions, indexes with Bowtie2, and stores the result in HDFS, excluding reference pre-processing time from measured runtimes. Run BiSpark-plain and BiSpark-balance (portable_hash load balancing) on the Spark cluster in non-directional library mode; run CloudAligner v1.8 (Hadoop MapReduce), Bison v0.3.3 (MPI, using 21 nodes for a 20-node cluster per its 4((N-1)/4)+1 restriction), and Bismark v0.18.1 with the 5-multicore option on the same inputs. For the cluster-size scaling experiment, run 100M reads on 10, 20, and 40 slave nodes and compute throughput as processed reads per second. Record all runtimes in a summary table mirroring the scalabilities of Fig. 2a and 2b.

## Related tools

- **BiSpark** (Distributed bisulfite aligner under benchmark (plain and portable_hash load-balanced modes)) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed framework on which BiSpark runs; nodes/executors set via --nodes)
- **CloudAligner** (Hadoop MapReduce comparator aligner (v1.8))
- **Bison** (MPI comparator aligner (v0.3.3), run on 21 nodes for a 20-node cluster)
- **Bismark** (Single-server comparator aligner (v0.18.1) run with the 5-multicore option on 24 cores)
- **Bowtie2** (Alignment engine used inside BiSpark's build_index/mapping; must be callable on all slave nodes) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml

## Examples

```
./src/alignment/run.sh --input <fastq_in_hdfs> --output <out_dir_hdfs> --ref <index_dir_hdfs> --nodes 20 --num_mm 4 --local_save out.sam --testmode plain
```

## Evaluation signals

- BiSpark-plain on 10M reads with 20 nodes completes in ~617 s versus CloudAligner's 14,783 s (>20x speedup)
- BiSpark-plain is ~2.5x faster than Bismark at 10M reads and ~3.5x at 200M reads
- BiSpark-plain achieves roughly 22% faster alignment than Bison on 200M reads
- Cluster-size scaling at 100M reads (10/20/40 slaves) shows near-linear throughput scaling when reported as reads/second
- Reference pre-processing time is excluded from all reported elapsed times

## Limitations

- Whether BiSpark-balance improves performance is a load-balancing-vs-shuffling trade-off with no theoretical conclusion; it was not resolved empirically
- The reason BiSpark-balance is faster on large clusters remains speculative (more nodes may wait for the slowest node as cluster size grows)
- Bison imposes a node-count restriction (4((N-1)/4)+1 nodes), so a 21-node allocation is needed for a nominal 20-node cluster, complicating fair comparison
- Bismark must be run on a separate 24-core single server, so comparisons mix scale-up and scale-out hardware

## Evidence

- [methods] BiSpark-plain only took 617 s and this is around more than 20 times faster than CloudAligner that took 14,783 s: "BiSpark-plain only took 617 s and this is around more than 20 times faster than CloudAligner that took 14,783 s"
- [methods] Bismark speedups: "BiSpark-plain is around 2.5 times faster than Bismark on 10M reads and 3.5 times faster on 200M reads"
- [methods] Bison comparison: "the BiSpark-plain achieved around 22% performance improvement on 200M reads"
- [methods] Reference conversion and indexing: "all four reference genome conversion and indexing are performed in the master node first and moved to HDFS for multi-node sharing"
- [discussion] Load-balancing trade-off: "Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not."
- [readme] README usage of build_index/alignment: "--nodes: The number of executors. = (# of nodes of cluster) * (# of spark executors in each node)"
