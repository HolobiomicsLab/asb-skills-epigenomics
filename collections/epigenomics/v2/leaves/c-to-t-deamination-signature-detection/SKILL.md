---
name: c-to-t-deamination-signature-detection
description: Use when RIMS-seq (alkaline-treated) Illumina paired-end fastq.gz reads
  from bacteria must be converted into a de novo m5C methylase motif with a significance
  P-value, e.g. recovering CCWGG from E. coli K12 MG1655 time-course data.
when_to_use_negative:
- The sequencing library was prepared by tagmentation or another non-adapter-ligation
  protocol, which RIMS-seq explicitly does not support
- The methylation of interest is not m5C (e.g. m6A or m4C), since the C-to-T deamination
  signal is specific to 5-methylcytosine
- Sites have too little read coverage — RIMS-seq signals require enough coverage to
  compensate for the scarcity of methylated motif sites
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0102
tools:
- name: RIMS-seq.pl
  role: top-level pipeline script (option 1) that assembles the reference genome from
    RIMS-seq reads using SPAdes and runs all downstream steps
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: mapping.pl
  role: maps paired-end adapters-trimmed reads to the assembled reference with BWA-mem
    (--fq1, --fq2, --genome, --out) to produce a bam file
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: split_mapped_reads.pl
  role: splits the bam via samtools mpileup (-Q 10, -q 0 defaults) into Read 1 and
    Read 2 mpileup files for strand-specific deamination detection
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: get_motif_all.pl
  role: identifies the methylase specificity from the two mpileup files using MoSDi,
    with --qualityscore, --flank and --significance parameters
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: SPAdes
  role: assembles the reference genome from the RIMS-seq reads when no exact reference
    is available (option 1)
  repo: https://github.com/ablab/spades
- name: BWA
  role: BWA-mem aligner used by mapping.pl to map reads to the assembled genome
  repo: https://github.com/lh3/bwa
- name: MoSDi
  role: motif discovery program that computes the over-represented methylated motif
    and its significance
  repo: https://bitbucket.org/tobiasmarschall/mosdi/src/master/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:4e8f315062d9d77805af3f88595c06184fdaeeb940f13f288bdd279832f944db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ettwiller/RIMS-seq
  tools:
  - BWA
  - MoSDi
  - RIMS-seq.pl
  - SPAdes
  - get_motif_all.pl
  - mapping.pl
  - split_mapped_reads.pl
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# c-to-t-deamination-signature-detection

## Summary

Recover de novo m5C methylase specificity motifs from RIMS-seq data, in which heat-alkaline treatment deaminates methylated cytosines to produce C-to-T signatures detectable in standard Illumina paired-end reads. The skill chains genome assembly, BWA-mem mapping, mpileup read splitting, and MoSDi-based motif discovery.

## When to use

Use when RIMS-seq (alkaline-treated) Illumina paired-end fastq.gz reads from bacteria must be converted into a de novo m5C methylase motif with a significance P-value, e.g. recovering CCWGG from E. coli K12 MG1655 time-course data.

## When NOT to use

- The sequencing library was prepared by tagmentation or another non-adapter-ligation protocol, which RIMS-seq explicitly does not support
- The methylation of interest is not m5C (e.g. m6A or m4C), since the C-to-T deamination signal is specific to 5-methylcytosine
- Sites have too little read coverage — RIMS-seq signals require enough coverage to compensate for the scarcity of methylated motif sites

## Inputs

- paired-end fastq.gz RIMS-seq read files (alkaline-treated Illumina libraries, adapter-ligation based, e.g. 1-5 million read pairs per sample)
- assembled or provided reference genome in FASTA (SPAdes assembly from option 1, or exact reference for option 2)
- bam file of paired-end reads mapped with BWA-mem

## Outputs

- Read 1 and Read 2 mpileup files from split_mapped_reads.pl
- de novo m5C methylase specificity motif (e.g. CCWGG) with methylation position and MoSDi significance P-value per sample/treatment time

## How to apply

