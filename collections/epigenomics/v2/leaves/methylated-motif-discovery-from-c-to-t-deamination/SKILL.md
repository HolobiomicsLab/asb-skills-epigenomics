---
name: methylated-motif-discovery-from-c-to-t-deamination
description: Use when RIMS-seq reads must yield m5C methylase specificities without
  prior knowledge of the methylome — e.g. mapping A. calcoaceticus reads to a de novo
  SPAdes assembly to recover motifs like GATC and CGCG with MosDi significance thresholds.
when_to_use_negative:
- The sequencing library was prepared by tagmentation or similar technologies — RIMS-seq
  requires an adapter-ligation-based protocol
- The goal is the precise genomic location of methylated cytosines rather than global
  motif specificity — EM-seq or bisulfite-seq are more appropriate
- The input is metagenomic community data with closely related co-existing species
  and SNPs, where C-to-T deamination detection background rises and motif-to-species
  assignment is unreliable
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: RIMS-seq.pl
  role: Wrapper that takes paired-end reads and outputs a list of methylase specificities,
    automating the Option 1 assembly-based route
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: SPAdes
  role: De novo assembly of the bacterial genome from RIMS-seq reads when no exact
    reference genome exists
  repo: https://github.com/ablab/spades
- name: mapping.pl
  role: Maps RIMS-seq paired-end reads to the reference/assembly with BWA-mem, producing
    the .bam input for mpileup
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: split_mapped_reads.pl
  role: Separates Read 1 and Read 2 from the BWA bam into two mpileup files using
    samtools mpileup
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: get_motif_all.pl
  role: Identifies methylase specificity / methylated motifs from the two mpileup
    files using MosDi
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: MosDi
  role: Motif discovery engine invoked by get_motif_all.pl, with --qualityscore, --flank
    and --significance parameters
  repo: https://bitbucket.org/tobiasmarschall/mosdi/src/master/
- name: BWA-mem
  role: Short-read mapper used inside mapping.pl to align paired-end RIMS-seq reads
  repo: https://github.com/lh3/bwa
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:e6a8c8e24c13a1c5e0783a922d78e02ad7ba926b3a963af9d080109d69237905
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ettwiller/RIMS-seq
  tools:
  - BWA-mem
  - MosDi
  - RIMS-seq.pl
  - SPAdes
  - get_motif_all.pl
  - mapping.pl
  - split_mapped_reads.pl
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# methylated-motif-discovery-from-c-to-t-deamination

## Summary

De novo discovery of m5C methylase specificities from RIMS-seq reads by exploiting C-to-T deamination at methylated cytosines, running MosDi-based motif discovery (get_motif_all.pl) on Read1/Read2 mpileup files derived from either a reference genome or a SPAdes de novo assembly.

## When to use

Use when RIMS-seq reads must yield m5C methylase specificities without prior knowledge of the methylome — e.g. mapping A. calcoaceticus reads to a de novo SPAdes assembly to recover motifs like GATC and CGCG with MosDi significance thresholds.

## When NOT to use

- The sequencing library was prepared by tagmentation or similar technologies — RIMS-seq requires an adapter-ligation-based protocol
- The goal is the precise genomic location of methylated cytosines rather than global motif specificity — EM-seq or bisulfite-seq are more appropriate
- The input is metagenomic community data with closely related co-existing species and SNPs, where C-to-T deamination detection background rises and motif-to-species assignment is unreliable

## Inputs

- Paired-end RIMS-seq reads in fastq.gz (downsampled to 1-5 million read pairs, adapters removed)
- Reference genome in FASTA, or a de novo SPAdes assembly used as the reference (RIMS-seq Option 1)
- BWA-mem paired-end alignments in .bam format
- Read1 and Read2 samtools mpileup files generated with -Q 10 / -q 0

## Outputs

- List of m5C methylase specificities / methylated motifs with MosDi significance values (e.g. GATC P = 1.44e-1255, CGCG P = 8.6e-228)
- A de novo genome assembly (SPAdes) usable as reference for downstream motif discovery

## How to apply

