---
name: bisulfite-read-alignment
description: Use when WGBS/bisulfite FASTQ reads must be aligned to a reference genome
  (e.g. hg19) at large scale (10M–200M reads) where single-node 'three-letter' aligners
  are too slow and a distributed scale-out approach is needed.
when_to_use_negative:
- When a small, single-server bisulfite alignment suffices — a scale-up aligner like
  Bismark (5-multicore) on a 24-core server avoids Spark/HDFS cluster setup overhead
- When reads are not bisulfite-treated (ordinary DNA-seq or RNA-seq) — the CT/GA 'three-letter'
  conversion would incorrectly transform them and inflate the data volume up to fourfold
- When raw reads contain adapter contamination and low-quality tails and no pre-processing
  is planned — the README notes quality control is required to avoid degraded mappability
  and accuracy
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0157
tools:
- name: BiSpark
  role: 'The bisulfite aligner itself: orchestrates the four-phase Spark workflow
    (readRDD, transRDD three-letter mapping, best-alignment filtering, methylation
    profiling) in plain or balancing (portable_hash) modes'
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed, memory-optimized framework on which BiSpark's key-value RDD transformations
    and mapRDD unions are executed
  repo: https://spark.apache.org/
- name: Bowtie2
  role: Underlying aligner invoked by BiSpark to map CT/GA-transformed reads against
    the four customized indexed genomes; must be callable on all slave nodes
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: Cutadapt
  role: Recommended optional pre-processing to remove adaptor sequences from reads
    before alignment to improve mappability and alignment accuracy
  repo: https://github.com/marcelm/cutadapt/
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:d5cb6190e12161dd7488c2f277d59e075d1e9e9e063de459fbe5cf9bed614e7a
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
  - Cutadapt
  - Sherman
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# bisulfite-read-alignment

## Summary

Align bisulfite-treated sequencing reads (e.g. WGBS FASTQ) using the 'three-letter' transform-and-map strategy implemented as a parallelized Spark/Bowtie2 pipeline (BiSpark), for scalable methylome read alignment with metrics such as mappability, sensitivity, precision, and accuracy.

## When to use

Use when WGBS/bisulfite FASTQ reads must be aligned to a reference genome (e.g. hg19) at large scale (10M–200M reads) where single-node 'three-letter' aligners are too slow and a distributed scale-out approach is needed.

## When NOT to use

- When a small, single-server bisulfite alignment suffices — a scale-up aligner like Bismark (5-multicore) on a 24-core server avoids Spark/HDFS cluster setup overhead
- When reads are not bisulfite-treated (ordinary DNA-seq or RNA-seq) — the CT/GA 'three-letter' conversion would incorrectly transform them and inflate the data volume up to fourfold
- When raw reads contain adapter contamination and low-quality tails and no pre-processing is planned — the README notes quality control is required to avoid degraded mappability and accuracy

## Inputs

- Bisulfite-treated sequencing reads in FASTQ or FASTA format (e.g. WGBS GSE80911 95 bp reads, or Sherman-simulated 95 bp reads)
- Reference genome FASTA (human Build 37 / hg19 chromosome 1) located in HDFS
- BiSpark Bowtie2 index directory in HDFS produced by build_index (four CT/GA-converted genomes)

## Outputs

- SAM alignment file (--local_save local output path)
- Best-alignment filteredRDD reduced to unique-best-mismatch reads, plus per-read methylation information (methylRDD) against the original reference
- Benchmark outputs: mapping-quality metrics (mappability, precision, sensitivity, accuracy) per error rate and alignment runtimes/throughput per dataset size and cluster size

## How to apply

Convert the reference (e.g. hg19 Build 37 chromosome 1) FASTA into the four customized 'three-letter' genomes (W-CT, W-GA, C-CT, C-GA), index each with Bowtie2 via BiSpark's build_index step, and stage the indexes in HDFS. Load reads (FASTQ/A, e.g. 95 bp reads from GSE80911 or Sherman simulations) into a readRDD of (read id, sequence) key-value tuples, transform into CT and GA transRDDs, and map each against the corresponding customized genome with Bowtie2 under a user-defined mismatch cutoff (default 4) in the appropriate library mode (non-directional). Union the four mapRDDs without shuffling, combine by read id, and keep only reads with a uniquely smallest mismatch count as the best alignment, discarding ambiguous alignments into a filteredRDD. Optionally enable BiSpark-balance (portable_hash repartitioning) for large clusters (e.g. 40 nodes) where load redistribution outweighs shuffling cost, and exclude reference pre-processing time from measured alignment runtime.

