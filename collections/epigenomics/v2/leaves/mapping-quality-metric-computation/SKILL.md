---
name: mapping-quality-metric-computation
description: Use when you have simulated bisulfite-seq reads (e.g., Sherman FASTQ
  with embedded original positions) plus a filtered alignment (SAM), and need to quantify
  mappability, sensitivity, precision, and accuracy per error rate.
when_to_use_negative:
- Real WGBS data without embedded ground-truth positions — true positives cannot be
  determined, so precision/sensitivity/accuracy are not computable (methylation profiling,
  not evaluation, is the goal)
- The goal is alignment speed or scalability benchmarking against CloudAligner, Bismark,
  or Bison rather than mapping-quality measurement
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: BiSpark
  role: Three-letter bisulfite aligner producing the mapRDDs, combRDD, and filteredRDD
    whose best unique alignments are scored by the metrics
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Distributed framework on which readRDD/transRDD/mapRDD transformations and
    the metric computation are implemented
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Bowtie2
  role: Aligner invoked by BiSpark on each transformed transRDD against the corresponding
    customized genome
- name: Sherman
  role: Simulator generating the bisulfite FASTQ reads with known original positions
    used as ground truth
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:bf1f3f99808a1b094af4b8a5f7e841e0f622508ad4c08e425259d4a84cdeb7e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bhi-kimlab/BiSpark
  tools:
  - Apache Spark
  - BiSpark
  - Bowtie2
  - Sherman
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mapping-quality-metric-computation

## Summary

Compute the four mapping-quality metrics (mappability, sensitivity, precision, accuracy) for bisulfite aligner output by comparing alignments against known original positions embedded in simulated reads, as done for BiSpark on Sherman-simulated chromosome 1 data.

## When to use

Use when you have simulated bisulfite-seq reads (e.g., Sherman FASTQ with embedded original positions) plus a filtered alignment (SAM), and need to quantify mappability, sensitivity, precision, and accuracy per error rate.

## When NOT to use

- Real WGBS data without embedded ground-truth positions — true positives cannot be determined, so precision/sensitivity/accuracy are not computable (methylation profiling, not evaluation, is the goal)
- The goal is alignment speed or scalability benchmarking against CloudAligner, Bismark, or Bison rather than mapping-quality measurement

## Inputs

- Sherman-simulated FASTQ reads (human chromosome 1, 95bp, 1,000,000 reads) at 0%, 1%, and 2% error rates, with known original positions embedded
- Bowtie2 indexes of the four 'three-letter' converted hg19 chromosome 1 genomes (W-CT, W-GA, C-CT, C-GA) staged in HDFS
- BiSpark filteredRDD / SAM output of best unique alignments (user-defined mismatch cutoff, default 4)

## Outputs

- Per-error-rate values of mappability, sensitivity, precision, and accuracy
- A single table file containing all metrics for the 0%, 1%, and 2% error datasets

## How to apply

Align the simulated reads with the three-letter BiSpark pipeline: transform each readRDD into CT and GA transRDDs, map each with Bowtie2 to the four W-CT/W-GA/C-CT/C-GA converted hg19 chromosome 1 genomes (index stored in HDFS) using a mismatch cutoff of 4, then union the four mapRDDs without shuffling, combine by read id, and keep only reads with a uniquely smallest mismatch count as the best alignment (filteredRDD), discarding ambiguous alignments. Then, using the known original positions embedded in the Sherman simulation, compute mappability as mapped/all reads, precision as TP/(TP+FP), sensitivity as TP/(TP+FN), and accuracy as TP/(TP+FP+FN) for each error rate (0%, 1%, 2%). Write all metrics for the three error rates into a single table file; run once in BiSpark-plain and once in BiSpark-balance mode, which must give identical metric values.

## Related tools

- **BiSpark** (Three-letter bisulfite aligner producing the mapRDDs, combRDD, and filteredRDD whose best unique alignments are scored by the metrics) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Distributed framework on which readRDD/transRDD/mapRDD transformations and the metric computation are implemented) — https://github.com/bhi-kimlab/BiSpark/
- **Bowtie2** (Aligner invoked by BiSpark on each transformed transRDD against the corresponding customized genome)
- **Sherman** (Simulator generating the bisulfite FASTQ reads with known original positions used as ground truth)

## Examples

```
./src/alignment/run.sh --input reads.fastq --ref hdfs:///index_hg19_chr1 --num_mm 4 --local_save out.sam --testmode plain
```

## Evaluation signals

- 0% error dataset must yield mappability/sensitivity 0.9569, precision 1.0, accuracy 0.9569 — the best values among compared aligners
- 1% error dataset must yield mappability 0.9494, sensitivity 0.9489, accuracy 0.9392
- 2% error dataset must yield mappability 0.9422, sensitivity 0.9411, accuracy 0.9234
- BiSpark-plain and BiSpark-balance runs must produce identical metric values
- Metrics must only count filteredRDD alignments — reads with ambiguous (tied) best alignments are discarded before scoring

## Limitations

- Precision, sensitivity, and accuracy depend entirely on simulated ground truth; results are only defined for datasets with known original positions
- Reads whose smallest mismatch count is not unique are discarded, which lowers mappability and sensitivity by design
- The mismatch cutoff (default 4) is user-defined and shifts the trade-off between restriction intensity and the reported metrics

## Evidence

- [methods] the BiSpark shows the best performance on all four metrics with the 0% error dataset: "the BiSpark shows the best performance on all four metrics with the 0% error dataset"
- [methods] union of mapRDDs and filtering ambiguous reads: "the next step of the analysis is figuring out the best alignment among these results"
- [methods] user-defined mismatch cutoff: "the BiSpark also supports a user-defined mismatch cutoff to adjust the intensity of the restriction"
- [methods] Sherman simulation setup: "setting up with human chromosome 1, read length to 95bp, and the number of reads to 1,000,000"
- [readme] four-phase workflow including filtering: "Aggregating mapping results and filtering ambiguous reads"
