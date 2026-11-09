---
name: hash-partitioning-load-balancing
description: Decision support for choosing between BiSpark --testmode plain and balancing
  when scaling bisulfite read alignment across 10-, 20-, or 40-node Spark/HDFS clusters
  and throughput is limited by the slowest node.
when_to_use_negative:
- When the input data are already evenly partitioned or the cluster is small (e.g.
  10-20 nodes), where shuffling overhead from portable_hash repartitioning makes balancing
  mode slower than plain mode
- 'When measuring preprocessing performance: reference genome transformation/indexing
  and input-distribution pre-processing should be excluded from the timed comparison'
- When comparing against single-node aligners (e.g. Bismark, BSMAP) rather than evaluating
  BiSpark's own scalability variants
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: BiSpark
  role: Bisulfite aligner whose --testmode plain/balancing flag switches the portable_hash
    load-balancing partitioning on or off
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed framework providing the portable_hash partitioning function used
    to determine which partition each data tuple is placed on
- name: HDFS
  role: Distributed file system holding the input reads and the hg19 index shared
    across slave nodes
- name: Bowtie2
  role: Underlying aligner invoked by BiSpark for the three-letter mapping phase in
    both plain and balancing modes; must be callable on all slave nodes
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:d87509baf967123d26454444266f91009e5421a4a556191221c1571eead19b13
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hash-partitioning-load-balancing

## Summary

A BiSpark execution mode that applies Spark's portable_hash partitioning to redistribute key-value RDD tuples evenly across slave nodes during bisulfite 'three-letter' alignment, trading shuffling overhead for protection against straggler nodes that dominates as cluster size grows.

## When to use

Decision support for choosing between BiSpark --testmode plain and balancing when scaling bisulfite read alignment across 10-, 20-, or 40-node Spark/HDFS clusters and throughput is limited by the slowest node.

## When NOT to use

- When the input data are already evenly partitioned or the cluster is small (e.g. 10-20 nodes), where shuffling overhead from portable_hash repartitioning makes balancing mode slower than plain mode
- When measuring preprocessing performance: reference genome transformation/indexing and input-distribution pre-processing should be excluded from the timed comparison
- When comparing against single-node aligners (e.g. Bismark, BSMAP) rather than evaluating BiSpark's own scalability variants

## Inputs

- Bisulfite-treated sequencing reads (FASTQ/A) from a WGBS library, e.g. the 100-million-read / 16 GB dataset derived from GEO GSE80911, staged in HDFS
- Transformed and indexed hg19 (Build 37) reference genome (chromosome 1) index directory in HDFS produced by BiSpark's build_index step
- Cluster configuration: number of Spark executors (--nodes), number of slave nodes (10, 20, or 40)

## Outputs

- Aligned reads (SAM file, via --local_save) from both plain and balancing runs
- Table of variant (plain vs. balancing), cluster size, elapsed time, and throughput (reads/second) for each of the six runs
- Throughput-versus-number-of-slave-nodes plot comparing the scalability gradients of the two variants

## How to apply

Run the BiSpark alignment workflow (key-value RDD construction, three-letter transforming/mapping, best-alignment filtering with the default mismatch cutoff of 4, methylation profiling) twice per cluster size: once with --testmode plain and once with --testmode balancing, which enables Spark's portable_hash repartitioning. Use a 100-million-read (~16 GB) WGBS FASTQ input (e.g. from GEO GSE80911) against an hg19 Build 37 chromosome index staged in HDFS, and exclude reference indexing and input-distribution pre-processing from the measured elapsed time. Compute throughput as processed reads per second for each variant/cluster-size combination and tabulate variant, cluster size, elapsed time, and throughput. The expected trade-off: balancing is relatively slower at 10 and 20 nodes (shuffling cost dominates) but faster at 40 nodes, because more nodes must wait for the slowest node as cluster size increases. Note the authors' caveat that load balancing vs. shuffling is a trade-off that cannot be concluded theoretically, so the decision must be validated empirically per cluster.

## Related tools

- **BiSpark** (Bisulfite aligner whose --testmode plain/balancing flag switches the portable_hash load-balancing partitioning on or off) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed framework providing the portable_hash partitioning function used to determine which partition each data tuple is placed on)
- **HDFS** (Distributed file system holding the input reads and the hg19 index shared across slave nodes)
- **Bowtie2** (Underlying aligner invoked by BiSpark for the three-letter mapping phase in both plain and balancing modes; must be callable on all slave nodes) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml

## Examples

```
./src/alignment/run.sh --input hdfs:///data/GSE80911_100M.fastq --ref hdfs:///index/hg19_chr1 --testmode balancing --num_mm 4 --nodes 40 --local_save /out/aligned.sam --log /logs/bispark.log
```

## Evaluation signals

- All six throughput values (2 variants x 3 cluster sizes) are positive and recorded in the output table
- BiSpark-balance shows relatively lower throughput than BiSpark-plain at 10 and 20 nodes but higher throughput at 40 nodes
- Elapsed time excludes reference genome transformation/indexing and input-distribution pre-processing steps
- Both runs use identical conditions: same 100M-read WGBS input, same hg19 index in HDFS, and default mismatch cutoff of 4
- Log files confirm --testmode was set to 'balancing' (portable_hash enabled) vs 'plain' for each corresponding run

## Limitations

- Load balancing and shuffling are a trade-off in terms of speed; there is no theoretical conclusion on whether balancing improves performance — it must be measured empirically
- The explanation for BiSpark-balance's speedup on large clusters (straggler waiting grows with cluster size) is speculative, not established
- No software version numbers are stated for BiSpark, Spark, or HDFS, complicating exact reproduction (README requires Python 2.6/2.7, Apache Spark >= 1.6, HDFS 2.6)

## Evidence

- [methods] portable_hash partitioning rationale: "the BiSpark utilizes portable_hash function, supported by Spark framework, to determine which partition the data should be placed"
- [discussion] trade-off is empirical, not theoretical: "Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not."
- [discussion] straggler explanation for large-cluster gain: "The reason BiSpark-balance works faster for big cluster might be that the more nodes should wait for the slowest node as cluster size increases."
- [readme] testmode flag controls balancing: "--testmode: Switching testmode for experiment. Should be one of 'balancing' or 'plain'."
