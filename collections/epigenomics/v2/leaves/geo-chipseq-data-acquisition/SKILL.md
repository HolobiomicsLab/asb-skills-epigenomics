---
name: geo-chipseq-data-acquisition
description: Use when a comparative ChIP-seq analysis requires raw reads from GEO
  (sample plus spike-in chromatin controls) that must be mapped and peak-called before
  running a normalization method like spikChIP, ChIP-Rx or tag removal.
when_to_use_negative:
- The experiment lacks exogenous spike-in chromatin — spike-in-based normalization
  (and this acquisition/preparation scheme) is not applicable.
- Only processed/normalized tracks are needed and no cross-condition normalization
  comparison will be performed — downloading pre-computed bigWig tracks may suffice.
- Peak files already exist from the original publication in the required BED format
  and only the normalization step remains — re-mapping and re-calling peaks is unnecessary.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: Bowtie
  role: Maps experimental sample reads and fruit fly spike-in reads separately, discarding
    multi-locus reads, to produce the BAM files for each genome
  repo: http://bowtie-bio.sourceforge.net
- name: MACS2
  role: Calls broad peaks on the mapped BAM files to generate the BED peak files required
    for classifying bins into peaks or background
  repo: https://pypi.org/project/MACS2
- name: spikChIP
  role: Downstream consumer of the acquired and preprocessed BAM/BED inputs; takes
    the configuration file and ChromInfo to run genome-wide spike-in normalization
  repo: https://github.com/eblancoga/spikChIP
- name: Samtools
  role: Informs about the number of reads in BAM files (to verify the ~1 million read
    threshold) and for downsampling
  repo: https://github.com/eblancoga/seqcode
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:2dd104ea94a35a211fdd3259a319ad07b7c02a8aac2c03a366180800f51ff37c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/eblancoga/spikChIP
  tools:
  - Bowtie
  - MACS2
  - R
  - Samtools
  - SeqCode
  - gawk
  - spikChIP
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# geo-chipseq-data-acquisition

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Acquire raw ChIP-seq FASTQ/BAM data with exogenous spike-in chromatin from GEO accessions (e.g. GSE64243, samples GSM1890165/GSM1890166) and preprocess them — mapping sample and spike-in reads separately with Bowtie and calling broad peaks with MACS2 — to produce the BAM and BED inputs required by spike-in normalization pipelines such as spikChIP.

## When to use

Use when a comparative ChIP-seq analysis requires raw reads from GEO (sample plus spike-in chromatin controls) that must be mapped and peak-called before running a normalization method like spikChIP, ChIP-Rx or tag removal.

## When NOT to use

- The experiment lacks exogenous spike-in chromatin — spike-in-based normalization (and this acquisition/preparation scheme) is not applicable.
- Only processed/normalized tracks are needed and no cross-condition normalization comparison will be performed — downloading pre-computed bigWig tracks may suffice.
- Peak files already exist from the original publication in the required BED format and only the normalization step remains — re-mapping and re-calling peaks is unnecessary.

## Inputs

- GEO study accession (e.g. GSE64243) with per-sample accessions for control and treated conditions (GSM1890165, GSM1890166)
- Raw ChIP-seq reads with exogenous Drosophila spike-in chromatin (FASTQ/SRA from GEO)
- Reference genome assemblies for the sample genome (hg19) and spike-in genome (dm3)

## Outputs

- BAM files of mapped sample reads and separately mapped spike-in reads, stored in map_files/
- BED files of broad peaks called by MACS2 for sample and spike-in genomes, stored in peak_files/
- spikChIP configuration file (e.g. config/H3K27me3_control-EZH2inh_config.txt) mapping each condition to its BAM and BED files
- ChromInfo.txt for the combined hg19+dm3 genome with 'FLY'-suffixed spike-in chromosomes

## How to apply

Identify the GEO study accession (e.g. GSE64243 for H3K27me3 in PC9 cells treated with the EZH2 inhibitor GSK126) and download the raw reads for each condition, including the Drosophila spike-in chromatin channels (e.g. GSM1890165 control and GSM1890166 treated). Map experimental-sample reads and spike-in reads separately with Bowtie, discarding multi-locus reads, so that human (hg19) and fly (dm3) genomes remain uncross-contaminated. Call peaks with MACS2 using broad-peak mode, depositing sample/spike BAM files in map_files/ and BED peak files in peak_files/. Build a plain-text configuration file with one line per condition (sample name, bam_sample, bam_spike, peaks_sample, peaks_spike) and a ChromInfo file for the combined genome (e.g. hg19+dm3 with 'FLY'-suffixed spike-in chromosomes) so downstream normalization tools can consume the data. BAM files must contain at least ~1 million reads unless the -d option is used, and spike-in reads should ideally be at least one million reads (~2%–5% of the experimental genome).

## Related tools

- **Bowtie** (Maps experimental sample reads and fruit fly spike-in reads separately, discarding multi-locus reads, to produce the BAM files for each genome) — http://bowtie-bio.sourceforge.net
- **MACS2** (Calls broad peaks on the mapped BAM files to generate the BED peak files required for classifying bins into peaks or background) — https://pypi.org/project/MACS2
- **spikChIP** (Downstream consumer of the acquired and preprocessed BAM/BED inputs; takes the configuration file and ChromInfo to run genome-wide spike-in normalization) — https://github.com/eblancoga/spikChIP
- **Samtools** (Informs about the number of reads in BAM files (to verify the ~1 million read threshold) and for downsampling) — https://github.com/eblancoga/seqcode

## Examples

```
bowtie -m 1 hg19 -S sample.fastq > map_files/1_sample.bam; bowtie -m 1 dm3 -S spike.fastq > map_files/1_spike.bam; macs2 callpeak -t map_files/1_sample.bam -f BAM --broad -o peak_files/1_sample_peaks
```

## Evaluation signals

- Sample and spike-in BAM files exist separately in map_files/ for every condition listed in the configuration file (spikChIP Step 0 checks their existence).
- BED peak files for both sample and spike-in genomes exist in peak_files/ for every condition before running the normalization pipeline.
- Each sample/spike BAM pair contains sufficient reads: at least ~1 million reads unless the -d option permits smaller files, with spike-in reads ideally ~2%–5% of experimental genome reads.
- Reproduced metrics from the acquired data match published values, e.g. spike-in equilibration Wilcoxon P-values (ChIP-Rx <2.2e-16, tag removal 9.611e-14, spikChIP 0.01547) and H3K27me3 peak/background reduction (~81–85% for ChIP-Rx/tag removal; 69% peaks vs 23% background for spikChIP).
- The ChromInfo file correctly includes 'FLY'-suffixed spike-in chromosomes so spike-in bins are not confused with sample-genome bins.

## Limitations

- BAM files below 1 million reads are rejected unless the spikChIP -d option is used explicitly.
- The final amount of spike-in reads can be influenced by the mixture ratio, antibody quality, and target abundance in the experimental condition, so acquired GEO samples may have suboptimal spike-in depth.
- Different spike-in normalization strategies can introduce inconsistencies in final results, so the choice of downstream normalization affects reproducibility of the acquired-data comparison.

## Evidence

- [readme] Configuration file links each condition to its BAM and BED files: [evidence span withheld — anchor status: external_doc]