Retrieve paired-end fastq.gz RIMS-seq reads (e.g. BioProject PRJNA706563) and downsample to 1-5 million read pairs to avoid lengthy MoSDi processing. Run RIMS-seq.pl option 1 per sample, which assembles the reference genome from the RIMS-seq reads with SPAdes and uses that assembled genome downstream (option 2 requires an exactly matching reference genome). Map paired-end adapters-trimmed reads with mapping.pl (BWA-mem, --fq1, --fq2, --genome, --out), then split the bam with split_mapped_reads.pl via samtools mpileup (default -Q 10 base quality, -q 0 mapping quality) into Read1 and Read2 mpileup files. Run get_motif_all.pl on the two mpileup files to call the methylase specificity with MoSDi, using --qualityscore 35 (30-35 recommended for Illumina), --flank 8 (default 7) and --significance 1e-50. For E. coli K12, validate that the recovered motif is CCWGG with m5C at position 2 and P-values of 9.71e-77, 4.25e-858 and 3.61e-4371 for 10, 60 and 180 min alkaline treatment.

## Related tools

- **RIMS-seq.pl** (top-level pipeline script (option 1) that assembles the reference genome from RIMS-seq reads using SPAdes and runs all downstream steps) — https://github.com/Ettwiller/RIMS-seq/
- **mapping.pl** (maps paired-end adapters-trimmed reads to the assembled reference with BWA-mem (--fq1, --fq2, --genome, --out) to produce a bam file) — https://github.com/Ettwiller/RIMS-seq/
- **split_mapped_reads.pl** (splits the bam via samtools mpileup (-Q 10, -q 0 defaults) into Read 1 and Read 2 mpileup files for strand-specific deamination detection) — https://github.com/Ettwiller/RIMS-seq/
- **get_motif_all.pl** (identifies the methylase specificity from the two mpileup files using MoSDi, with --qualityscore, --flank and --significance parameters) — https://github.com/Ettwiller/RIMS-seq/
- **SPAdes** (assembles the reference genome from the RIMS-seq reads when no exact reference is available (option 1)) — https://github.com/ablab/spades
- **BWA** (BWA-mem aligner used by mapping.pl to map reads to the assembled genome) — https://github.com/lh3/bwa
- **MoSDi** (motif discovery program that computes the over-represented methylated motif and its significance) — https://bitbucket.org/tobiasmarschall/mosdi/src/master/

## Examples

```
get_motif_all.pl --qualityscore 35 --flank 8 --significance 1e-50 read1.mpileup read2.mpileup
```

## Evaluation signals

- Recovered motif for E. coli K12 is CCWGG with the methylated cytosine at position 2 for each treatment time
- Significance P-values match the reported values: 9.71e-77 (10 min), 4.25e-858 (60 min) and 3.61e-4371 (180 min alkaline treatment)
- Exactly one significant motif is reported at the --significance 1e-50 threshold
- split_mapped_reads.pl produces two separate mpileup files (Read 1 and Read 2) from the bam
- Base quality filter in get_motif_all.pl falls in the recommended 30-35 range for Illumina data

## Limitations

- RIMS-seq does not work with tagmentation-based library preparation protocols; adapter-ligation-based protocols are required
- Single nucleotide polymorphisms in microbial communities may confound the identification of the C-to-T deamination, increasing background noise for motif detection
- Partially methylated sites can be found but the pervasiveness of methylation needed for a signal has not been evaluated, and low-coverage or rare motif sites may not yield detectable signal
- Skipping the column-based cleaning step (using a small library volume for amplification) is theoretically possible but has not been tested

## Evidence

- [methods] This script takes the two mpileup files previsouly generated and identifies the methylase specificity using a program called MosDi.: "This script takes the two mpileup files previsouly generated and identifies the methylase specificity using a program called MosDi."
- [methods] split reads: "This script splits the reads from the bam file generated by BWA. Reads are separated between Read 1 and Read 2 and two files are created accordingly."
- [methods] downsample: "it is highly recommended to downsample the number of paired-end read to 1-5 millions to avoid lengthly processing time during the motif discovery step."
- [methods] qualityscore: "`--qualityscore` : minimum base calling quality score (default=30)."
- [readme] adapter ligation: "Note that the sequencing library should be done using a protocol based on adapter ligation."
- [methods] assembly step: "It includes a genome assembly step using SPAdes and uses this assembled genome for the downstream analysis."
- [discussion] SNP confounding: "single nucleotide polymorphisms found in microbial communities may confound the identification of the C to T deamination, increasing the background noise for the detection of motifs"
