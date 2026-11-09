---
name: three-letter-reference-genome-conversion
description: 'Use when preparing a reference genome for bisulfite read alignment with
  a ''three-letter'' aligner (BiSpark, Bismark-style): an unconverted reference FASTA
  must be transformed into CT/GA strand versions and indexed before mapping.'
when_to_use_negative:
- The aligner uses the wildcard approach (BSMAP, RMAP) on an unconverted genome rather
  than the 'three-letter' approach
- The task is aligning reads — conversion/indexing must precede alignment (Phase 2),
  not replace it
- The reference is already a converted three-letter genome or its Bowtie2 index already
  exists in HDFS
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0654
tools:
- name: BiSpark
  role: Aligner whose three-letter approach motivates the four-fold genome conversion;
    provides the build_index workflow
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Bowtie2
  role: Indexes each of the four converted 'three-letter' genomes for the mapping
    phase; must be callable on all slave nodes
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: HDFS
  role: Distributed store where the converted, indexed genomes are moved for multi-node
    sharing during alignment
  repo: https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:8b20a98fda4b63d685f96cba129b79cb3d63012abbafdfa988ef84ba8f305c77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - BiSpark
  - Bowtie2
  - HDFS
  - Sherman
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# three-letter-reference-genome-conversion

## Summary

Converts a bisulfite sequencing reference genome (e.g. hg19 chromosome 1 FASTA) into the four 'three-letter' transformed strands (W-CT, W-GA, C-CT, C-GA) required by three-letter aligners like BiSpark, then indexes them with Bowtie2 and stages the indices in HDFS for multi-node sharing.

## When to use

Use when preparing a reference genome for bisulfite read alignment with a 'three-letter' aligner (BiSpark, Bismark-style): an unconverted reference FASTA must be transformed into CT/GA strand versions and indexed before mapping.

## When NOT to use

- The aligner uses the wildcard approach (BSMAP, RMAP) on an unconverted genome rather than the 'three-letter' approach
- The task is aligning reads — conversion/indexing must precede alignment (Phase 2), not replace it
- The reference is already a converted three-letter genome or its Bowtie2 index already exists in HDFS

## Inputs

- Reference genome FASTA (e.g. human reference genome Build 37 hg19, chromosome 1)
- Library type (directional vs non-directional, determines whether all four strand conversions are used)

## Outputs

- Four converted 'three-letter' genomes: W-CT, W-GA, C-CT, C-GA
- Bowtie2 index files for each converted genome
- Indexed genomes staged in HDFS directories for distributed access

## How to apply

Take the reference FASTA (e.g. human Build 37 hg19, chromosome 1) and generate all four customized 'three-letter' genome conversions representing Watson/CRICK strands with C-to-T and G-to-A transformations, matching the four possible origins of bisulfite-treated nucleotides. Index each converted genome with Bowtie2 so reads from the CT and GA transRDDs can be aligned in BiSpark Phase 2. Per the BiSpark genome-preparation step, perform conversion and indexing in the master node first, then move the indexed genomes to HDFS for multi-node sharing (the README's build_index step takes the input FASTA from and writes indices to HDFS directories). Keep the '--ref' alignment parameter pointing to the same HDFS path used as the build_index '--output' so alignment resolves the correct converted indices. For non-directional library mode, all four strand conversions are needed since reads may originate from either strand after bisulfite treatment.

## Related tools

- **BiSpark** (Aligner whose three-letter approach motivates the four-fold genome conversion; provides the build_index workflow) — https://github.com/bhi-kimlab/BiSpark/
- **Bowtie2** (Indexes each of the four converted 'three-letter' genomes for the mapping phase; must be callable on all slave nodes) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **HDFS** (Distributed store where the converted, indexed genomes are moved for multi-node sharing during alignment) — https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html

## Examples

```
./src/build_index/run.sh --input hdfs:///ref/hg19.chr1.fa --output hdfs:///ref/bispark_index --log build_index.log
```

## Evaluation signals

- Exactly four converted genomes are produced (W-CT, W-GA, C-CT, C-GA), one per strand × transformation combination
- Each converted genome has a complete, usable Bowtie2 index before alignment starts
- All indexed genomes are present in HDFS and accessible from slave nodes, and the alignment '--ref' path matches the build_index '--output' path
- Downstream alignment (Phase 2) resolves each transRDD (CT and GA) to its corresponding converted genome without missing-index errors
- Non-directional mode mappings draw on all four mapRDDs (W-CT, W-GA, C-CT, C-GA) before union into combRDD

## Limitations

- No software version numbers are stated for BiSpark, Spark, or HDFS, so reproducibility of the exact conversion environment is unverified
- Conversion and indexing are performed only in the master node first, so the master's capacity bounds genome size for this step
- The approach is specific to 'three-letter' bisulfite alignment (up to four times more data volume) and is not appropriate for wildcard-style aligners

## Evidence

- [methods] all four reference genome conversion and indexing are performed in the master node first and moved to HDFS for multi-node sharing: "all four reference genome conversion and indexing are performed in the master node first and moved to HDFS for multi-node sharing"
- [intro] three-letter origin determination: "Bismark [3] and BSSeeker [4] use the ‘three-letter’ approach [5] to determine the origin of bisulfite-treated nucleotides"
- [readme] build_index input flag: "--input: Input reference FASTA file located in HDFS."
- [readme] ref index path consistency: "--ref: Index dicrectory in HDFS. Should be same path as denoted as '--output'."
