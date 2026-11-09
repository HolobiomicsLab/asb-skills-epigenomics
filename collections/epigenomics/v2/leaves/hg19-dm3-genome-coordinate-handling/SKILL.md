---
name: hg19-dm3-genome-coordinate-handling
description: Use when a ChIP-seq spike-in workflow must process reads and bins over
  two different genome assemblies (e.g. human hg19 sample plus fruit fly dm3 spike-in)
  and requires consistent bin segmentation and BAM read totals for both.
when_to_use_negative:
- The experiment uses a single genome without exogenous spike-in material — there
  is no second genome to segment and no cross-genome coordinate handling is needed
- The sample and spike-in genomes are not hg19 and dm3 (e.g. MOUSE or EXO) — the -e
  identifier and ChromInfo files must be swapped to the correct assemblies
- Input is already a binned signal table rather than mapped BAM files — spikChIP expects
  Bowtie-mapped BAM files, not pre-computed counts
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: spikChIP
  role: Segments both the sample (hg19) and spike-in (dm3) genomes into same-size
    bins using the ChromInfo file and configuration file before normalization
  repo: https://github.com/eblancoga/spikChIP
- name: SeqCode
  role: Counts reads of the BAM files inside bins of each genome and overlaps bins
    with peak BED files for peak/background classification
  repo: https://github.com/eblancoga/seqcode
- name: Samtools
  role: Reports the number of reads in the sample and spike-in BAM files (needed for
    ChIP-Rx division by total dm3 spike-in reads)
- name: Bowtie
  role: Maps reads to hg19 and dm3 (multi-locus reads discarded) to produce the BAM
    files per genome prior to spikChIP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:875caca5200255137f90b1d0e24cbdb5b979c0ab6a4ef563035dfffacb09b284
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/eblancoga/spikChIP
  repo_url: https://github.com/eblancoga/spikChIP
  tools:
  - Bowtie
  - Samtools
  - SeqCode
  - gawk
  - spikChIP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# hg19-dm3-genome-coordinate-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simultaneous coordinate handling of the experimental (hg19) and spike-in (dm3) genomes in spikChIP: each genome is described by its own ChromInfo file and segmented into equal-size bins (e.g. 1000 bp via -k 1) so that per-bin ChIP counts and spike-in read totals can be normalized on a common bin framework.

## When to use

Use when a ChIP-seq spike-in workflow must process reads and bins over two different genome assemblies (e.g. human hg19 sample plus fruit fly dm3 spike-in) and requires consistent bin segmentation and BAM read totals for both.

## When NOT to use

- The experiment uses a single genome without exogenous spike-in material — there is no second genome to segment and no cross-genome coordinate handling is needed
- The sample and spike-in genomes are not hg19 and dm3 (e.g. MOUSE or EXO) — the -e identifier and ChromInfo files must be swapped to the correct assemblies
- Input is already a binned signal table rather than mapped BAM files — spikChIP expects Bowtie-mapped BAM files, not pre-computed counts

## Inputs

- ChromInfo.txt chromosome-size files for hg19 and dm3 (plain text)
- spikChIP configuration file mapping each condition to sample and spike-in BAM files plus peak BED files (e.g. config/H3K79me2_25-75_config.txt)
- Mapped sample BAM files (e.g. GSM1465005 Jurkat_K79_25%_R1, GSM1465007 Jurkat_K79_75%_R1)
- Mapped spike-in (dm3) BAM files
- MACS2 broad-peak BED files for peak/background classification

## Outputs

- Bin-segmented genomes (1000-bp bins) for both hg19 and dm3
- Per-bin read counts (average and maximum values) for sample and spike-in genomes
- Genome-wide binned signal tables under multiple normalizations (RAW, TRADITIONAL, CHIPRX, TAG REMOVAL, SPIKCHIP), gzip-compressed in results/

## How to apply

Provide a ChromInfo plain-text file listing the chromosomes and their sizes for each genome (hg19 and dm3) and pass it with the spikChIP configuration file. Run spikChIP with the -k option set to the bin size in kbp (1 for the 1000-bp bins used in the H3K79me2 25:75 vs 75:25 analysis) and the -e option set to FLY for the exogenous dm3 genome; spikChIP segments both the sample and spike-in genomes into bins of the same size. Count reads per bin with SeqCode from the sample BAM (GSM1465005/GSM1465007) and spike-in BAM files, computing both average and maximum per-bin values, and report BAM read totals with Samtools. Bin coordinates are then overlapped with user-provided MACS2 broad-peak BED files via SeqCode to classify bins as peaks or background within each genome's coordinate system.

## Related tools

- **spikChIP** (Segments both the sample (hg19) and spike-in (dm3) genomes into same-size bins using the ChromInfo file and configuration file before normalization) — https://github.com/eblancoga/spikChIP
- **SeqCode** (Counts reads of the BAM files inside bins of each genome and overlaps bins with peak BED files for peak/background classification) — https://github.com/eblancoga/seqcode
- **Samtools** (Reports the number of reads in the sample and spike-in BAM files (needed for ChIP-Rx division by total dm3 spike-in reads))
- **Bowtie** (Maps reads to hg19 and dm3 (multi-locus reads discarded) to produce the BAM files per genome prior to spikChIP)

## Examples

```
./spikChIP -vhcde FLY -k 1 -p 1 config/H3K79me2_25-75_config.txt ChromInfo.txt
```

## Evaluation signals

- Both genomes produce bin tables with bins of the requested size (1000 bp when -k 1); verify chromosome names/sizes match the respective ChromInfo files (hg19 vs dm3)
- spikChIP must report the existence of BAM and BED files in Step 0 and fail cleanly if any file listed in the configuration file is missing
- Per-bin counts exist for both average and maximum within-bin values for each sample/spike pair
- BAM read totals reported by Samtools are non-zero and dm3 spike-in totals are in the expected 2%-5% range relative to the experimental genome
- Downstream bins overlap MACS2 broad-peak BED files, yielding a non-empty peaks class and a non-empty background class for each genome

## Limitations

- spikChIP by default refuses BAM files with fewer than 1 million reads; the -d option is needed to process smaller files
- Spike-in reads in the final sequencing step should be at least one million reads, approximately 2%-5% of the experimental genome, to minimize changes in overall material used
- The ChromInfo file must be a plain-text list of chromosomes compatible with the UCSC genome/assembly releases (hg19 and dm3); mixing chromosome names across assemblies will corrupt bin coordinates

## Evidence
