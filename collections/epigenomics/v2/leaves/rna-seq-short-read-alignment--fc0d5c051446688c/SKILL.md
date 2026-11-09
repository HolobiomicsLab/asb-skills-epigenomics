---
name: rna-seq-short-read-alignment
description: Use when FASTQ RNA-Seq reads (single- or paired-end Illumina) must be
  spliced-aligned to a reference genome before expression estimation, and novel junction
  discovery should be suppressed in favor of known annotated junctions.
when_to_use_negative:
- The alignment target requires novel splice-junction discovery — --no-novel-juncs
  restricts TopHat to annotated refGene junctions only
- Reads are already aligned (an accepted_hits.bam or other BAM of mapped reads exists);
  proceed directly to eXpress-D expression estimation and bias correction
- The reference genome/annotation combination is not hg18 with RefSeq refGene and
  the transcript models needed downstream differ
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: TopHat
  role: Spliced short-read mapper (v1.1.0) that aligns SRA012427 Illumina reads to
    the hg18 index using refGene annotation with --no-novel-juncs, producing accepted_hits.bam
- name: eXpress-D
  role: Downstream consumer of accepted_hits.bam for likelihood-based expression estimation
    and fragment-bias correction
  repo: https://github.com/adarob/express-d/wiki
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/gb-2011-12-3-r22
    title: Improving RNA-Seq expression estimates by correcting for fragment bias
  source_content_hash: sha256:83146a672e2b2b9d4a837ba27ce933722b981458fab46dcc5deb8901d846ebc8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TopHat
  - eXpress-D
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# rna-seq-short-read-alignment

## Summary

Maps Illumina RNA-Seq reads (e.g., MAQC Human Brain Reference reads from SRA012427) to the NCBI36/hg18 genome with TopHat 1.1.0, using the UCSC RefSeq refGene annotation and --no-novel-juncs so only annotated junctions are considered. The resulting accepted_hits.bam is the sole alignment input for downstream expression-estimation and bias-correction with the eXpress-D likelihood-based pipeline.

## When to use

Use when FASTQ RNA-Seq reads (single- or paired-end Illumina) must be spliced-aligned to a reference genome before expression estimation, and novel junction discovery should be suppressed in favor of known annotated junctions.

## When NOT to use

- The alignment target requires novel splice-junction discovery — --no-novel-juncs restricts TopHat to annotated refGene junctions only
- Reads are already aligned (an accepted_hits.bam or other BAM of mapped reads exists); proceed directly to eXpress-D expression estimation and bias correction
- The reference genome/annotation combination is not hg18 with RefSeq refGene and the transcript models needed downstream differ

## Inputs

- Illumina RNA-Seq FASTQ reads from SRA (e.g., SRA012427 MAQC Human Brain Reference sample)
- hg18/NCBI36 reference genome index for TopHat
- RefSeq refGene hg18 gene annotation (GTF/GFF) downloaded from the UCSC Genome Browser

## Outputs

- accepted_hits.bam — TopHat alignment file in valid BAM format containing mapped read alignments
- TopHat run logs/metrics confirming completion

## How to apply

Download the Illumina FASTQ reads from the SRA (e.g., the MAQC Human Brain Reference sample, accession SRA012427) and build the hg18 (NCBI36) reference index. Prepare the RefSeq refGene hg18 annotation downloaded from the UCSC Genome Browser for TopHat. Run TopHat version 1.1.0 against the hg18 index with the refGene GTF supplied and the --no-novel-juncs flag enabled, so only annotated splice junctions are considered; this is deliberate because the downstream eXpress-D pipeline of Roberts et al. performs likelihood-based expression and bias estimation on fragments aligned to known transcripts. Verify TopHat emitted accepted_hits.bam, a valid BAM containing mapped read alignments, which serves as the sole alignment input to expression estimation and bias correction in eXpress-D.

## Related tools

- **TopHat** (Spliced short-read mapper (v1.1.0) that aligns SRA012427 Illumina reads to the hg18 index using refGene annotation with --no-novel-juncs, producing accepted_hits.bam)
- **eXpress-D** (Downstream consumer of accepted_hits.bam for likelihood-based expression estimation and fragment-bias correction) — https://github.com/adarob/express-d/wiki

## Examples

```
tophat -p 4 -G refGene_hg18.gtf --no-novel-juncs hg18 SRA012427_HBR_reads.fastq
```

## Evaluation signals

- accepted_hits.bam exists after the TopHat run (file_exists check)
- accepted_hits.bam passes BAM format validation (file_format_is check)
- BAM contains mapped read alignments usable by downstream eXpress-D tasks (script_runs check)
- TopHat was invoked with the refGene annotation supplied and --no-novel-juncs set, so alignments should lie on annotated transcripts/junctions only
- Alignment rate should be consistent with the MAQC Human Brain Reference library sequenced on the Illumina platform

## Limitations

- --no-novel-juncs restricts alignment to annotated junctions; reads spanning unannotated splice junctions in hg18 refGene will be lost or misassigned
- The 2011-era TopHat 1.1.0 / hg18 / RefSeq refGene combination is version- and annotation-specific; changing genome builds or annotation sources changes the alignment set
- eXpress-D requires the alignment to be provided as accepted_hits.bam in BAM format; other mapper outputs must be converted to this form

## Evidence

- [other] Run TopHat (version 1.1.0) on the HBR reads against the hg18 index with the refGene annotation supplied and the --no-novel-juncs flag enabled: [evidence span withheld — anchor status: not_found]
- [other] the RefSeq refGene NCBI36/hg18 annotation downloaded from the UCSC Genome Browser: [evidence span withheld — anchor status: not_found]
- [other] confirm accepted_hits.bam exists, is a valid BAM, and contains mapped read alignments usable by the downstream eXpress-D tasks: [evidence span withheld — anchor status: not_found]
- [readme] Distributed ambiguous fragment assignment using eXpress and Spark.: "Distributed ambiguous fragment assignment using [eXpress](http://bio.math.berkeley.edu/eXpress) and [Spark](http://spark.incubator.apache.org/)."
- [methods] For current documentation, see the [eXpress-D Wiki]: "For current documentation, see the [eXpress-D Wiki]"