Downsample the paired-end RIMS-seq reads to 1-5 million read pairs to avoid lengthy processing during motif discovery. If no exact reference genome is available, assemble the genome de novo with SPAdes (RIMS-seq Option 1, automated by RIMS-seq.pl) and use the assembly as the mapping reference. Map reads with BWA-mem in paired-end mode (mapping.pl, .bam output, adapters removed beforehand), then split the alignments into Read1 and Read2 mpileup files with split_mapped_reads.pl using samtools mpileup with default filters (-Q 10 minimum base quality, -q 0 minimum mapping quality). Run get_motif_all.pl on the two mpileup files with MosDi, setting --qualityscore 35 (default 30), --flank 8, and --significance 1e-50; the script detects C-to-T deamination signal to call methylase specificity. Accept the run when top-ranked motif significances match expected values — for A. calcoaceticus ATCC 49823, GATC (P = 1.44e-1255) and CGCG (P = 8.6e-228) — and when assembly-based motifs match those obtained against a reference genome.

## Related tools

- **RIMS-seq.pl** (Wrapper that takes paired-end reads and outputs a list of methylase specificities, automating the Option 1 assembly-based route) — https://github.com/Ettwiller/RIMS-seq/
- **SPAdes** (De novo assembly of the bacterial genome from RIMS-seq reads when no exact reference genome exists) — https://github.com/ablab/spades
- **mapping.pl** (Maps RIMS-seq paired-end reads to the reference/assembly with BWA-mem, producing the .bam input for mpileup) — https://github.com/Ettwiller/RIMS-seq/
- **split_mapped_reads.pl** (Separates Read 1 and Read 2 from the BWA bam into two mpileup files using samtools mpileup) — https://github.com/Ettwiller/RIMS-seq/
- **get_motif_all.pl** (Identifies methylase specificity / methylated motifs from the two mpileup files using MosDi) — https://github.com/Ettwiller/RIMS-seq/
- **MosDi** (Motif discovery engine invoked by get_motif_all.pl, with --qualityscore, --flank and --significance parameters) — https://bitbucket.org/tobiasmarschall/mosdi/src/master/
- **BWA-mem** (Short-read mapper used inside mapping.pl to align paired-end RIMS-seq reads) — https://github.com/lh3/bwa

## Examples

```
get_motif_all.pl read1.mpileup read2.mpileup --qualityscore 35 --flank 8 --significance 1e-50
```

## Evaluation signals

- Top-ranked motifs and significances match expected values: GATC (P = 1.44e-1255) and CGCG (P = 8.6e-228) for A. calcoaceticus ATCC 49823
- Assembly-based motifs match those identified when mapping to a reference genome with similar significance (Figure 3C reference values)
- Motif discovery output passes the --significance 1e-50 cutoff with --qualityscore 35 and --flank 8
- mpileup files were produced with samtools default filters (-Q 10 minimum base quality, -q 0 minimum mapping quality)
- Sequencing QC of the RIMS-seq data (coverage, GC content, mapping rate) is comparable to standard DNA-seq

## Limitations

- RIMS-seq does not work if the library preparation protocol is based on tagmentation or other technologies; adapter-ligation-based protocols are required
- For Option 2 the reference genome must be exactly the same strain as the sequenced bacterium; otherwise use Option 1 (de novo assembly)
- Single nucleotide polymorphisms in microbial communities confound C-to-T deamination detection, increasing background noise for motif detection
- Uneven species representation can cause motifs to be identified only in the most abundant species, and RIMS-seq signals require enough read coverage to compensate for scarce motif sites
- Skipping the column-based cleaning step (small library volume for amplification) is theoretically possible but has not been tested

## Evidence

- [methods] This script takes the two mpileup files previsouly generated and identifies the methylase specificity using a program called MosDi.: "This script takes the two mpileup files previsouly generated and identifies the methylase specificity using a program called MosDi."
- [methods] downsampling recommendation: "it is highly recommended to downsample the number of paired-end read to 1-5 millions to avoid lengthly processing time during the motif discovery step."
- [intro] de novo assembly rationale: "RIMS-seq de novo identifies novel activities without the need for a reference genome and permits the assembly of the bacterial genome at metrics comparable to standard shotgun sequencing."
- [discussion] SNP background caveat: "single nucleotide polymorphisms found in microbial communities may confound the identification of the C to T deamination, increasing the background noise for the detection of motifs"
- [readme] option 2 reference requirement: "In case of option2, your reference genome needs to be **exactly** the same as the bacteria sequenced using RIMS-seq."
- [readme] adapter ligation requirement: "RIMS-seq does not work if the library preparation protocol is based on tagmentation or other technologies"