## Related tools

- **BiSpark** (The bisulfite aligner itself: orchestrates the four-phase Spark workflow (readRDD, transRDD three-letter mapping, best-alignment filtering, methylation profiling) in plain or balancing (portable_hash) modes) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed, memory-optimized framework on which BiSpark's key-value RDD transformations and mapRDD unions are executed) — https://spark.apache.org/
- **Bowtie2** (Underlying aligner invoked by BiSpark to map CT/GA-transformed reads against the four customized indexed genomes; must be callable on all slave nodes) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **Cutadapt** (Recommended optional pre-processing to remove adaptor sequences from reads before alignment to improve mappability and alignment accuracy) — https://github.com/marcelm/cutadapt/

## Examples

```
./src/alignment/run.sh --input hdfs:///data/GSE80911_10M.fq --ref hdfs:///index/hg19_chr1 --output hdfs:///out/10M --log /logs/align.log --nodes 20 --num_mm 4 --local_save /out/aligned.sam --testmode plain --appname BiSpark-10M
```

## Evaluation signals

- Mapping-quality metrics on Sherman-simulated 95 bp, 1,000,000-read sets match Table 3 BiSpark rows: 0% error mappability/sensitivity 0.9569, precision 1.0, accuracy 0.9569; 1% error mappability 0.9494, sensitivity 0.9489, accuracy 0.9392; 2% error mappability 0.9422, sensitivity 0.9411, accuracy 0.9234
- BiSpark-plain and BiSpark-balance produce identical mapping-quality results, differing only in runtime/throughput
- Runtime sanity check: BiSpark-plain aligns 10M WGBS reads on 20 nodes in approximately 617 s (vs CloudAligner 14,783 s), ~2.5x faster than Bismark at 10M and ~3.5x at 200M reads, and ~22% faster than Bison at 200M reads
- Throughput scaling check on a 100M-read input: BiSpark-balance shows lower throughput than plain at 10 and 20 nodes but higher throughput at 40 nodes, with all throughput values positive
- Only reads with a uniquely smallest mismatch count within the mismatch cutoff (default --num_mm 4) survive into the output SAM; ambiguous multi-best alignments are discarded

## Limitations

- The trade-off between load balancing and shuffling is not resolved theoretically; BiSpark-balance is relatively slower on small clusters (10, 20 nodes) and only gains at larger cluster sizes, and the authors' explanation (more nodes waiting on the slowest node) is speculative
- Measured elapsed times must exclude reference transformation/indexing and input-distribution pre-processing, so runtime comparisons to other tools require identical accounting conventions
- No software version numbers are stated for BiSpark, Spark, or HDFS in the article; README requirements pin Python 2.6/2.7, Apache Spark >= 1.6, and HDFS 2.6
- Bowtie2 must be callable on all slave nodes, and reference indexes must be pre-staged in HDFS; misconfiguration degrades or blocks alignment

## Evidence

- [methods] Four-phase Spark workflow: "the BiSpark transforms readRDDs to transRDD which consists of <read id, transformed read sequence > tuples"
- [methods] Phase 3 filtering: "the next step of the analysis is figuring out the best alignment among these results"
- [methods] Mismatch cutoff parameter: "the BiSpark also supports a user-defined mismatch cutoff to adjust the intensity of the restriction"
- [methods] HDFS reference staging: "all four reference genome conversion and indexing are performed in the master node first and moved to HDFS for multi-node sharing"
- [results] Runtime benchmark: "BiSpark-plain only took 617 s and this is around more than 20 times faster than CloudAligner that took 14,783 s"
- [discussion] Balancing trade-off: "Since load balancing and shuffling are a trade-off in terms of the speed, it is hard to conclude theoretically whether the performance would be improved or not."
- [readme] Alignment CLI parameters: "--testmode: Switching testmode for experiment. Should be one of 'balancing' or 'plain'."
- [readme] Quality-control recommendation: "Run Cutadapt (https://github.com/marcelm/cutadapt/) to remove adaptor sequence from the reads."
- [readme] Cluster prerequisites: "Bowtie2 should be callable on all slave nodes."
