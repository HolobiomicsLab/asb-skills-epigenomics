---
name: spark-distributed-data-load-balancing
description: Use when running bisulfite (WGBS/FASTQ) read alignment on a Spark cluster
  where uneven partitions — e.g. from HDFS chunk boundaries or post-filtering imbalances
  — cause some executors to lag and dominate wall-clock time.
when_to_use_negative:
- Input data is already evenly distributed across partitions — re-shuffling only adds
  overhead since load balancing and shuffling are a trade-off in terms of speed.
- Small datasets or single-node runs where BiSpark-plain suffices; the paper attributes
  BiSpark-balance's advantage to large clusters where more nodes wait for the slowest
  node.
- The task is alignment accuracy or methylation profiling rather than distributed-execution
  balancing — this skill alters partition placement, not mapping results.
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: BiSpark
  role: bisulfite aligner whose --testmode 'balancing' option enables the hash-partitioning
    load-balancing mode under evaluation
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: distributed framework providing portable_hash and RDD partitioning used to
    re-distribute reads across executors
- name: HDFS
  role: shared file system whose chunk boundaries are one source of partition imbalance
    being corrected
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:c6f52b36139874223bd967fd17ce42a9aef951ee6edc89cfd569016cf38bf9a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - Apache Spark
  - BiSpark
  - HDFS
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spark-distributed-data-load-balancing

## Summary

Reproduces BiSpark-balance's hash-partitioning load-balancing step, which applies Spark's portable_hash function to re-distribute bisulfite read RDDs almost evenly across cluster executors to minimize straggler delays during 'three-letter' alignment.

## When to use

Use when running bisulfite (WGBS/FASTQ) read alignment on a Spark cluster where uneven partitions — e.g. from HDFS chunk boundaries or post-filtering imbalances — cause some executors to lag and dominate wall-clock time.

## When NOT to use

- Input data is already evenly distributed across partitions — re-shuffling only adds overhead since load balancing and shuffling are a trade-off in terms of speed.
- Small datasets or single-node runs where BiSpark-plain suffices; the paper attributes BiSpark-balance's advantage to large clusters where more nodes wait for the slowest node.
- The task is alignment accuracy or methylation profiling rather than distributed-execution balancing — this skill alters partition placement, not mapping results.

## Inputs

- WGBS FASTQ reads (e.g. GEO GSE80911)
- key-value RDD of <read id, read sequence> tuples (readRDD) in Apache Spark
- partition-size record counts at imbalance points (HDFS chunk boundaries, post-filter valid-alignment ratios)

## Outputs

- re-partitioned RDD with near-even records per executor
- before/after partition-size balance table (partition_balance_stats.csv) with min, max, mean, and max/min ratio per partition set

## How to apply

Load WGBS FASTQ reads (e.g. GEO GSE80911) as key-value RDD tuples (read id, read sequence) matching BiSpark's readRDD structure, then reproduce the known imbalance points by recording records per partition. At each imbalance point, invoke Spark's portable_hash function on the RDD keys to decide which partition each record should be placed, as the paper specifies for its load-balancing design. Compute per-partition record counts before and after re-partitioning with min, max, mean, and max/min ratio statistics. Validation succeeds when balanced partitions show approximately equal counts and a markedly lower max/min ratio than the unbalanced baseline; note the paper leaves the load-balancing-versus-shuffling speed trade-off unresolved theoretically, so verify empirically on your cluster.

## Related tools

- **BiSpark** (bisulfite aligner whose --testmode 'balancing' option enables the hash-partitioning load-balancing mode under evaluation) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (distributed framework providing portable_hash and RDD partitioning used to re-distribute reads across executors)
- **HDFS** (shared file system whose chunk boundaries are one source of partition imbalance being corrected)

## Examples

```
./src/alignment/run.sh --input reads.fastq --ref hdfs:///index --nodes 8 --testmode balancing
```

## Evaluation signals

- Post-re-partitioning max/min partition-size ratio is markedly lower than the unbalanced baseline.
- Per-partition record counts are approximately equal across all executors after applying portable_hash.
- Speedup is observed on large clusters, consistent with the rationale that more nodes would otherwise wait for the slowest node.
- Mapped-output equivalence: re-partitioning changes placement, not alignment results, versus BiSpark-plain.

## Limitations

- No theoretical conclusion on whether BiSpark-balance improves performance; load balancing and shuffling are a trade-off in terms of speed and the effect is hard to predict.
- The reason BiSpark-balance works faster for big clusters is speculative — possibly the more nodes wait for the slowest node as cluster size increases.
- No software version numbers stated for BiSpark, Spark, or HDFS in the article (README requirements list Python 2.6/2.7, Spark >= 1.6, HDFS 2.6).

## Evidence

- [methods] the BiSpark utilizes portable_hash function, supported by Spark framework, to determine which partition the data should be placed: "the BiSpark utilizes portable_hash function, supported by Spark framework, to determine which partition the data should be placed"
- [discussion] Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not.: "Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not."
- [abstract] we present a highly scalable, efficient, and load-balanced bisulfite aligner, BiSpark, which is designed for processing large volumes of bisulfite sequencing data: "we present a highly scalable, efficient, and load-balanced bisulfite aligner, BiSpark, which is designed for processing large volumes of bisulfite sequencing data"
- [readme] --testmode: Switching testmode for experiment. Should be one of 'balancing' or 'plain'.: "--testmode: Switching testmode for experiment. Should be one of 'balancing' or 'plain'."
- [intro] (1) Distributing the reads into key-value pairs: "(1) Distributing the reads into key-value pairs"
