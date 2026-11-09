---
name: distributed-cluster-throughput-measurement
description: 'Use when benchmarking a Spark/HDFS-based distributed aligner''s scalability:
  you need per-cluster-size throughput for two algorithmic variants (e.g. --testmode
  plain vs balancing) on the same dataset.'
when_to_use_negative:
- Benchmarking mappability, sensitivity, or alignment accuracy rather than speed/scalability
  — this skill times alignment only and does not compare mapping outputs against truth
  sets
- Single-node or scale-up performance testing — the skill requires a multi-node scale-out
  cluster (10/20/40 slave nodes) served by HDFS
- End-to-end pipeline timing where index building and input distribution must be included
  — those stages are deliberately excluded from the measured elapsed time here
tools:
- name: BiSpark
  role: Distributed bisulfite aligner executed in plain and balancing test modes;
    provides --testmode and --nodes parameters for the cluster-size throughput runs
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed execution framework whose portable_hash repartitioning is toggled
    to create the load-balanced variant being benchmarked
  repo: https://spark.apache.org/
- name: HDFS
  role: Distributed file system hosting the input reads and hg19 chromosome 1 index
    so data are shared without user-managed distribution
  repo: https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
- name: Bowtie2
  role: Alignment engine invoked by BiSpark for the 'three-letter' mapping; must be
    callable on all slave nodes so throughput reflects the full cluster
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:c76816d7e8a6de45af19e10e33a8c1912c86a9ddabf6468a8dfa8fb9b2047504
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - Apache Spark
  - BiSpark
  - Bowtie2
  - HDFS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# distributed-cluster-throughput-measurement

## Summary

Measures alignment throughput (reads per second) of a distributed aligner across increasing cluster sizes (10, 20, 40 slave nodes) under two execution variants (plain vs load-balanced) to quantify the scalability gradient and the load-balancing versus shuffling trade-off.

## When to use

Use when benchmarking a Spark/HDFS-based distributed aligner's scalability: you need per-cluster-size throughput for two algorithmic variants (e.g. --testmode plain vs balancing) on the same dataset.

## When NOT to use

- Benchmarking mappability, sensitivity, or alignment accuracy rather than speed/scalability — this skill times alignment only and does not compare mapping outputs against truth sets
- Single-node or scale-up performance testing — the skill requires a multi-node scale-out cluster (10/20/40 slave nodes) served by HDFS
- End-to-end pipeline timing where index building and input distribution must be included — those stages are deliberately excluded from the measured elapsed time here

## Inputs

- 100-million-read (16 GB) WGBS FASTQ/A dataset (GEO GSE80911) staged in HDFS
- Bowtie2 'three-letter' reference index for hg19 (Build 37) chromosome 1, built on the master node and stored in HDFS
- Cluster configurations of 10, 20, and 40 slave nodes (Apache Spark >= 1.6, HDFS 2.6, Python 2.6/2.7)

## Outputs

- Six-row table with columns: variant (plain/balancing), cluster size, elapsed time, throughput (reads per second)
- Scalability plot of throughput versus number of slave nodes comparing the two variants' gradients

## How to apply

Run the aligner (BiSpark, Bowtie2 'three-letter' alignment, default mismatch cutoff --num_mm 4) on a fixed large input — the 100-million-read (16 GB) WGBS dataset from GEO GSE80911 aligned against the hg19 (Build 37) chromosome 1 index served from HDFS — first in plain mode (--testmode plain) and then in balancing mode (--testmode balancing, portable_hash re-partitioning enabled) on 10-, 20-, and 40-slave-node clusters, setting --nodes to (# of nodes) × (# of Spark executors per node). Exclude reference transformation/indexing and input-distribution pre-processing from the measured elapsed time so only alignment time is timed. For each of the six runs compute throughput as processed reads per second (the inverse-proportional speed metric used for scalability visualization), tabulate variant, cluster size, elapsed time, and throughput, and plot throughput versus number of slave nodes to compare the two variants' gradients. Interpret the result as the load-balancing/shuffling trade-off: BiSpark-balance is relatively slower on small clusters but gains as cluster size grows, plausibly because more nodes must wait for the slowest node in larger clusters.

## Related tools

- **BiSpark** (Distributed bisulfite aligner executed in plain and balancing test modes; provides --testmode and --nodes parameters for the cluster-size throughput runs) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed execution framework whose portable_hash repartitioning is toggled to create the load-balanced variant being benchmarked) — https://spark.apache.org/
- **HDFS** (Distributed file system hosting the input reads and hg19 chromosome 1 index so data are shared without user-managed distribution) — https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
- **Bowtie2** (Alignment engine invoked by BiSpark for the 'three-letter' mapping; must be callable on all slave nodes so throughput reflects the full cluster) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml

## Examples

```
bash ./src/alignment/run.sh --ref hdfs:///hg19_chr1_index --nodes 40 --num_mm 4 --testmode balancing --appname BiSpark-balance-40nodes --local_save out.sam
```

## Evaluation signals

- All six throughput values (2 variants × 10/20/40 nodes) are positive and recorded in the output table
- Elapsed times include only alignment: reference transformation/indexing and input-distribution pre-processing are excluded
- Both variants used identical conditions per cluster size (same dataset, same --num_mm 4 mismatch cutoff, same --nodes executor counts)
- Reproduced trade-off: BiSpark-balance relatively lower throughput at 10 and 20 nodes, higher throughput at 40 nodes
- Throughput-versus-node-count plot shows the balancing variant's gradient catching up with / surpassing the plain variant as cluster size grows

## Limitations

- Whether load balancing improves performance cannot be concluded theoretically; load balancing and shuffling are a speed trade-off
- The cause of BiSpark-balance's speedup on large clusters is speculative (straggler waiting grows with cluster size), not established
- Benchmark timing must exclude index building and input distribution, so reported throughput is not comparable to end-to-end wall-clock timings

## Evidence

- [methods] the BiSpark utilizes portable_hash function, supported by Spark framework, to determine which partition the data should be placed: "the BiSpark utilizes portable_hash function, supported by Spark framework, to determine which partition the data should be placed"
- [discussion] Balance works faster for big clusters due to straggler waiting: "The reason BiSpark-balance works faster for big cluster might be that the more nodes should wait for the slowest node as cluster size increases."
- [discussion] Load balancing and shuffling trade-off unresolved theoretically: "Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not."
- [readme] README documents --testmode and --nodes parameters used for the throughput experiment: "--testmode: Switching testmode for experiment. Should be one of 'balancing' or 'plain'."
- [readme] README defines --nodes as executors multiplied across cluster nodes: "--nodes: The number of executors. = (# of nodes of cluster) * (# of spark executors in each node)"
