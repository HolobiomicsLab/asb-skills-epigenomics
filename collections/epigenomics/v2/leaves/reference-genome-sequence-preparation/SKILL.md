---
name: reference-genome-sequence-preparation
description: Use when raw FASTQ reads must be aligned with BiSpark, or simulated bisulfite
  reads must be generated, and the reference genome (hg19 Build 37) has not yet been
  converted, indexed, or placed in HDFS.
when_to_use_negative:
- The reads will be aligned with Bismark, BSSeeker, BSMAP, or another aligner that
  performs its own genome conversion/indexing (BiSpark's 'three-letter' conversion
  indexes are not interchangeable).
- Only methylation profiling is needed and a previously built HDFS index with a known
  --ref path already exists.
- The genome build required is not hg19/Build 37 and the evaluation assumptions (95
  bp reads, chromosome 1) do not apply to the target organism.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: BiSpark
  role: 'Performs the Genome Preparation step: reference genome conversion and indexing
    on the master node, then moves indexes to HDFS; provides ./src/build_index/run.sh'
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed framework over which the BiSpark index/conversion routines are
    implemented
- name: HDFS
  role: Distributed file system used to share the prepared reference index across
    cluster nodes
- name: Sherman
  role: Bisulfite-treated read FastQ simulator run on the prepared chromosome 1 sequence
    to generate 95 bp, 1,000,000-read test sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:a0e90fbbcd7e72f4a36ee3774093c223fc905e75255c71301519672495613138
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - Apache Spark
  - BiSpark
  - HDFS
  - Sherman
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# reference-genome-sequence-preparation

## Summary

Prepares a bisulfite-ready reference genome for BiSpark by converting and indexing the chosen genome build (e.g., human Build 37 / hg19) on a master node and staging the index in HDFS, or by extracting the target chromosome (e.g., chromosome 1) for simulation with Sherman.

## When to use

Use when raw FASTQ reads must be aligned with BiSpark, or simulated bisulfite reads must be generated, and the reference genome (hg19 Build 37) has not yet been converted, indexed, or placed in HDFS.

## When NOT to use

- The reads will be aligned with Bismark, BSSeeker, BSMAP, or another aligner that performs its own genome conversion/indexing (BiSpark's 'three-letter' conversion indexes are not interchangeable).
- Only methylation profiling is needed and a previously built HDFS index with a known --ref path already exists.
- The genome build required is not hg19/Build 37 and the evaluation assumptions (95 bp reads, chromosome 1) do not apply to the target organism.

## Inputs

- Human reference genome FASTA (Build 37, hg19), located in HDFS
- Target chromosome sequence (e.g., human chromosome 1 of hg19) for simulation workflows

## Outputs

- BiSpark index files (four reference genome conversions/indexings) stored in an HDFS output directory
- Local log file from the indexing run
- Chromosome 1 FASTA suitable as Sherman bisulfite read-simulation input

## How to apply

Obtain the human reference genome Build 37 (hg19); for simulation-based benchmarks extract only human chromosome 1, since BiSpark's evaluation data were simulated with Sherman using chromosome 1, 95 bp reads, and 1,000,000 reads. Run BiSpark's Genome Preparation step: all four reference genome conversions and indexing (performed for the 'three-letter' approach) are executed first on the master node, then moved to HDFS for multi-node sharing. Use the build_index script in the BiSpark repository, supplying the input FASTA in HDFS, an output directory for index files in HDFS, and a local log path; the same path must later be passed as --ref to the alignment step. Verify the index directory contents before launching alignment, since the alignment --ref parameter must match the build_index --output path exactly.

## Related tools

- **BiSpark** (Performs the Genome Preparation step: reference genome conversion and indexing on the master node, then moves indexes to HDFS; provides ./src/build_index/run.sh) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed framework over which the BiSpark index/conversion routines are implemented)
- **HDFS** (Distributed file system used to share the prepared reference index across cluster nodes)
- **Sherman** (Bisulfite-treated read FastQ simulator run on the prepared chromosome 1 sequence to generate 95 bp, 1,000,000-read test sets)

## Examples

```
./src/build_index/run.sh --input hdfs:///ref/hg19.fa --output hdfs:///ref/hg19_index --log ./build_index.log
```

## Evaluation signals

- The HDFS index output directory exists and its path matches the --ref value supplied to the alignment step.
- The indexing run completed without error and produced a non-empty local log file at the --log path.
- For simulation use, the extracted chromosome 1 FASTA corresponds to hg19 Build 37 coordinates so that simulated read statistics match those reported in Table 1 (122 MB per read set).
- Downstream alignment on the index returns key-value read alignments consistent with the four-phase BiSpark workflow, i.e. mapping succeeded rather than failing on a missing/mismatched index.
- All four reference genome conversions/indexings used by the 'three-letter' approach were completed before alignment, not just one.

## Limitations

- No software version numbers are stated for BiSpark, Spark, or HDFS in the article, so reproducing the exact index build may require contacting the authors or inspecting the repository.
- The prepared index must be relocated from the master node to HDFS; forgetting this step breaks multi-node sharing since all slave nodes must see the reference.
- README requirements constrain the environment: Python 2.6/2.7, Apache Spark >= 1.6, HDFS 2.6, and Bowtie2 callable on all slave nodes.
- Only human hg19 (Build 37) is documented as validated; other genome builds are untested in this context.

## Evidence

- [methods] Genome Preparation step: "all four reference genome conversion and indexing are performed in the master node first and moved to HDFS for multi-node sharing"
- [methods] Evaluation reference genome: "we used human reference genome (ver. Build 37, hg19)"
- [methods] Simulation configuration: "setting up with human chromosome 1, read length to 95bp, and the number of reads to 1,000,000"
- [readme] build_index script parameters: [evidence span withheld — anchor status: not_found]
- [readme] Alignment must reuse the index path: "--ref: Index dicrectory in HDFS. Should be same path as denoted as '--output'."
