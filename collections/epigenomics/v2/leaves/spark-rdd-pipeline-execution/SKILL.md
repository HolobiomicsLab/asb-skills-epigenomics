---
name: spark-rdd-pipeline-execution
description: Use when aligning bisulfite-sequencing reads (FASTQ/A) at genome scale
  on a Spark/HDFS cluster, where single-node aligners are too slow and you need scalable
  'three-letter' alignment with optional load balancing.
when_to_use_negative:
- 'Reads are non-bisulfite DNA-seq: the ''three-letter'' transformation and bisulfite-specific
  filtering are unnecessary and reduce accuracy'
- The dataset is small enough for a single-node aligner (Bismark, BSMAP, RMAP); Spark/HDFS
  cluster setup overhead will not pay off
- No HDFS or Spark cluster (>= 1.6) is available, since BiSpark depends on both plus
  Bowtie2 callable on all slave nodes
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: BiSpark
  role: The distributed bisulfite aligner whose four-phase RDD pipeline this skill
    executes
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed framework that hosts the key-value RDD transformations and portable_hash
    partitioning
  repo: https://spark.apache.org/
- name: HDFS
  role: Shared file system for reference index, input reads, and pipeline outputs
  repo: https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
- name: Bowtie2
  role: Underlying aligner invoked for 'three-letter' read mapping; must be callable
    on all slave nodes
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: Cutadapt
  role: Optional pre-processing to remove adapter sequences before the main BiSpark
    phase
  repo: https://github.com/marcelm/cutadapt/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:a104fb401e08d97185ad6a3544232d44ab54fe9a8ac0cc4386102c94a2c6c66e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - Apache Spark
  - BiSpark
  - Bowtie2
  - Cutadapt
  - HDFS
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# spark-rdd-pipeline-execution

## Summary

Executes BiSpark's four-phase Apache Spark RDD pipeline for bisulfite-treated read alignment: distributing reads into key-value RDDs, 'three-letter' transformation and Bowtie2 mapping, best-alignment aggregation/filtering, and methylation profiling, with plain vs. portable_hash balancing modes.

## When to use

Use when aligning bisulfite-sequencing reads (FASTQ/A) at genome scale on a Spark/HDFS cluster, where single-node aligners are too slow and you need scalable 'three-letter' alignment with optional load balancing.

## When NOT to use

- Reads are non-bisulfite DNA-seq: the 'three-letter' transformation and bisulfite-specific filtering are unnecessary and reduce accuracy
- The dataset is small enough for a single-node aligner (Bismark, BSMAP, RMAP); Spark/HDFS cluster setup overhead will not pay off
- No HDFS or Spark cluster (>= 1.6) is available, since BiSpark depends on both plus Bowtie2 callable on all slave nodes

## Inputs

- FASTQ/A raw sequencing reads (e.g. 100M-read / 16GB WGBS dataset from GEO GSE80911) staged in HDFS
- Reference genome FASTA for index building (e.g. human Build 37 / hg19 chromosome 1)
- HDFS-hosted BiSpark index directory built by ./src/build_index/run.sh

## Outputs

- SAM alignment file written to the --local_save path
- HDFS output directory with alignment results
- Per-read methylation profile (methylRDD) as the pipeline's Phase 4 product
- Local log file from the --log argument

## How to apply

Prepare the reference first: run all four reference genome conversion and indexing steps on the master node (e.g. hg19 Build 37 chromosome 1) and move the index to HDFS for multi-node sharing. Launch the alignment via ./src/alignment/run.sh, giving the HDFS input FASTQ/A, the HDFS index path for --ref, the executor count for --nodes ((# nodes) x (# Spark executors per node)), and a --num_mm mismatch cutoff (default 4); exclude reference transformation/indexing and input-distribution pre-processing from any measured elapsed time. The pipeline then converts reads into key-value RDD tuples, transforms them into 'three-letter' reads mapped against the transformed reference, finds the best alignment while filtering ambiguous reads, and profiles per-read methylation against the original reference. Choose --testmode plain for raw Bowtie2-based throughput or --testmode balancing to enable portable_hash re-partitioning, which trades shuffling overhead for more even node loads on larger clusters. Optionally pre-sanitize reads with Cutadapt adapter removal and FastQC quality inspection to improve mappability and accuracy.

## Related tools

- **BiSpark** (The distributed bisulfite aligner whose four-phase RDD pipeline this skill executes) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed framework that hosts the key-value RDD transformations and portable_hash partitioning) — https://spark.apache.org/
- **HDFS** (Shared file system for reference index, input reads, and pipeline outputs) — https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
- **Bowtie2** (Underlying aligner invoked for 'three-letter' read mapping; must be callable on all slave nodes) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **Cutadapt** (Optional pre-processing to remove adapter sequences before the main BiSpark phase) — https://github.com/marcelm/cutadapt/

## Examples

```
./src/alignment/run.sh --input reads.fastq --ref hdfs://hg19_chr1_idx --nodes 40 --num_mm 4 --testmode plain --appname bispark_chr1
```

## Evaluation signals

- A local SAM file is produced at --local_save and the HDFS output directory is populated for each run
- All four phases execute: key-value RDD conversion, 'three-letter' mapping, best-alignment filtering, and methylation profiling appear in logs
- --testmode accepts only 'balancing' or 'plain'; any other value fails the README contract
- Throughput (reads per second) is positive and computable for each run, enabling scalability plots across 10-, 20-, and 40-node clusters
- Reference indexing output path for build_index equals the --ref path passed to alignment

## Limitations

- Load balancing vs. shuffling is a trade-off that is hard to conclude theoretically; the reason BiSpark-balance is faster on big clusters is speculative
- No software version numbers are stated for BiSpark, Spark, or HDFS beyond README requirements (Python 2.6/2.7, Spark >= 1.6, HDFS 2.6)
- Bowtie2 must be callable on every slave node; missing per-node installation breaks the pipeline

## Evidence

- [readme] Analysis workflow within BiSpark consists of 4 processing phases: (1) Distributing the reads into key-value pairs, (2) Transforming reads into ‘three-letter’ reads and mapping to transformed reference genome: "Analysis workflow within BiSpark consists of 4 processing phases: (1) Distributing the reads into key-value pairs, (2) Transforming reads into ‘three-letter’ reads and mapping to transformed"
- [methods] the BiSpark accepts raw sequencing data files, FASTQ/A format, as inputs and converts them into list of key-value structured tuples: "the BiSpark accepts raw sequencing data files, FASTQ/A format, as inputs and converts them into list of key-value structured tuples"
- [methods] all four reference genome conversion and indexing are performed in the master node first and moved to HDFS for multi-node sharing: "all four reference genome conversion and indexing are performed in the master node first and moved to HDFS for multi-node sharing"
- [methods] the BiSpark utilizes portable_hash function, supported by Spark framework, to determine which partition the data should be placed: "the BiSpark utilizes portable_hash function, supported by Spark framework, to determine which partition the data should be placed"
- [readme] BiSpark is implemented on [Apache Spark](https://spark.apache.org/) framework and [HDFS](https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html) file system.: "BiSpark is implemented on [Apache Spark](https://spark.apache.org/) framework and [HDFS](https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html) file system."
