---
name: rdd-portable-hash-partitioning
description: Use when a Spark RDD pipeline shows imbalanced partition sizes (e.g.,
  after HDFS chunk-boundary loading or post-filtering valid-alignment skew) and you
  need to quantify and rebalance records-per-partition across executors.
when_to_use_negative:
- When partition sizes are already near-uniform and a re-partition shuffle would add
  cost without benefit, since load balancing and shuffling are a trade-off in terms
  of speed
- When the goal is measuring alignment speed/mappability of the full BiSpark pipeline
  (BiSpark-plain vs BiSpark-balance) rather than isolating the re-partitioning step
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: BiSpark
  role: 'Source of the load-balancing design: uses Spark''s portable_hash function
    to determine which partition the data should be placed, re-distributing data almost
    evenly across cluster nodes'
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed framework providing the portable_hash function, RDDs, and partitioner
    used to re-partition the read key-value tuples
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:cc0c0cadebea3594c3baa6e7e2dc49a875ac77a3136145bdbc68cae69dd1d71b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - Apache Spark
  - BiSpark
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# rdd-portable-hash-partitioning

## Summary

Reproduces BiSpark's hash-partitioning load-balancing step: applying Spark's portable_hash function to re-partition key-value RDDs of bisulfite reads so that records are redistributed almost evenly across cluster nodes, minimizing straggler delays during bisulfite read alignment.

## When to use

Use when a Spark RDD pipeline shows imbalanced partition sizes (e.g., after HDFS chunk-boundary loading or post-filtering valid-alignment skew) and you need to quantify and rebalance records-per-partition across executors.

## When NOT to use

- When partition sizes are already near-uniform and a re-partition shuffle would add cost without benefit, since load balancing and shuffling are a trade-off in terms of speed
- When the goal is measuring alignment speed/mappability of the full BiSpark pipeline (BiSpark-plain vs BiSpark-balance) rather than isolating the re-partitioning step

## Inputs

- WGBS FASTQ read files (e.g., GEO GSE80911) loaded as Spark key-value RDD of <read id, read sequence> tuples (readRDD)
- Partition-size counts at known imbalance points (HDFS chunk boundaries, post-filtering valid-alignment ratio)

## Outputs

- Re-partitioned RDD with balanced per-partition record counts
- Before/after partition-size balance table (per-partition counts, min, max, mean, max/min ratio), e.g., partition_balance_stats.csv

## How to apply

Load WGBS FASTQ reads (e.g., GEO GSE80911) as key-value RDD tuples of <read id, read sequence>, reproducing BiSpark's readRDD structure, and record the number of records per partition to identify imbalance points such as HDFS chunk boundaries and post-filtering valid-alignment skew. Apply Spark's portable_hash function via re-partitioning at each imbalance point, since BiSpark uses portable_hash 'to determine which partition the data should be placed'. Compute per-partition record counts before and after re-partitioning, plus min, max, mean, and max/min ratio summary statistics for each partition set. Judge success by the balanced (post-portable_hash) partition set showing approximately equal record counts across executors, i.e., a markedly lower max/min partition-size ratio than the unbalanced baseline. Note the paper's caveat that load balancing and shuffling are a trade-off in terms of speed, so improved partition balance does not theoretically guarantee improved runtime.

## Related tools

- **BiSpark** (Source of the load-balancing design: uses Spark's portable_hash function to determine which partition the data should be placed, re-distributing data almost evenly across cluster nodes) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed framework providing the portable_hash function, RDDs, and partitioner used to re-partition the read key-value tuples)

## Examples

```
rdd.repartition(nExecutors)  # apply Spark's portable_hash partitioner to readRDD at each imbalance point, then count records per partition
```

## Evaluation signals

- Re-partitioned (balanced) partitions show approximately equal record counts across executors
- Max/min partition-size ratio after portable_hash re-partitioning is markedly lower than the unbalanced baseline
- Per-partition record counts are recorded at both imbalance points (HDFS chunk boundary and post-filtering valid-alignment ratio) before and after re-partitioning
- Summary statistics table (min, max, mean, max/min ratio) is computed for each partition set and written to the output CSV

## Limitations

- Load balancing and shuffling are a trade-off in terms of speed, and it is hard to conclude theoretically whether the performance would be improved or not
- The reason BiSpark-balance works faster for a big cluster is speculative: the more nodes there are, the more should wait for the slowest node as cluster size increases
- Requires Apache Spark >= 1.6, HDFS 2.6, and Python 2.6/2.7 per the BiSpark README

## Evidence

- [methods] portable_hash load balancing: "the BiSpark utilizes portable_hash function, supported by Spark framework, to determine which partition the data should be placed"
- [abstract] load-balanced aligner claim: "we present a highly scalable, efficient, and load-balanced bisulfite aligner, BiSpark, which is designed for processing large volumes of bisulfite sequencing data"
- [discussion] shuffling trade-off: "Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not."
- [methods] key-value RDD input structure: "the BiSpark accepts raw sequencing data files, FASTQ/A format, as inputs and converts them into list of key-value structured tuples"
- [readme] README run parameters: "--testmode: Switching testmode for experiment. Should be one of 'balancing' or 'plain'."
