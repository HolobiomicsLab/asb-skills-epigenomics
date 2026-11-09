---
name: splice-junction-aware-mapping
description: Use when RNA-Seq FASTQ reads must be aligned to a genome for downstream
  expression estimation or fragment-bias correction, and known gene annotations are
  available to constrain splice junctions.
when_to_use_negative:
- When the goal is discovery of novel splice junctions not present in the supplied
  annotation — the --no-novel-juncs setting explicitly restricts alignments to annotated
  junctions only.
- When reads can be aligned directly to a transcriptome without a spliced genome alignment
  step, since this skill's purpose is splice-aware genome mapping.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: TopHat
  role: Splice-aware aligner (v1.1.0) that maps RNA-Seq reads to the hg18 genome using
    RefSeq refGene annotation with --no-novel-juncs enabled
- name: eXpress-D
  role: Downstream pipeline that consumes the accepted_hits.bam alignment as the sole
    alignment input for expression estimation and bias correction
  repo: https://github.com/adarob/express-d/wiki
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/gb-2011-12-3-r22
    title: Improving RNA-Seq expression estimates by correcting for fragment bias
  source_content_hash: sha256:e906dcf6bfb8cec0e5930b59ceca495ecf93d5ca04ed72c9d4bc3b93833bb32a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TopHat
  - eXpress-D
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# splice-junction-aware-mapping

## Summary

Map Illumina RNA-Seq reads to a reference genome with a splice-aware aligner (TopHat 1.1.0), supplying known transcript annotations and disallowing novel junction discovery so only annotated splice junctions are considered, producing the BAM alignments consumed by downstream expression-estimation and bias-correction tools such as Cufflinks/eXpress-D.

## When to use

Use when RNA-Seq FASTQ reads must be aligned to a genome for downstream expression estimation or fragment-bias correction, and known gene annotations are available to constrain splice junctions.

## When NOT to use

- When the goal is discovery of novel splice junctions not present in the supplied annotation — the --no-novel-juncs setting explicitly restricts alignments to annotated junctions only.
- When reads can be aligned directly to a transcriptome without a spliced genome alignment step, since this skill's purpose is splice-aware genome mapping.

## Inputs

- Illumina RNA-Seq FASTQ reads from the SRA public archive (e.g. SRA012427 MAQC Human Brain Reference)
- Pre-built reference genome index for the target assembly (NCBI36/hg18)
- Known gene annotation in a format accepted by TopHat (RefSeq refGene hg18 from the UCSC Genome Browser)

## Outputs

- accepted_hits.bam — a valid BAM file containing mapped read alignments, usable by downstream eXpress-D expression-estimation tasks

## How to apply

Download the RNA-Seq FASTQ reads from the SRA public archive (e.g. accession SRA012427 for the MAQC Human Brain Reference Illumina sample). Prepare a genome index for the appropriate assembly (NCBI36/hg18 for human datasets) plus the RefSeq refGene hg18 annotation downloaded from the UCSC Genome Browser. Run TopHat (version 1.1.0) against the hg18 index with the refGene annotation supplied via the annotation option and the --no-novel-juncs flag enabled, so that only annotated junctions are considered rather than novel junction discovery. Verify TopHat completed and emitted accepted_hits.bam, which serves as the sole alignment input for downstream expression-estimation and bias-correction steps of the eXpress-D pipeline.

## Related tools

- **TopHat** (Splice-aware aligner (v1.1.0) that maps RNA-Seq reads to the hg18 genome using RefSeq refGene annotation with --no-novel-juncs enabled)
- **eXpress-D** (Downstream pipeline that consumes the accepted_hits.bam alignment as the sole alignment input for expression estimation and bias correction) — https://github.com/adarob/express-d/wiki

## Examples

```
tophat -p 4 --no-novel-juncs refGene/hg18 hg18_index SRA012427_HBR_reads.fastq
```

## Evaluation signals

- Confirm accepted_hits.bam exists after the TopHat run (file_exists check)
- Confirm accepted_hits.bam is a valid BAM file (file_format_is check)
- Confirm the BAM contains mapped read alignments usable by downstream eXpress-D expression-estimation tasks (script_runs check)
- Confirm the run used TopHat 1.1.0 with the refGene hg18 annotation supplied and the --no-novel-juncs flag set, so only annotated junctions were considered

## Limitations

- Restricting junctions with --no-novel-juncs means reads spanning unannotated splice junctions will not be mapped; alignments are limited to annotated junctions only.
- The alignment is assembly-specific: the hg18 index and RefSeq refGene hg18 annotation must match the assembly expected by downstream expression-estimation and bias-correction steps.

## Evidence

- [methods] Read mapping was performed with TopHat 1.1.0 using supplied annotations and the --no-novel-juncs option set; for human datasets this meant the RefSeq refGene NCBI36/hg18 annotation downloaded from the UCSC Genome Browser: [evidence span withheld — anchor status: not_found]
- [methods] Run TopHat (version 1.1.0) on the HBR reads against the hg18 index with the refGene annotation supplied and the --no-novel-juncs flag enabled so that only annotated junctions are considered.: [evidence span withheld — anchor status: not_found]
- [methods] confirm accepted_hits.bam exists, is a valid BAM, and contains mapped read alignments usable by the downstream eXpress-D tasks (file_exists / file_format_is / script_runs checks).: [evidence span withheld — anchor status: not_found]
- [readme] For current documentation, see the [eXpress-D Wiki]: "For current documentation, see the [eXpress-D Wiki](https://github.com/adarob/express-d/wiki)."
