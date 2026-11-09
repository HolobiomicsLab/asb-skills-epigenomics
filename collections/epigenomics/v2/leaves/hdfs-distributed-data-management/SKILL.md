---
name: hdfs-distributed-data-management
description: Use when running BiSpark or a similar Spark/Hadoop pipeline over large
  FASTQ/A read sets (e.g. GSE80911 WGBS data at 10M-200M reads) where reference indices
  and inputs must be accessible from every slave node.
when_to_use_negative:
- The analysis runs on a single node where a local filesystem suffices and HDFS overhead
  adds no benefit
- The aligner being used (e.g. single-server Bismark with the 5-multicore option)
  does not read from HDFS
- Only small test datasets (e.g. Sherman-simulated 1M reads from chr1) where manual
  per-node file distribution is trivial
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiSpark
  role: Provides the build_index and alignment entry points whose --input/--output/--ref
    arguments consume and produce HDFS paths
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed processing framework whose executors read inputs and reference
    indices directly from HDFS across nodes
- name: HDFS
  role: Distributed file system that stores the FASTQ/A inputs and the four bisulfite-converted
    Bowtie2 indices for multi-node sharing
  repo: https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
- name: Bowtie2
  role: Indexing/alignment engine applied to the HDFS-shared converted reference;
    must be callable on all slave nodes
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:f29a9354d63d1fade2c8dd1ba04910cb7b094570015ef227e35991200a330ac1
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
  - HDFS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# HDFS distributed data management

## Summary

Stages bisulfite sequencing inputs and reference index artifacts on HDFS so that a Spark-based aligner cluster (BiSpark) can share reference conversions and FASTQ read data across all nodes without manual per-node distribution.

## When to use

Use when running BiSpark or a similar Spark/Hadoop pipeline over large FASTQ/A read sets (e.g. GSE80911 WGBS data at 10M-200M reads) where reference indices and inputs must be accessible from every slave node.

## When NOT to use

- The analysis runs on a single node where a local filesystem suffices and HDFS overhead adds no benefit
- The aligner being used (e.g. single-server Bismark with the 5-multicore option) does not read from HDFS
- Only small test datasets (e.g. Sherman-simulated 1M reads from chr1) where manual per-node file distribution is trivial

## Inputs

- Reference genome FASTA (e.g. human Build 37 / hg19 chromosome FASTA) placed in HDFS
- WGBS FASTQ/A read files (e.g. Illumina HiSeq 2500, 95 bp reads from GEO GSE80911) placed in HDFS
- HDFS 2.6 and Apache Spark >= 1.6 cluster with Bowtie2 installed on all slave nodes

## Outputs

- HDFS index directory containing the four bisulfite-converted, Bowtie2-indexed reference conversions (from build_index --output)
- HDFS-resident input data accessible by all Spark executors for the alignment step
- Local SAM alignment output (--local_save) and HDFS output directory from the alignment run

## How to apply

Deploy HDFS (version 2.6 per the README) alongside Apache Spark (>= 1.6) before any alignment run. Perform the four CT/GA 'three-letter' conversions of the reference genome (Build 37/hg19) and their Bowtie2 indexing on the master node via BiSpark's build_index step, then move the resulting index directory into HDFS for multi-node sharing. Place the input FASTQ/A files in HDFS and pass their HDFS paths to the alignment step via --input and --ref; this is why the aligner does not require the user to scatter files to individual nodes. Reference pre-processing time is excluded from measured alignment runtime, so index in HDFS ahead of benchmarking. Verify that Bowtie2 is callable on all slave nodes so each executor can use the HDFS-shared index during the mapping phase.

## Related tools

- **BiSpark** (Provides the build_index and alignment entry points whose --input/--output/--ref arguments consume and produce HDFS paths) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed processing framework whose executors read inputs and reference indices directly from HDFS across nodes)
- **HDFS** (Distributed file system that stores the FASTQ/A inputs and the four bisulfite-converted Bowtie2 indices for multi-node sharing) — https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
- **Bowtie2** (Indexing/alignment engine applied to the HDFS-shared converted reference; must be callable on all slave nodes) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml

## Examples

```
bash ./src/build_index/run.sh --input hdfs://ref/hg19_chr1.fa --output hdfs://idx/hg19_bisulfite --log build_index.log
```

## Evaluation signals

- The HDFS output directory from build_index contains index files for all four reference conversions (CT/GA 'three-letter' variants) and --ref points to that same path
- Alignment runs complete on slave nodes without any user-side per-node file copying, confirming HDFS sharing is active
- Reference pre-processing (conversion and indexing) is finished before timing the alignment runs, mirroring the paper's runtime exclusions
- Spark/HDFS versions satisfy README requirements (Spark >= 1.6, HDFS 2.6) and Bowtie2 is callable on every slave node
- A full benchmark run (e.g. 10M GSE80911 reads on 20 nodes) reproduces the paper's scale of results (~617 s for BiSpark-plain), indicating data staging did not bottleneck the run

## Limitations

- No software version numbers are stated for BiSpark itself or its changelog; version compatibility beyond the README's Spark/HDFS requirements is unverified
- Reference genome conversion and indexing are done on the master node first, so index-building can be a bottleneck for very large genomes before the move to HDFS
- BiSpark requires the full Spark/HDFS/Bowtie2 stack installed cluster-wide; Bowtie2 must be callable on all slave nodes or alignment will fail

## Evidence

- [methods] all four reference genome conversion and indexing are performed in the master node first and moved to HDFS for multi-node sharing: "all four reference genome conversion and indexing are performed in the master node first and moved to HDFS for multi-node sharing"
- [readme] Using HDFS: Do not need to consider files to be distributed by user.: "Using HDFS: Do not need to consider files to be distributed by user."
- [readme] BiSpark is implemented on Apache Spark framework and HDFS file system. Bowtie2 should be callable on all slave nodes.: "Bowtie2 should be callable on all slave nodes."
- [methods] we used human reference genome (ver. Build 37, hg19): "we used human reference genome (ver. Build 37, hg19)"
