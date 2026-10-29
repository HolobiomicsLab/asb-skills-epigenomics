---
name: scalability-curve-analysis
description: 'Use when evaluating a distributed aligner or Spark-based pipeline''s
  scale-out behavior: you have access to clusters of differing node counts (e.g. 10,
  20, 40 slaves) and need to quantify throughput gain or loss as nodes are added.'
when_to_use_negative:
- You only have access to a single cluster size — a two-point (or one-point) throughput
  measurement cannot reveal a scaling gradient or the balance-vs-shuffle trade-off.
- The benchmark question is per-node performance against single-node tools (e.g. BiSpark
  vs Bismark or CloudAligner timing) rather than scale-out behavior of one tool across
  cluster sizes.
- Input data or reference indexing time varies between runs — including non-alignment
  pre-processing (reference transformation/indexing, input distribution) will confound
  the elapsed-time comparison.
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: BiSpark
  role: Bisulfite aligner benchmarked in plain (--testmode plain) and load-balanced
    (--testmode balancing) variants across 10/20/40-node clusters
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed framework on which BiSpark runs; its portable_hash partitioner
    provides the load-balancing mode under test
- name: HDFS
  role: Shared file system holding the 16GB WGBS input dataset and hg19 chromosome
    1 index so all nodes read identical inputs
  repo: https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
- name: Bowtie2
  role: Underlying short-read aligner used by BiSpark's 'three-letter' mapping; must
    be callable on all slave nodes for the benchmark
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:b1a29416f7bf311db109100505bc17309a401d2b434013aad7435f1fac1b49d2
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
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# scalability-curve-analysis

## Summary

Measures how a distributed tool's throughput scales with cluster size by running identical workloads on clusters of increasing node counts and plotting throughput (reads/second) versus number of slave nodes. Used in the BiSpark evaluation to show that load balancing (BiSpark-balance) costs throughput on small clusters but gains it on large ones.

## When to use

Use when evaluating a distributed aligner or Spark-based pipeline's scale-out behavior: you have access to clusters of differing node counts (e.g. 10, 20, 40 slaves) and need to quantify throughput gain or loss as nodes are added.

## When NOT to use

- You only have access to a single cluster size — a two-point (or one-point) throughput measurement cannot reveal a scaling gradient or the balance-vs-shuffle trade-off.
- The benchmark question is per-node performance against single-node tools (e.g. BiSpark vs Bismark or CloudAligner timing) rather than scale-out behavior of one tool across cluster sizes.
- Input data or reference indexing time varies between runs — including non-alignment pre-processing (reference transformation/indexing, input distribution) will confound the elapsed-time comparison.

## Inputs

- Fixed-size sequencing dataset (e.g. 100M reads, 16GB WGBS FASTQ from GEO GSE80911) staged in HDFS
- Pre-built reference index (hg19 Build 37 chromosome 1) in HDFS
- Clusters of at least two sizes (e.g. 10, 20, and 40 slave nodes)
- Tool variant configurations to compare (e.g. BiSpark --testmode plain vs --testmode balancing)

## Outputs

- Table of variant, cluster size, elapsed time, and throughput (reads/second) for each run
- Scalability plot of throughput versus number of slave nodes, one series per variant
- Trade-off statement: relative throughput of the balancing variant at each cluster size

## How to apply

Load a fixed test dataset (e.g. the 100-million-read / 16GB WGBS dataset from GEO GSE80911) and the reference index (hg19 chromosome 1, Build 37) from HDFS so input distribution is identical across runs. Execute the tool on clusters of 10, 20, and 40 slave nodes in each variant configuration — for BiSpark, --testmode plain (Bowtie2-based 'three-letter' alignment, default mismatch cutoff 4) versus --testmode balancing (portable_hash re-partitioning enabled) — excluding reference transformation/indexing and input-distribution pre-processing from measured elapsed time. Compute throughput for each run as number of processed reads per second, then tabulate variant, cluster size, elapsed time, and throughput. Plot throughput versus number of slave nodes and compare gradients: the load-balanced variant should be relatively slower at 10–20 nodes but overtake the plain variant at 40 nodes, because with more nodes more must wait for the slowest straggler, making load redistribution worthwhile despite shuffling overhead.

## Related tools

- **BiSpark** (Bisulfite aligner benchmarked in plain (--testmode plain) and load-balanced (--testmode balancing) variants across 10/20/40-node clusters) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed framework on which BiSpark runs; its portable_hash partitioner provides the load-balancing mode under test)
- **HDFS** (Shared file system holding the 16GB WGBS input dataset and hg19 chromosome 1 index so all nodes read identical inputs) — https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
- **Bowtie2** (Underlying short-read aligner used by BiSpark's 'three-letter' mapping; must be callable on all slave nodes for the benchmark) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml

## Examples

```
sh ./src/alignment/run.sh --input reads.fastq --ref hg19_chr1_index --nodes 40 --num_mm 4 --testmode balancing
```

## Evaluation signals

- All six throughput values (2 variants × 3 cluster sizes) are positive and present in the output table.
- Measured elapsed time excludes reference transformation/indexing and input-distribution pre-processing for every run, so runs are comparable.
- BiSpark-balance shows relatively lower throughput than BiSpark-plain at 10 and 20 nodes but higher throughput at 40 nodes (the reported trade-off).
- Throughput-versus-node-count plot shows the balance variant's curve overtaking the plain variant's gradient as cluster size grows.
- The same fixed dataset (100M reads / 16GB) and identical alignment parameters (default mismatch cutoff 4) are used for all runs.

## Limitations

- The trade-off is empirical, not theoretical: 'Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not.'
- The mechanism behind BiSpark-balance's speedup on large clusters is speculative: 'might be that the more nodes should wait for the slowest node as cluster size increases.'
- The balancing mode's gain only appears at large cluster sizes; on small clusters it imposes shuffling overhead with no offsetting benefit.

## Evidence

- [discussion] Trade-off mechanism: "The reason BiSpark-balance works faster for big cluster might be that the more nodes should wait for the slowest node as cluster size increases."
- [discussion] Theoretical indeterminacy of the trade-off: "Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not."
- [methods] Balancing mechanism (portable_hash repartitioning): "the BiSpark utilizes portable_hash function, supported by Spark framework, to determine which partition the data should be placed"
- [readme] Cluster sizing parameter in the benchmark harness: "--nodes: The number of executors. = (# of nodes of cluster) * (# of spark executors in each node)"
- [readme] Variant switch used to toggle the benchmark modes: "--testmode: Switching testmode for experiment. Should be one of 'balancing' or 'plain'."
