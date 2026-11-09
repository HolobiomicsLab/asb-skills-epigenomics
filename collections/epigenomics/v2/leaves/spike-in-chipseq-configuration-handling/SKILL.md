---
name: spike-in-chipseq-configuration-handling
description: Use when preparing to run spikChIP on multiple ChIP-seq experiments with
  exogenous spike-in (e.g. human hg19 + fruit fly dm3) and BAM/peak files must be
  wired into a configuration file before normalization.
when_to_use_negative:
- BAM files or BED peak files do not yet exist — spikChIP requires them as pre-generated
  user inputs and its Step 0 check will fail
- The ChIP-seq experiment has no spike-in control, so there is no sample/spike BAM
  pair or exogenous genome identifier to declare in the configuration
- The exogenous genome is not among the supported identifiers (FLY, HUMAN, MOUSE,
  EXO) handled by the -e option
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: spikChIP
  role: Perl pipeline whose Step 0 reads the options, loads the configuration file,
    and checks the existence of the declared BAM and BED files before bin segmentation
  repo: https://github.com/eblancoga/spikChIP
- name: Bowtie
  role: Mapping tool used to generate the per-genome (sample/spike) BAM files referenced
    in the configuration file; multi-locus reads are discarded
  repo: http://bowtie-bio.sourceforge.net
- name: MACS2
  role: Peak caller run in broad-peak mode to generate the BED peak files declared
    in the configuration (peaks_sample, peaks_spike)
  repo: https://pypi.org/project/MACS2
- name: SeqCode
  role: Downstream suite used to count reads in bins and compute the overlap between
    1-kbp bins and BED peaks for peaks/background classification
  repo: https://github.com/eblancoga/seqcode
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:d8924773d7885912376072a40cb85a1b42a1918b82985cb0bec6209e7ca4e191
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
  - MACS2
  - SeqCode
  - spikChIP
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spike-in-chipseq-configuration-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assemble and validate the spikChIP inputs for comparative ChIP-seq with spike-in chromatin: a plain-text configuration file mapping each condition to its sample/spike BAM files and BED peak files, plus a ChromInfo file listing sample (hg19) and spike-in (dm3, 'FLY'-suffixed) chromosome names and lengths.

## When to use

Use when preparing to run spikChIP on multiple ChIP-seq experiments with exogenous spike-in (e.g. human hg19 + fruit fly dm3) and BAM/peak files must be wired into a configuration file before normalization.

## When NOT to use

- BAM files or BED peak files do not yet exist — spikChIP requires them as pre-generated user inputs and its Step 0 check will fail
- The ChIP-seq experiment has no spike-in control, so there is no sample/spike BAM pair or exogenous genome identifier to declare in the configuration
- The exogenous genome is not among the supported identifiers (FLY, HUMAN, MOUSE, EXO) handled by the -e option

## Inputs

- Plain-text configuration file (tab-separated: sample name, bam_sample, bam_spike, peaks_sample, peaks_spike paths)
- ChromInfo.txt file listing chromosome names and lengths for the sample and spike-in genome assemblies (e.g. hg19 and dm3)
- BAM files of sample and spike genomes produced by Bowtie mapping (multi-locus reads discarded)
- BED peak files produced by MACS2 broad-peak calling (e.g. peak_files/1_sample_peaks.bed)

## Outputs

- Validated configuration: existence-checked BAM and BED files per condition
- Segmentation of the sample and spike-in genomes into fixed-size bins (1,000 bp in the published workflow) following ChromInfo.txt
- Labelled bin files assigning bins to peaks or background classes, written to the results folder

## How to apply

Build the configuration file as a plain-text tab-separated table with the header '#sample bam_sample bam_spike peaks_sample peaks_spike' and one line per experimental condition. Generate the per-genome BAM files beforehand with a mapper such as Bowtie (multi-locus reads discarded) and call peaks with MACS2 in broad-peak mode (as used for H3K79me2 on Jurkat_K79_25%_R1, GSM1465005), converting MACS2 output to BED peak files such as peak_files/1_sample_peaks.bed. Prepare a ChromInfo.txt listing chromosome names and lengths for both the sample (hg19) and spike-in (dm3) genomes, and invoke spikChIP with options such as -v (verbose), -e FLY (exogenous genome identifier; also HUMAN/MOUSE/EXO), -k (bin size in kbp; the paper segments both genomes into 1,000-bp bins), and -p (boxplot palette). spikChIP's Step 0 reads these options, loads the configuration file, and checks the existence of all declared BAM and BED files before segmentation and downstream classification of bins into peaks versus background regions.

## Related tools

- **spikChIP** (Perl pipeline whose Step 0 reads the options, loads the configuration file, and checks the existence of the declared BAM and BED files before bin segmentation) — https://github.com/eblancoga/spikChIP
- **Bowtie** (Mapping tool used to generate the per-genome (sample/spike) BAM files referenced in the configuration file; multi-locus reads are discarded) — http://bowtie-bio.sourceforge.net
- **MACS2** (Peak caller run in broad-peak mode to generate the BED peak files declared in the configuration (peaks_sample, peaks_spike)) — https://pypi.org/project/MACS2
- **SeqCode** (Downstream suite used to count reads in bins and compute the overlap between 1-kbp bins and BED peaks for peaks/background classification) — https://github.com/eblancoga/seqcode

## Examples

```
./spikChIP -vhcde FLY -k 1 -p 0 my_config.txt hg19_dm3_ChromInfo.txt
```

## Evaluation signals

- Step 0 completes without errors: every BAM and BED file declared in the configuration file exists and is readable
- Each configuration line resolves to four files (bam_sample, bam_spike, peaks_sample, peaks_spike) and the peak BED files are in valid BED format (e.g. peak_files/1_sample_peaks.bed)
- The ChromInfo file covers both genomes: hg19 chromosomes for the sample and dm3 chromosomes carrying the 'FLY' suffix for the spike-in
- After segmentation, the labelled bin files contain non-null bins assigned to both the peaks and background classes, matching the published spikChIP results
- BAM files with fewer than 1 million reads are rejected unless the -d option is set, confirming the configuration was parsed correctly

## Limitations

- By default spikChIP refuses to process BAM files with fewer than 1 million reads; the -d option must be set to allow smaller files
- The exogenous genome identifier is restricted to FLY, HUMAN, MOUSE or EXO (default FLY), so other spike-in species require the EXO setting
- Peak files must be generated with an external peak caller (MACS2, broad peaks in the published workflow) before running spikChIP; the pipeline does not call peaks itself
- Intermediate files can inflate the output folder unless the -c option is used to remove them

## Evidence

- [readme] The configuration file of spikChIP is a plain-text file in which each line contains the information about the files of a particular experimental condition: [evidence span withheld — anchor status: external_doc]
