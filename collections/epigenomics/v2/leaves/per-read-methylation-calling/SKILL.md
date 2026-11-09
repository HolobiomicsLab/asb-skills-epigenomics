---
name: per-read-methylation-calling
description: Use when bisulfite-treated reads have been aligned with the 'three-letter'
  approach and you need per-read/per-site methylation states recovered by comparing
  alignments back to the original (un-converted) reference genome.
when_to_use_negative:
- Alignments have not yet passed Phase 3 filtering — ambiguous multi-best hits and
  over-cutoff mismatches must be removed before methylation estimation
- Only a converted ('three-letter') reference is available — methylation profiling
  requires comparison against the original un-converted genome sequence
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: BiSpark
  role: 'Implements Phase 4: merges sequences and mappings into mergedRDD and computes
    methylRDD by comparing filteredRDD with the original reference genome'
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed framework providing RDDs and broadcast shared variables used to
    hold the original reference genome during methylation profiling
- name: Bowtie2
  role: Aligner producing the mapRDDs against the four converted reference genomes
    whose filtered results feed methylation profiling
- name: HDFS
  role: Stores the four converted/indexed hg19 reference genomes shared across the
    cluster before methylation profiling
  repo: https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:053c70bbb0203a3551c7cf8f3c24aeee921fd83beb93f6b3093b4bfb620f6c8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - Apache Spark
  - BiSpark
  - Bowtie2
  - HDFS
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# per-read-methylation-calling

## Summary

Phase 4 of the BiSpark workflow: estimate per-site methylation by merging filtered alignments with original read sequences and comparing them against the un-converted reference genome, broadcast via Spark shared variables, to produce the methylRDD methylation profile.

## When to use

Use when bisulfite-treated reads have been aligned with the 'three-letter' approach and you need per-read/per-site methylation states recovered by comparing alignments back to the original (un-converted) reference genome.

## When NOT to use

- Alignments have not yet passed Phase 3 filtering — ambiguous multi-best hits and over-cutoff mismatches must be removed before methylation estimation
- Only a converted ('three-letter') reference is available — methylation profiling requires comparison against the original un-converted genome sequence

## Inputs

- filteredRDD: filtered alignment tuples (best unique alignments after mismatch-cutoff filtering, default 4 mismatches)
- Original read sequences (from readRDD, FASTQ/A input)
- Original (un-converted) hg19 reference genome, broadcast via Spark shared variables

## Outputs

- methylRDD: key-value RDD with per-site methylation information estimated per read
- SAM-format methylation/mapping output file written to local disk

## How to apply

After Phase 3 yields filteredRDD (alignments with the uniquely smallest mismatch count, ambiguous multi-best hits discarded and results exceeding the user-defined mismatch cutoff, default 4, removed), merge read sequences and mapping information into mergedRDD. Broadcast the original hg19 reference genome via Spark shared variables so it is available on all executors without re-distribution. Compare each filteredRDD alignment against the original (non-bisulfite-converted) reference sequence to estimate methylation per cytosine site, storing the result as the value in methylRDD. Finally write methylRDD in SAM format to the output file alongside serialized intermediate artifacts (readRDD, transRDD, mapRDD, filteredRDD). Correctness on the 0%-error Sherman simulation (chr1, 95 bp, 1,000,000 reads) is judged by reproducing mappability 0.9569 with precision 1.0.

## Related tools

- **BiSpark** (Implements Phase 4: merges sequences and mappings into mergedRDD and computes methylRDD by comparing filteredRDD with the original reference genome) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed framework providing RDDs and broadcast shared variables used to hold the original reference genome during methylation profiling)
- **Bowtie2** (Aligner producing the mapRDDs against the four converted reference genomes whose filtered results feed methylation profiling)
- **HDFS** (Stores the four converted/indexed hg19 reference genomes shared across the cluster before methylation profiling) — https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html

## Examples

```
./src/alignment/run.sh --input reads.fastq --output out_dir --log run.log --ref hg19_index --nodes 4 --num_mm 4 --local_save methyl_out.sam --testmode plain --appname methylation_call
```

## Evaluation signals

- On the 0%-error Sherman simulation (human chromosome 1, 95 bp, 1,000,000 reads), mappability of 0.9569 and precision 1.0 should be reproduced (identical for BiSpark-plain and BiSpark-balance)
- methylRDD values must be derivable only from filteredRDD alignments compared against the original, not converted, reference sequence
- Final output must be written in SAM format; intermediates readRDD, transRDD, mapRDD and filteredRDD should serialize cleanly
- Ambiguous multi-best hits and alignments exceeding the mismatch cutoff (default 4) must not contribute methylation calls

## Limitations

- No theoretical conclusion on whether BiSpark-balance improves performance; load balancing vs. shuffling is an unresolved trade-off in terms of speed
- No software version numbers stated for BiSpark, Spark, or HDFS in the article, complicating exact reproduction
- Recommended but optional upstream QC (Cutadapt adapter removal, FastQC inspection) is not enforced, so poor-quality reads entering Phase 4 can degrade methylation estimates

## Evidence

- [methods] Phase 4: Methylation Profiling: "methylRDD has the methylation information, estimated by comparing filteredRDD with the original reference genome sequence, as the value"
- [intro] Four-phase workflow: "(4) Profiling the methylation information for each read"
- [methods] Filtering before profiling: "the BiSpark also supports a user-defined mismatch cutoff to adjust the intensity of the restriction"
- [methods] Validation metric: "the BiSpark shows the best performance on all four metrics with the 0% error dataset"
- [readme] README workflow summary: "Analysis workflow within BiSpark consists of 4 processing phases: (1) Distributing the reads into key-value pairs, (2) Transforming reads into ‘three-letter’ reads"
