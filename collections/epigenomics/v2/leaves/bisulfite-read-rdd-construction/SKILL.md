---
name: bisulfite-read-rdd-construction
description: Use when WGBS FASTQ/A reads (e.g. GEO GSE80911) must be loaded into Spark
  key-value RDD tuples and re-partitioned with portable_hash so per-partition record
  counts become approximately even across executors.
when_to_use_negative:
- Input is already a transformed transRDD or filteredRDD downstream of Phase 1, not
  raw FASTQ/A reads
- The cluster is so small that shuffling overhead dominates — the paper notes load
  balancing and shuffling are a trade-off in terms of speed
- The target is single-node bisulfite alignment (e.g. Bismark-style scale-up), where
  Spark RDD partitioning offers no benefit
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BiSpark
  role: Provides the Phase 1 conversion of FASTQ/A inputs into key-value RDD tuples
    and the portable_hash-based load-balancing mode (--testmode balancing)
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed framework supplying the portable_hash function used to determine
    which partition the data should be placed in
  repo: https://spark.apache.org/
- name: HDFS
  role: File system that distributes the read data across nodes so RDDs need not be
    copied by the user
  repo: https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:24119657256a007eaa560e770b47088603f4845565ba5e25a8c9bf20b5195858
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - Apache Spark
  - BiSpark
  - HDFS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bisulfite-read-rdd-construction

## Summary

Constructs Apache Spark key-value RDDs from raw WGBS FASTQ/A reads (read id, read sequence) and re-distributes them across cluster nodes using Spark's portable_hash function, reproducing BiSpark's Phase 1 RDD structure and its hash-partitioning load-balancing design.

## When to use

Use when WGBS FASTQ/A reads (e.g. GEO GSE80911) must be loaded into Spark key-value RDD tuples and re-partitioned with portable_hash so per-partition record counts become approximately even across executors.

## When NOT to use

- Input is already a transformed transRDD or filteredRDD downstream of Phase 1, not raw FASTQ/A reads
- The cluster is so small that shuffling overhead dominates — the paper notes load balancing and shuffling are a trade-off in terms of speed
- The target is single-node bisulfite alignment (e.g. Bismark-style scale-up), where Spark RDD partitioning offers no benefit

## Inputs

- WGBS FASTQ/A read files (e.g. GEO GSE80911) stored on HDFS
- Unpartitioned or HDFS-chunk-partitioned readRDD as (read id, read sequence) tuples
- Alignment run configuration: --input, --ref, --nodes, --num_mm (default 4), --testmode (balancing|plain)

## Outputs

- readRDD of key-value structured tuples (read id, read sequence) ready for the 'three-letter' transform to transRDD
- Re-partitioned RDD balanced via portable_hash across executors
- partition_balance_stats.csv: before/after per-partition record counts with min, max, mean, and max/min ratio

## How to apply

Load the raw FASTQ/A files from HDFS and convert them into key-value structured tuples (read id, read sequence) forming the readRDD, per BiSpark's Phase 1. Run on Apache Spark >= 1.6 with HDFS 2.6 and Python 2.6/2.7 so the RDD can be distributed by HDFS chunk boundaries; record the number of records per partition to expose the imbalance points. Apply Spark's portable_hash function to re-partition the data at each imbalance point, mirroring BiSpark-balance's load-balancing step. Compute per-partition record counts before and after re-partitioning plus min, max, mean, and max/min ratio, writing the before/after table to partition_balance_stats.csv. Invoke the aligner with --testmode balancing (versus plain) so the load-balanced path is exercised, with --nodes set to (number of cluster nodes) * (Spark executors per node).

## Related tools

- **BiSpark** (Provides the Phase 1 conversion of FASTQ/A inputs into key-value RDD tuples and the portable_hash-based load-balancing mode (--testmode balancing)) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed framework supplying the portable_hash function used to determine which partition the data should be placed in) — https://spark.apache.org/
- **HDFS** (File system that distributes the read data across nodes so RDDs need not be copied by the user) — https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html

## Examples

```
sh ./src/alignment/run.sh --input <HDFS_FASTQ> --ref <HDFS_INDEX> --nodes 8 --num_mm 4 --testmode balancing
```

## Evaluation signals

- Per-partition record counts after portable_hash re-partitioning are approximately equal, with a markedly lower max/min partition-size ratio than the unbalanced baseline
- readRDD tuple schema matches <read id, read sequence>, matching BiSpark's Phase 1 key-value structure
- Record count in the re-partitioned RDD equals the original read count (no reads lost during re-partitioning)
- Run uses --testmode balancing and the stated --nodes count equals (# of nodes) * (# of Spark executors per node)
- partition_balance_stats.csv contains before and after summary statistics (min, max, mean, max/min ratio) for both imbalance points

## Limitations

- Load balancing and shuffling are a trade-off in terms of speed; the paper states it is hard to conclude theoretically whether BiSpark-balance performance improves
- The reason BiSpark-balance works faster for big clusters is speculative — more nodes should wait for the slowest node as cluster size increases
- Requires an older stack: Python 2.6/2.7, Apache Spark >= 1.6, HDFS 2.6, and Bowtie2 callable on all slave nodes

## Evidence

- [methods] the BiSpark accepts raw sequencing data files, FASTQ/A format, as inputs and converts them into list of key-value structured tuples: "the BiSpark accepts raw sequencing data files, FASTQ/A format, as inputs and converts them into list of key-value structured tuples"
- [methods] the BiSpark utilizes portable_hash function, supported by Spark framework, to determine which partition the data should be placed: "the BiSpark utilizes portable_hash function, supported by Spark framework, to determine which partition the data should be placed"
- [methods] the BiSpark transforms readRDDs to transRDD which consists of <read id, transformed read sequence > tuples: "the BiSpark transforms readRDDs to transRDD which consists of <read id, transformed read sequence > tuples"
- [readme] BiSpark is designed based on the Apache Spark distributed framework and shows highly efficient scalability.: "BiSpark is designed based on the Apache Spark distributed framework and shows highly efficient scalability."
- [discussion] Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not.: "Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not."
