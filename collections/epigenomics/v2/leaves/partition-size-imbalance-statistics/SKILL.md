---
name: partition-size-imbalance-statistics
description: Use when a distributed Spark pipeline (e.g. BiSpark-style bisulfite alignment
  on WGBS FASTQ reads) shows uneven per-partition loads across executors, and you
  need to quantify imbalance and verify that hash-based re-partitioning balances record
  counts.
when_to_use_negative:
- Input is not distributed Spark RDD data (e.g. a single-node aligner output such
  as a Bismark SAM file) — partition-size balance statistics have no meaning there
- When the goal is alignment accuracy or mappability assessment rather than data-distribution
  skew across nodes
- 'When shuffling cost dominates and the pipeline is single-node: the paper notes
  load balancing and shuffling are a trade-off and it is hard to conclude theoretically
  whether performance would improve'
edam_topics:
- http://edamontology.org/topic_0769
tools:
- name: BiSpark
  role: Source of the hash-partitioning load-balancing design (BiSpark-balance); its
    key-value readRDD structure and two imbalance points define what is measured
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed framework whose portable_hash function is used to re-partition
    records evenly across cluster partitions
  repo: https://spark.apache.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:9995512b0a7f0c95d3ecf1b1a167484dce7116eaf4d931478464dec883f00be1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - Apache Spark
  - BiSpark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# partition-size-imbalance-statistics

## Summary

A diagnostic skill for quantifying data skew across Spark RDD partitions during bisulfite read alignment, using per-partition record counts and max/min ratios before and after hash-partitioning re-distribution. It operationalizes BiSpark's load-balancing claim by making partition imbalance measurable with Spark's portable_hash function.

## When to use

Use when a distributed Spark pipeline (e.g. BiSpark-style bisulfite alignment on WGBS FASTQ reads) shows uneven per-partition loads across executors, and you need to quantify imbalance and verify that hash-based re-partitioning balances record counts.

## When NOT to use

- Input is not distributed Spark RDD data (e.g. a single-node aligner output such as a Bismark SAM file) — partition-size balance statistics have no meaning there
- When the goal is alignment accuracy or mappability assessment rather than data-distribution skew across nodes
- When shuffling cost dominates and the pipeline is single-node: the paper notes load balancing and shuffling are a trade-off and it is hard to conclude theoretically whether performance would improve

## Inputs

- WGBS FASTQ reads (e.g. GEO GSE80911) loaded as key-value RDD tuples (read id, read sequence) in Apache Spark
- Per-partition record counts from the unbalanced baseline (HDFS chunk-boundary partitions, post-filtering valid-alignment-ratio partitions)
- Number of executors (--nodes: (# of nodes of cluster) * (# of spark executors in each node))

## Outputs

- Before/after partition-size balance table (partition_balance_stats.csv) with per-partition record counts
- Summary statistics per partition set: min, max, mean, and max/min partition-size ratio
- Verification that re-partitioned (balanced) partitions show approximately equal record counts across executors

## How to apply

Load the FASTQ reads (e.g. the WGBS dataset GEO GSE80911) into key-value RDD tuples of (read id, read sequence), reproducing BiSpark's readRDD structure from Phase 1 (Converting To Key-Value RDD Structure). Identify the known imbalance points: partitions formed on HDFS chunk boundaries and on the post-filtering valid-alignment ratio, recording records per partition at each point. Apply Spark's portable_hash function — which BiSpark uses to determine which partition data should be placed in — via partitionBy to re-distribute the data. Compute per-partition record counts before and after re-partitioning, plus min, max, mean, and the max/min ratio for each partition set, then write the before/after balance table (e.g. partition_balance_stats.csv). Validate by checking that the balanced partitions show approximately equal counts, i.e. a markedly lower max/min ratio than the unbalanced baseline; note the paper's caveat that load balancing trades off against shuffling cost, so improved partition balance does not guarantee faster runtime.

## Related tools

- **BiSpark** (Source of the hash-partitioning load-balancing design (BiSpark-balance); its key-value readRDD structure and two imbalance points define what is measured) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed framework whose portable_hash function is used to re-partition records evenly across cluster partitions) — https://spark.apache.org/

## Examples

```
./src/alignment/run.sh --input reads.fastq --ref index_dir --nodes 8 --num_mm 4 --testmode balancing
```

## Evaluation signals

- Re-partitioned (balanced) partitions show approximately equal record counts across executors
- Max/min partition-size ratio after portable_hash re-partitioning is markedly lower than the unbalanced HDFS-chunk-boundary baseline
- Per-partition record counts are recorded at both imbalance points (HDFS chunk boundaries and post-filtering valid-alignment ratio) before and after re-partitioning
- partition_balance_stats.csv contains before/after columns plus min, max, mean, and max/min summary statistics per partition set
- Total record count after re-partitioning equals the total before (no reads lost or duplicated during partitionBy)

## Limitations

- Load balancing and shuffling are a trade-off in terms of speed; it is hard to conclude theoretically whether performance improves, and the paper leaves this unresolved empirically
- The reason BiSpark-balance works faster on large clusters is speculative — possibly because more nodes must wait for the slowest node as cluster size increases — not established
- No software version numbers are stated for BiSpark, Spark, or HDFS, so reproducing the exact partitioning behavior may vary across Spark >= 1.6 / HDFS 2.6 environments
- The paper notes a valid-alignment-ratio-driven imbalance after filtering ambiguous reads, which is data-dependent and may not reproduce identically on other datasets

## Evidence

- [methods] the BiSpark utilizes portable_hash function, supported by Spark framework, to determine which partition the data should be placed: "the BiSpark utilizes portable_hash function, supported by Spark framework, to determine which partition the data should be placed"
- [full_text] BiSpark implements a highly-optimized load-balancing algorithm that re-distributes data almost evenly across the cluster nodes: [evidence span withheld — anchor status: not_found]
- [discussion] Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not.: "Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not."
- [readme] --testmode: Switching testmode for experiment. Should be one of 'balancing' or 'plain'.: "--testmode: Switching testmode for experiment. Should be one of 'balancing' or 'plain'."
