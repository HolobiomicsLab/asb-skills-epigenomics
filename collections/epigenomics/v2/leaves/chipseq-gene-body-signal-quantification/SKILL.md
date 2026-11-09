---
name: chipseq-gene-body-signal-quantification
description: Use when you have mapped ChIP-seq BAM files plus RefSeq gene groups ranked
  by expression (e.g. highly/moderately/silent RNA-seq groups) and need per-group,
  sequencing-depth-normalized signal levels over gene bodies to build boxplot correlation
  panels.
when_to_use_negative:
- Input is still raw FASTQ — reads must first be mapped with Bowtie and unaligned
  reads removed from the BAM before any SeqCode command can process them
- 'You need genome-browser-ready tracks: bigWig profile generation is not supported
  and deferred to future SeqCode versions'
- The experiment includes spike-in material and you need global scaling — SeqCode
  normalization is by sequencing depth and the spike-in factor must be manually introduced
  by the user
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0204
tools:
- name: SeqCode
  role: recoverChIPlevels computes avg/max/total read counts over gene-body regions,
    normalized by sequencing depth, and R-based scripts generate the boxplots
  repo: https://github.com/eblancoga/seqcode
- name: Bowtie
  role: maps the ChIP-seq reads over the mouse mm9 genome before counting
- name: MACS
  role: MACS12 performs peak calling on the ChIP-seq experiments after mapping; processmacs
    cleans the MACS BedGraph profiles
- name: SAMtools
  role: integrated C libraries inside SeqCode (SAMtools/HTSlib) read the BAM/SAM mapped
    reads
- name: R
  role: internally generates the boxplot and other graphical outputs produced by SeqCode
    commands
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:55b4490be305259af4a45c9e6c1c3841ad6ad892116aa78a576a57949aacab04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Bowtie
  - MACS
  - R
  - SAMtools
  - SeqCode
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# chipseq-gene-body-signal-quantification

## Summary

Quantify normalized ChIP-seq signal (avg/max/total read counts) over gene bodies of selected gene groups using SeqCode's recoverChIPlevels, after Bowtie mapping to mm9, MACS peak calling, and exclusion of unaligned reads from the BAM. This enables expression-class correlation panels such as the mESC active/repressive chromatin mark analysis (Fig. 6).

## When to use

Use when you have mapped ChIP-seq BAM files plus RefSeq gene groups ranked by expression (e.g. highly/moderately/silent RNA-seq groups) and need per-group, sequencing-depth-normalized signal levels over gene bodies to build boxplot correlation panels.

## When NOT to use

- Input is still raw FASTQ — reads must first be mapped with Bowtie and unaligned reads removed from the BAM before any SeqCode command can process them
- You need genome-browser-ready tracks: bigWig profile generation is not supported and deferred to future SeqCode versions
- The experiment includes spike-in material and you need global scaling — SeqCode normalization is by sequencing depth and the spike-in factor must be manually introduced by the user

## Inputs

- ChIP-seq BAM files (unaligned reads excluded) mapped to mm9 with Bowtie, one per chromatin mark
- mm9 chromInfo file (list of chromosome names and sizes, e.g. from UCSC)
- mm9 RefSeq refGene transcript annotation file
- Text files listing the gene bodies of each expression class (highly, moderately, silently expressed, e.g. 100 RefSeq genes each from GSM1562339 RNA-seq ranking)
- MACS12 peak calls (optional, for downstream comparisons)

## Outputs

- Per-mark, per-gene-group avg/max/total normalized ChIP-seq read counts over gene bodies (data files in an output folder)
- R scripts stored in the output folder for regenerating plots
- Boxplots of ChIP levels per expression group for each mark (Fig. 6-style panel)

## How to apply

Retrieve raw ChIP-seq FASTQ files (e.g. SRX367147 for H3K4me3, GSM1019772 for H3K27me3) and the matched RNA-seq (GSM1562339), then map reads to mm9 with Bowtie and call peaks with MACS12. Before running SeqCode, exclude unaligned reads from the resulting BAM files, since SeqCode commands require filtered BAM input. Provide the mm9 chromInfo (chromosome sizes) and refGene annotation files, plus one BAM per mark; recoverChIPlevels computes avg/max/total read counts over the gene bodies of each expression group, normalized by total reads in the BAM file (sequencing depth). The command creates an output folder with data files and R scripts; run these R scripts to produce per-mark boxplots of the per-group ChIP levels. Accept the result when the boxplots reproduce the qualitative Fig. 6 pattern: active marks (H3K4me3, H3K36me3, Ser5P, Ser2P) high on highly expressed genes and Polycomb/repressive marks (H3K27me3, Suz12, Jarid2, Ring1b, H2Aub) high on silent genes.

## Related tools

- **SeqCode** (recoverChIPlevels computes avg/max/total read counts over gene-body regions, normalized by sequencing depth, and R-based scripts generate the boxplots) — https://github.com/eblancoga/seqcode
- **Bowtie** (maps the ChIP-seq reads over the mouse mm9 genome before counting)
- **MACS** (MACS12 performs peak calling on the ChIP-seq experiments after mapping; processmacs cleans the MACS BedGraph profiles)
- **SAMtools** (integrated C libraries inside SeqCode (SAMtools/HTSlib) read the BAM/SAM mapped reads)
- **R** (internally generates the boxplot and other graphical outputs produced by SeqCode commands)

## Examples

```
bin/recoverChIPlevels ChromInfo.txt refGene.txt H3K4me3.bam genes_high.txt out_prefix; R CMD BATCH out_prefix/Rscript.txt
```

## Evaluation signals

- The recovered levels are sequencing-depth normalized: counts per finite element (gene bodies) are divided by total reads in the BAM file
- The output folder contains the per-group avg/max/total count data files plus the R scripts and the boxplot PDF
- Boxplots reproduce the qualitative sign pattern of Fig. 6: H3K4me3/H3K36me3/Ser5P/Ser2P high on highly expressed genes; H3K27me3/Suz12/Jarid2/Ring1b/H2Aub high on silent genes
- BAM inputs contain no unaligned reads (exclusion filter was applied before SeqCode processing)
- tests/test_recoverChIPlevels.pl in the repository runs successfully, confirming the binary was correctly built with make all

## Limitations

- The article reports no numeric threshold for the correlation panel; acceptance is qualitative concordance of correlation directions with Fig. 6
- Spike-in normalization requires the user to manually introduce the spike-in factor into the applications
- Generation of genome-wide profiles in bigWig format is not supported and deferred to future versions
- The SeqCode web tools are intended as a teaching resource; the command-line release has no limits on input size and should be used for full analyses

## Evidence

- [methods] Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands.: "Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands."
- [methods] Normalization by sequencing depth: "SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth"
- [readme] Read counting over regions of interest: "Calculate the count of reads of one sequencing experiment on a set of regions"
- [readme] Graphical outputs are produced by running R scripts: "Graphical outputs are produced by running R scripts that will be stored in the resulting folder as well."
- [methods] Bowtie mapping to mm9: "Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3)."
