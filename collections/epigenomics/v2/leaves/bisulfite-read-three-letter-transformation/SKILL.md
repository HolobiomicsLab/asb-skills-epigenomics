---
name: bisulfite-read-three-letter-transformation
description: Use when aligning bisulfite-treated sequencing reads (FASTQ/A) under
  the 'three-letter' approach and you need to transform reads and reference genomes
  into CT/GA-converted variants before mapping.
when_to_use_negative:
- The aligner uses a wildcard approach (BSMAP, RMAP style) where ambiguous bisulfite-treated
  nucleotides are wildcard-matched instead of pre-converted
- Reads are ordinary (non-bisulfite) sequencing data; the three-letter transformation
  would needlessly distort the sequences
- Only a single-node, small dataset without scalability needs; the Spark/HDFS distributed
  overhead of this phase may not pay off
edam_operation: http://edamontology.org/operation_3185
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: BiSpark
  role: Implements the three-letter transformation of reads into transRDD (CTtransRDD/GAtransRDD)
    and drives the distributed mapping phase
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed framework holding readRDD/transRDD as key-value RDDs partitioned
    across cluster nodes
- name: Bowtie2
  role: Aligns each three-letter read variant against the correspondingly converted
    and indexed reference genomes; must be callable on all slave nodes
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: HDFS
  role: Stores the four converted and indexed reference genomes for multi-node sharing
  repo: https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:f15835e0f3c93b89bb1779a09bbf876b8922f759b775f1cd748e784994726f13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - Apache Spark
  - BiSpark
  - Bowtie2
  - HDFS
  edam_operation: http://edamontology.org/operation_3185
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bisulfite-read-three-letter-transformation

## Summary

Phase 2 of the BiSpark workflow: convert bisulfite-treated reads into CT- and GA-converted 'three-letter' variants and align each variant with Bowtie2 against correspondingly converted and indexed reference genomes. This handles the up-to-4x data-volume inflation inherent to the 'three-letter' bisulfite alignment approach.

## When to use

Use when aligning bisulfite-treated sequencing reads (FASTQ/A) under the 'three-letter' approach and you need to transform reads and reference genomes into CT/GA-converted variants before mapping.

## When NOT to use

- The aligner uses a wildcard approach (BSMAP, RMAP style) where ambiguous bisulfite-treated nucleotides are wildcard-matched instead of pre-converted
- Reads are ordinary (non-bisulfite) sequencing data; the three-letter transformation would needlessly distort the sequences
- Only a single-node, small dataset without scalability needs; the Spark/HDFS distributed overhead of this phase may not pay off

## Inputs

- Bisulfite-treated sequencing reads in FASTQ/A format (e.g. Sherman-simulated human chromosome 1, 95 bp, 1,000,000 reads)
- Reference genome FASTA (human Build 37, hg19), four converted and Bowtie2-indexed variants stored in HDFS

## Outputs

- transRDD of <read id, transformed read sequence> tuples (CTtransRDD and GAtransRDD)
- mapRDDs of Bowtie2 mapping results for the W-CT, W-GA, C-CT and C-GA combinations

## How to apply

Load raw FASTQ/A reads into Spark key-value tuples (<read id, read sequence>) as readRDD (Phase 1), then transform each read into transRDD sub-categorized into CTtransRDD and GAtransRDD three-letter variants. Prepare four reference genome conversions and index them with Bowtie2 on the master node first, then move them to HDFS for multi-node sharing. Align each read variant against its correspondingly converted hg19 reference index, producing four mapRDDs for the W-CT, W-GA, C-CT and C-GA strand-conversion combinations. The rationale is that bisulfite treatment converts unmethylated cytosines, so transforming both reads and reference into three-letter space lets a standard aligner (Bowtie2) determine the origin of bisulfite-treated nucleotides without wildcard matching.

## Related tools

- **BiSpark** (Implements the three-letter transformation of reads into transRDD (CTtransRDD/GAtransRDD) and drives the distributed mapping phase) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed framework holding readRDD/transRDD as key-value RDDs partitioned across cluster nodes)
- **Bowtie2** (Aligns each three-letter read variant against the correspondingly converted and indexed reference genomes; must be callable on all slave nodes) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **HDFS** (Stores the four converted and indexed reference genomes for multi-node sharing) — https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html

## Examples

```
./src/build_index/run.sh --input hg19_chr1.fa --output hg19_chr1_index --log build_index.log  # then ./src/alignment/run.sh --ref hg19_chr1_index --num_mm 4 --nodes <n> --local_save out.sam --testmode plain
```

## Evaluation signals

- On the 0%-error Sherman simulation (chr1, 95 bp, 1M reads) the downstream pipeline should reproduce mappability 0.9569 with precision 1.0, identical for BiSpark-plain and BiSpark-balance
- transRDD must contain both CT- and GA-converted variants for every read id, split into CTtransRDD and GAtransRDD
- Exactly four mapRDDs (W-CT, W-GA, C-CT, C-GA) are produced when the input library is non-directional
- The four converted, Bowtie2-indexed reference genomes must reside in HDFS and be accessible from all slave nodes

## Limitations

- The three-letter approach inflates data volume up to four times, which is the computational bottleneck of methylome analysis that BiSpark parallelizes but does not eliminate
- No software version numbers stated for BiSpark, Spark, or HDFS in the discussion; README requirements are Python 2.6/2.7, Spark >= 1.6, HDFS 2.6
- Mapping quality depends on the later mismatch cutoff (default 4) applied in Phase 3, not on this transformation phase alone

## Evidence

- [intro] Transforming reads into 'three-letter' reads and mapping to transformed reference genome: "(2) Transforming reads into ‘three-letter’ reads and mapping to transformed reference genome"
- [methods] Phase 2 transforms readRDDs to transRDD: "the BiSpark transforms readRDDs to transRDD which consists of <read id, transformed read sequence > tuples"
- [methods] Four reference genome conversions indexed on master node then moved to HDFS: "all four reference genome conversion and indexing are performed in the master node first and moved to HDFS for multi-node sharing"
- [intro] Bismark and BSSeeker use the three-letter approach to determine the origin of bisulfite-treated nucleotides: "Bismark [3] and BSSeeker [4] use the ‘three-letter’ approach [5] to determine the origin of bisulfite-treated nucleotides"
- [readme] README: workflow phase 2 description: "(2) Transforming reads into ‘three-letter’ reads and mapping to transformed reference genome"
