---
name: alignment-ambiguity-filtering
description: Use when aligning bisulfite-treated reads across four converted genomes
  (CT/GA x Watson/Crick) and you must merge mapRDDs, then filter ambiguous reads to
  guarantee unique alignments before methylation profiling.
when_to_use_negative:
- When a single-genome (non-bisulfite) aligner is used and ambiguity is already resolved
  by the aligner's own mapping-quality reporting
- When the goal is multi-mapping quantification rather than unique-best alignment,
  since this step explicitly discards ambiguous multi-best hits
- Before Phase 2 'three-letter' transformation exists — there are no CT/GA-strand
  mapRDDs to union and disambiguate
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: BiSpark
  role: 'Implements Phase 3: unions mapRDDs without shuffling, combines by read id
    into combRDD, and filters ambiguous reads into filteredRDD'
  repo: https://github.com/bhi-kimlab/BiSpark/
- name: Apache Spark
  role: Provides the context-level union (no shuffle), key-value RDD grouping by read
    id, and shared-variable broadcasting used in the aggregation step
- name: Bowtie2
  role: Produces the per-strand alignment results (mapRDDs) with mismatch counts that
    are then aggregated and filtered
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2498-2
    title: 'BiSpark: a Spark-based highly scalable aligner for bisulfite sequencing
      data'
  source_content_hash: sha256:00a418d4f94f06123a17353be7f8fd0da335100bf5cd2974c0acd50e4b1a40d0
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
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# alignment-ambiguity-filtering

## Summary

Aggregates mapping results from the four bisulfite 'three-letter' alignments (W-CT, W-GA, C-CT, C-GA) in BiSpark and resolves each read to a single best alignment by keeping only alignments with a uniquely smallest mismatch count, discarding ambiguous multi-best hits and reads exceeding the user-defined mismatch cutoff.

## When to use

Use when aligning bisulfite-treated reads across four converted genomes (CT/GA x Watson/Crick) and you must merge mapRDDs, then filter ambiguous reads to guarantee unique best alignments before methylation profiling.

## When NOT to use

- When a single-genome (non-bisulfite) aligner is used and ambiguity is already resolved by the aligner's own mapping-quality reporting
- When the goal is multi-mapping quantification rather than unique-best alignment, since this step explicitly discards ambiguous multi-best hits
- Before Phase 2 'three-letter' transformation exists — there are no CT/GA-strand mapRDDs to union and disambiguate

## Inputs

- Four mapRDDs of bisulfite alignments (W-CT, W-GA, C-CT, C-GA) keyed by read id
- User-defined mismatch cutoff parameter (--num_mm, default 4)
- transRDD/alignment outputs from Bowtie2 against the four converted and indexed hg19 reference genomes

## Outputs

- filteredRDD containing reads with a uniquely best (smallest mismatch count) alignment
- Discarded ambiguous multi-best-hit reads and reads exceeding the mismatch cutoff
- Downstream input to mergedRDD and methylRDD for Phase 4 methylation profiling

## How to apply

After Phase 2 of BiSpark produces the four mapRDDs (W-CT, W-GA, C-CT, C-GA), union them at the Spark context level without shuffling, then group results by read id into combRDD. For each read, retain only the alignment with a uniquely smallest number of mismatches; if two or more candidate alignments tie for the smallest mismatch count, the read is ambiguous and is discarded into filteredRDD exclusion. Enforce the user-defined mismatch cutoff (default 4) to drop alignments exceeding the allowed mismatches. This Phase 3 'Finding The Best Alignment' step is critical because retaining ambiguous multi-best hits would corrupt the per-site methylation estimates computed in Phase 4 by comparing filteredRDD against the original reference genome.

## Related tools

- **BiSpark** (Implements Phase 3: unions mapRDDs without shuffling, combines by read id into combRDD, and filters ambiguous reads into filteredRDD) — https://github.com/bhi-kimlab/BiSpark/
- **Apache Spark** (Provides the context-level union (no shuffle), key-value RDD grouping by read id, and shared-variable broadcasting used in the aggregation step)
- **Bowtie2** (Produces the per-strand alignment results (mapRDDs) with mismatch counts that are then aggregated and filtered) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml

## Examples

```
./src/alignment/run.sh --input reads.fastq --ref hdfs:///index/hg19_chr1 --num_mm 4 --testmode plain --local_save out.sam
```

## Evaluation signals

- Every read retained in filteredRDD has a strictly unique smallest mismatch count across all four mapRDDs; no ties remain
- No retained alignment exceeds the user-defined mismatch cutoff (default --num_mm 4)
- On the 0%-error Sherman simulation (chromosome 1, 95 bp, 1,000,000 reads), the filtered results yield mappability 0.9569 and precision 1.0, identical for BiSpark-plain and BiSpark-balance
- The union of mapRDDs is performed via Spark context-level union without shuffle, and the total number of input reads equals mapped + ambiguous-discarded + over-cutoff-discarded
- Phase 4 methylation profiling on filteredRDD against the broadcast original reference genome completes without ambiguous-origin conflicts

## Limitations

- Ambiguous multi-best-hit reads are discarded entirely, so heavily duplicated genomic regions reduce mappability (0.9569 at 0% error rather than 1.0)
- The mismatch cutoff is user-adjustable, so changing --num_mm changes filtering stringency and downstream metrics
- The paper does not theoretically resolve the load-balancing vs. shuffling trade-off, and no software version numbers are stated for BiSpark, Spark, or HDFS

## Evidence

- [methods] Union the mapRDDs without shuffling, combine results by read id into combRDD, and keep only reads with a uniquely smallest mismatch count as the best alignment, discarding ambiguous alignments: [evidence span withheld — anchor status: not_found]
- [methods] Mismatch cutoff parameterization: "the BiSpark also supports a user-defined mismatch cutoff to adjust the intensity of the restriction"
- [methods] Phase 3 definition from article: "the next step of the analysis is figuring out the best alignment among these results"
- [intro] Ambiguity filtering as a named workflow phase: "(3) Aggregating mapping results and filtering ambiguous reads"
- [methods] 0%-error validation target for the filtered alignments: "the BiSpark shows the best performance on all four metrics with the 0% error dataset"
