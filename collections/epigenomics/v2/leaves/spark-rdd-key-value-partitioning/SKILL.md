---
name: spark-rdd-key-value-partitioning
description: Use when building a Spark-based bisulfite alignment pipeline from FASTQ/A
  reads and you need to convert raw sequencing data into distributed, key-value-structured
  RDDs for parallel processing.
when_to_use_negative:
- The data is not sequencing reads and cannot be keyed by read identifier into key-value
  tuples
- A single-node, non-distributed aligner is preferred and cluster/HDFS infrastructure
  is unavailable
- The task is methylation calling on already-aligned BAM/SAM data rather than distributing
  raw reads for alignment
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: Apache Spark
  role: Distributed data processing framework used to implement readRDD key-value
    partitioning and the portable_hash load-balancing function
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: HDFS
  role: File system used to share converted and indexed reference genomes across all
    slave nodes
  repo: https://github.com/bhi-kimlab/BiSpark/blob/master/docs/BisPark_workflow_v2.png
- name: Bowtie2
  role: Aligner invoked on the transRDD variants against the converted reference genomes
    after key-value distribution
- name: BiSpark
  role: The overall Spark-based bisulfite aligner whose Phase 1 ('Converting To Key-Value
    RDD Structure') implements this skill
  repo: https://github.com/bhi-kimlab/BiSpark/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:498ad20e9a379475b0aec4a1fdf036ab86b149cb96b394fdf85c3892f5930ebe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - Apache Spark
  - BiSpark
  - Bowtie2
  - HDFS
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# spark-rdd-key-value-partitioning

## Summary

A method for distributing bisulfite-treated sequencing reads into Spark key-value RDD tuples so the four BiSpark workflow phases can run in parallel across a cluster; it is the foundational data-structuring step of BiSpark.

## When to use

Use when building a Spark-based bisulfite alignment pipeline from FASTQ/A reads and you need to convert raw sequencing data into distributed, key-value-structured RDDs for parallel processing.

## When NOT to use

- The data is not sequencing reads and cannot be keyed by read identifier into key-value tuples
- A single-node, non-distributed aligner is preferred and cluster/HDFS infrastructure is unavailable
- The task is methylation calling on already-aligned BAM/SAM data rather than distributing raw reads for alignment

## Inputs

- FASTQ/A raw sequencing read files (e.g., Sherman-simulated bisulfite reads)
- Reference genome FASTA file located in HDFS
- Bowtie2 indexes of the four converted reference genomes (CT/GA transformations) shared via HDFS

## Outputs

- readRDD: key-value tuples <read id, read sequence> partitioned across Spark executors
- transRDD (sub-categorized into CTtransRDD and GAtransRDD): <read id, transformed three-letter read sequence> tuples
- Downstream mapRDDs, combRDD, filteredRDD, and methylRDD feeding the final SAM-format output

## How to apply

Load the input FASTQ/A files (e.g., Sherman-simulated reads: human chromosome 1, 95 bp, 1,000,000 reads) into Spark key-value tuples keyed by read identifier and stored as readRDD partitions. Apply hash partitioning via Spark's portable_hash function to determine which partition data is placed in, enabling load-balanced distribution across nodes (the BiSpark-balance mode). Store the indexed, converted reference genomes in HDFS so all nodes share them without manual file distribution. From the initial readRDD, derive transRDD as <read id, transformed read sequence> tuples, sub-categorized into CTtransRDD and GAtransRDD for the 'three-letter' conversion. Downstream phases (mapping with Bowtie2 against four converted hg19 references, aggregation into combRDD, filtering to filteredRDD, methylation profiling into methylRDD) operate on these RDD structures.

## Related tools

- **Apache Spark** (Distributed data processing framework used to implement readRDD key-value partitioning and the portable_hash load-balancing function) — https://github.com/bhi-kimlab/BiSpark/
- **HDFS** (File system used to share converted and indexed reference genomes across all slave nodes) — https://github.com/bhi-kimlab/BiSpark/blob/master/docs/BisPark_workflow_v2.png
- **Bowtie2** (Aligner invoked on the transRDD variants against the converted reference genomes after key-value distribution)
- **BiSpark** (The overall Spark-based bisulfite aligner whose Phase 1 ('Converting To Key-Value RDD Structure') implements this skill) — https://github.com/bhi-kimlab/BiSpark/

## Examples

```
./src/alignment/run.sh --input /path/reads.fastq --output /path/output_in_hdfs --ref /path/index_in_hdfs --nodes 8 --num_mm 4 --testmode plain --appname BiSpark_run
```

## Evaluation signals

- On the 0%-error Sherman simulation set (human chromosome 1, 95 bp, 1,000,000 reads), the run should reproduce mappability of 0.9569 with precision 1.0 for both BiSpark-plain and BiSpark-balance
- readRDD partitions should cover all input reads keyed uniquely by read identifier
- transRDD should contain both CTtransRDD and GAtransRDD variants per read (two three-letter conversions)
- Union of the four mapRDDs (W-CT, W-GA, C-CT, C-GA) should occur at Spark context level without shuffle
- Ambiguous multi-best alignments and alignments exceeding the user-defined mismatch cutoff (default 4) should be discarded in filteredRDD

## Limitations

- Load balancing and shuffling are a trade-off in terms of speed; it is hard to conclude theoretically whether BiSpark-balance improves performance
- No software version numbers were stated for BiSpark, Spark, or HDFS in the discussion
- BiSpark requires Python 2.6/2.7, Apache Spark >= 1.6, HDFS 2.6, and Bowtie2 callable on all slave nodes

## Evidence

- [methods] Phase 1: Converting To Key-Value RDD Structure: "the BiSpark accepts raw sequencing data files, FASTQ/A format, as inputs and converts them into list of key-value structured tuples"
- [methods] Load balancing via portable_hash: "the BiSpark utilizes portable_hash function, supported by Spark framework, to determine which partition the data should be placed"
- [intro] Workflow phase 1 in intro: "(1) Distributing the reads into key-value pairs"
- [readme] README requirements: "Python: 2.6, 2.7 * Apache Spark: >= 1.6 * HDFS: 2.6"
- [readme] README HDFS rationale: "Using HDFS: Do not need to consider files to be distributed by user."
