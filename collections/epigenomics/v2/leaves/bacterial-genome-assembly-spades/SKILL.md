---
name: bacterial-genome-assembly-spades
description: Use when processing RIMS-seq (or similar shotgun) paired-end FASTQ reads
  from a bacterium for which no exact reference genome is available, and an assembled
  genome is needed for read mapping and motif discovery.
when_to_use_negative:
- An exact reference genome for the sequenced strain is available and you plan to
  map reads yourself (option 2; the pipeline notes SPAdes and BWA are not necessary
  in that case)
- The sequencing library was prepared by tagmentation or other non-adapter-ligation
  protocols, which RIMS-seq explicitly does not support
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: SPAdes
  role: De novo assembles the bacterial reference genome from the RIMS-seq paired-end
    reads (invoked by RIMS-seq.pl option 1) for use in all downstream analysis
  repo: https://github.com/ablab/spades
- name: RIMS-seq.pl
  role: Pipeline script (option 1) that orchestrates the SPAdes assembly of the genome
    from paired-end reads and feeds it to mapping and motif-discovery steps
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: mapping.pl
  role: Consumes the SPAdes-assembled genome (--genome) to map the paired-end reads
    with BWA-mem and produce the bam file
  repo: https://github.com/Ettwiller/RIMS-seq/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:640bec061f2f935256e751efc22d476bea99c1f7a4773c1b54db77af24e30183
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/spades
  tools:
  - RIMS-seq.pl
  - SPAdes
  - mapping.pl
  - split_mapped_reads.pl
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# bacterial-genome-assembly-spades

## Summary

Assemble a bacterial reference genome de novo from RIMS-seq Illumina paired-end reads with SPAdes, so that downstream methylation-motif discovery (m5C specificity via MoSDi) can proceed without an external reference genome.

## When to use

Use when processing RIMS-seq (or similar shotgun) paired-end FASTQ reads from a bacterium for which no exact reference genome is available, and an assembled genome is needed for read mapping and motif discovery.

## When NOT to use

- An exact reference genome for the sequenced strain is available and you plan to map reads yourself (option 2; the pipeline notes SPAdes and BWA are not necessary in that case)
- The sequencing library was prepared by tagmentation or other non-adapter-ligation protocols, which RIMS-seq explicitly does not support

## Inputs

- Paired-end fastq.gz RIMS-seq reads (Illumina, adapter-ligated library, adapter-removed), downsampled to 1-5 million read pairs (e.g. ecoli_rims_3h_5M.1.fastq.gz / .2.fastq.gz)

## Outputs

- Assembled bacterial reference genome (FASTA contigs) used as --genome for BWA-mem mapping and all downstream RIMS-seq steps

## How to apply

Run RIMS-seq.pl with option 1 per sample: the script performs a genome assembly step using SPAdes on the downsampled paired-end FASTQ reads and then uses that assembled genome as the reference for all downstream analysis. This 'de novo' route is chosen when the exact reference genome for the sequenced strain is not available, since option 2 requires a reference genome that is exactly the same as the bacteria sequenced. Before assembly, downsample the paired-end reads to 1-5 million read pairs to avoid lengthy processing time during the motif discovery step. The assembled genome is then indexed and used by mapping.pl (BWA-mem, paired-end mode via --fq1, --fq2, --genome, --out) to produce a bam file. Note SPAdes is optional in the RIMS-seq pipeline: if a reference genome exists and you do your own mapping, SPAdes and BWA are not necessary.

## Related tools

- **SPAdes** (De novo assembles the bacterial reference genome from the RIMS-seq paired-end reads (invoked by RIMS-seq.pl option 1) for use in all downstream analysis) — https://github.com/ablab/spades
- **RIMS-seq.pl** (Pipeline script (option 1) that orchestrates the SPAdes assembly of the genome from paired-end reads and feeds it to mapping and motif-discovery steps) — https://github.com/Ettwiller/RIMS-seq/
- **mapping.pl** (Consumes the SPAdes-assembled genome (--genome) to map the paired-end reads with BWA-mem and produce the bam file) — https://github.com/Ettwiller/RIMS-seq/

## Examples

```
RIMS-seq.pl --option1 --fq1 ecoli_rims_3h_5M.1.fastq.gz --fq2 ecoli_rims_3h_5M.2.fastq.gz --out rims_out  # runs SPAdes assembly, then BWA-mem mapping against the assembled genome
```

## Evaluation signals

- The assembled genome serves as the --genome argument for mapping.pl/BWA-mem and yields a bam file with RIMS-seq QC metrics (coverage, GC content, mapping rate) similar to standard DNA-seq
- Downstream get_motif_all.pl on mpileups from the assembled-genome mapping recovers the expected methylase specificity (e.g. CCWGG for E. coli K12 MG1655, dcm specificity) with significance passing --significance 1e-50
- Reads are downsampled to the recommended 1-5 million paired-end reads before assembly/pipeline run to keep motif-discovery processing tractable
- For the E. coli K12 time-course validation, motif P-values match the reported 9.71e-77 (10 min), 4.25e-858 (60 min) and 3.61e-4371 (180 min)

## Limitations

- If the exact reference genome is not available, option 1 (SPAdes assembly from the reads) is recommended; option 2's reference must be exactly the same strain, so a mismatched reference is a failure mode
- Assembly/processing time grows with read count; more than 1-5 million paired-end reads causes lengthy motif-discovery processing
- RIMS-seq signals require enough read coverage; low-coverage samples may not yield detectable motif signal after assembly and mapping

## Evidence

- [methods] It includes a genome assembly step using SPAdes and uses this assembled genome for the downstream analysis.: "It includes a genome assembly step using SPAdes and uses this assembled genome for the downstream analysis."
- [methods] it is highly recommended to downsample the number of paired-end read to 1-5 millions to avoid lengthly processing time during the motif discovery step.: "it is highly recommended to downsample the number of paired-end read to 1-5 millions to avoid lengthly processing time during the motif discovery step."
- [readme] Note: if you have a reference genome and perform you own mapping, SPAdes and BWA are not necessary (see option 2).: "Note: if you have a reference genome and perform you own mapping, SPAdes and BWA are not necessary (see option 2)."
- [intro] RIMS-seq de novo identifies novel activities without the need for a reference genome and permits the assembly of the bacterial genome at metrics comparable to standard shotgun sequencing.: "RIMS-seq de novo identifies novel activities without the need for a reference genome and permits the assembly of the bacterial genome at metrics comparable to standard shotgun sequencing."
