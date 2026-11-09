---
name: genome-index-preparation-three-letter-conversion
description: Use when aligning bisulfite-treated reads (e.g. WGBS FASTQ against hg19)
  with BiSpark, since the 'three-letter' aligner requires a pre-converted, Bowtie2-indexed
  reference stored in HDFS before alignment can start.
when_to_use_negative:
- Using a single-node aligner such as Bismark that performs its own bisulfite conversions
  internally — this HDFS-based index preparation is specific to BiSpark.
- The reference genome is already provided as a pre-built bisulfite index from another
  tool; BiSpark's build_index performs its own four conversions and Bowtie2 indexing.
- Only simulating or benchmarking alignment timing where index construction must be
  excluded — build the index beforehand so runtime is measured only on the alignment
  phase.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: BiSpark
  role: Provides the build_index script that performs the four 'three-letter' CT/GA
    reference conversions and Bowtie2 indexing, storing results in HDFS
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Bowtie2
  role: Used by BiSpark to index the converted bisulfite reference genomes; must be
    callable on all slave nodes
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: Apache Spark
  role: Distributed framework on which BiSpark runs; the index in HDFS is shared with
    all Spark executors
- name: HDFS
  role: Distributed file system that stores the converted and indexed reference for
    multi-node sharing
  repo: https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:98f70a9692f7b044611128d1559ef4355d2d93d156437371fd15f7c844eab133
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# genome-index-preparation-three-letter-conversion

## Summary

Prepare a bisulfite-aware reference genome index for BiSpark by performing the four CT/GA conversions of the 'three-letter' algorithm and indexing with Bowtie2, then storing the converted index in HDFS for multi-node sharing across the Spark cluster.

## When to use

Use when aligning bisulfite-treated reads (e.g. WGBS FASTQ against hg19) with BiSpark, since the 'three-letter' aligner requires a pre-converted, Bowtie2-indexed reference stored in HDFS before alignment can start.

## When NOT to use

- Using a single-node aligner such as Bismark that performs its own bisulfite conversions internally — this HDFS-based index preparation is specific to BiSpark.
- The reference genome is already provided as a pre-built bisulfite index from another tool; BiSpark's build_index performs its own four conversions and Bowtie2 indexing.
- Only simulating or benchmarking alignment timing where index construction must be excluded — build the index beforehand so runtime is measured only on the alignment phase.

## Inputs

- Reference genome FASTA file located in HDFS (e.g. human Build 37, hg19)

## Outputs

- Four CT/GA-converted, Bowtie2-indexed reference genomes stored in an HDFS output directory (usable as --ref for BiSpark alignment)

## How to apply

Download the reference genome FASTA (e.g. human Build 37, hg19) and run BiSpark's build_index step on the master node: it performs all four reference genome conversions of the 'three-letter' algorithm (CT/GA variants covering both reads and reference) and indexes each with Bowtie2. The converted and indexed references are then moved to HDFS so all slave nodes share them without user-side file distribution. Exclude this genome-preparation time from any downstream alignment runtime measurements, as done in the GSE80911 benchmark against CloudAligner, Bison, and Bismark. Reference the prepared index via the --ref argument, which must match the --output directory used at index-build time. Bowtie2 must be callable on all slave nodes for the index to be usable during the mapping phase.

## Related tools

- **BiSpark** (Provides the build_index script that performs the four 'three-letter' CT/GA reference conversions and Bowtie2 indexing, storing results in HDFS) — https://github.com/bhi-kimlab/BiSpark/
- **Bowtie2** (Used by BiSpark to index the converted bisulfite reference genomes; must be callable on all slave nodes) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **Apache Spark** (Distributed framework on which BiSpark runs; the index in HDFS is shared with all Spark executors)
- **HDFS** (Distributed file system that stores the converted and indexed reference for multi-node sharing) — https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html

## Examples

```
sh ./src/build_index/run.sh --input hdfs:///ref/hg19.fa --output hdfs:///index/hg19_bisc --log ./logs/build_index.log
```

## Evaluation signals

- The build_index output directory contains indexes for all four CT/GA-converted reference variants.
- The --ref argument of the alignment step points to the same HDFS path as the build_index --output directory.
- Bowtie2 is confirmed callable on every slave node before alignment is launched.
- Alignment benchmark runtimes (e.g. 617 s for 10M WGBS reads on 20 nodes) exclude genome-preparation time, so the index was built prior to timing.
- Index construction is done on the master node first and validated as accessible in HDFS before distributing to the cluster.

## Limitations

- Genome preparation time is excluded from BiSpark's reported alignment speedups, so the index-building step adds wall-clock overhead users must budget for.
- Reference conversions and Bowtie2 indexing are performed on the master node first, which may become a bottleneck for very large genomes before the index is moved to HDFS.
- BiSpark requires Python 2.6/2.7, Apache Spark >= 1.6, and HDFS 2.6; Bowtie2 must be installed and callable on all slave nodes for the prepared index to be usable.
- Only the alignments and conversions described for the 'three-letter' algorithm are supported; wildcard-based bisulfite approaches (e.g. BSMAP-style) are not covered by this index preparation.

## Evidence

- [methods] Genome Preparation: "all four reference genome conversion and indexing are performed in the master node first and moved to HDFS for multi-node sharing"
- [readme] build_index usage: "Building Index (reference: ./src/build_index/run.sh) 1. --input: Input reference FASTA file located in HDFS. 2. --output: Output directory for indexing files in HDFS."
- [readme] Bowtie2 requirement: "[Bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml) is also used to alignment, thus all three frameworks and programs should be installed before running BiSpark. Bowtie2 should be"
- [intro] three-letter conversion rationale: [evidence span withheld — anchor status: not_found]
