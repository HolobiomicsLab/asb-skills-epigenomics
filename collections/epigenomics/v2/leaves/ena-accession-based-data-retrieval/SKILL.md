---
name: ena-accession-based-data-retrieval
description: Use when a pipeline needs raw ChIP-seq FASTQ files (paired IP and matching
  control) for a defined list of ENA accessions — e.g., the 25 knocked-out transcription
  factor experiments listed in Supplementary Table S1 of the HOT-or-not study.
when_to_use_negative:
- The required data are already available locally as FASTQ or BAM files, so re-querying
  ENA would duplicate retrieval
- The analysis only needs deposited processed files (e.g., ENCODE peak calls or coverage
  tracks) rather than raw reads for IP/control reprocessing
- The accession list is not a matched IP/control set (e.g., single-end ChIP-seq without
  controls), since the downstream log2(IP/control) fold-enrichment computation on
  HOT regions requires both
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: bbduk
  role: Trims adapters and low-quality bases from the ENA-downloaded FASTQ reads before
    alignment
- name: Bowtie
  role: Aligns the retrieved FASTQ reads to the reference genome as the first validation
    that downloads are usable
- name: samtools
  role: Filters, sorts, and indexes Bowtie alignments from the retrieved reads (removing
    unmapped/multimapping reads)
- name: Bedtools
  role: Computes read coverage over HOT region intervals from the alignments derived
    from the downloaded data
- name: genomation
  role: R/Bioconductor package used to compute log2(IP/control) fold enrichment over
    HOT regions from the retrieved-experiment coverage
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:d8a827386bd089d5ee8ac16389f5be036f30164f7cdebe10a9a1abadb1d786aa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bedtools
  - Bowtie
  - bbduk
  - genomation
  - samtools
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ena-accession-based-data-retrieval

## Summary

Resolves ENA run accession numbers for IP and matching control samples into raw ChIP-seq FASTQ files, enabling reproducible re-analysis (here, KO transcription factor ChIP-seq to detect false-positive signal on HOT regions) from primary reads rather than preprocessed peaks.

## When to use

Use when a pipeline needs raw ChIP-seq FASTQ files (paired IP and matching control) for a defined list of ENA accessions — e.g., the 25 knocked-out transcription factor experiments listed in Supplementary Table S1 of the HOT-or-not study.

## When NOT to use

- The required data are already available locally as FASTQ or BAM files, so re-querying ENA would duplicate retrieval
- The analysis only needs deposited processed files (e.g., ENCODE peak calls or coverage tracks) rather than raw reads for IP/control reprocessing
- The accession list is not a matched IP/control set (e.g., single-end ChIP-seq without controls), since the downstream log2(IP/control) fold-enrichment computation on HOT regions requires both

## Inputs

- List of ENA study/run accession numbers for 25 KO transcription factor ChIP-seq experiments (Supplementary Table S1)
- ENA portal API / FTP endpoint (read_run result metadata, fastq_ftp fields)
- Mapping of each IP run to its matching control run

## Outputs

- Raw ChIP-seq FASTQ files for each IP experiment and its matching control
- Run metadata table (accession, experiment, fastq_ftp paths) used to track the 25 KO experiments

## How to apply

Start from the curated accession list (Supplementary Table S1 maps each of the 25 KO transcription factor experiments to its ENA accession, with a matching control run per IP). Query the ENA portal API (read_run result) with each accession to resolve run-level metadata, especially the fastq_ftp URLs for both the IP and its matching control, then download the FASTQ files. Treat the retrieved FASTQs as the raw input for the downstream HOT-region analysis chain: bbduk adapter/quality trimming, Bowtie alignment, samtools filtering of unmapped/multimapping reads, Bedtools coverage over HOT region intervals, and genomation-based log2(IP/control) computation. The rationale is that detecting false-positive enrichment on HOT regions requires reprocessing raw reads for ChIP-ed proteins that are absent from the cell, not reusing deposited peak calls.

## Related tools

- **bbduk** (Trims adapters and low-quality bases from the ENA-downloaded FASTQ reads before alignment)
- **Bowtie** (Aligns the retrieved FASTQ reads to the reference genome as the first validation that downloads are usable)
- **samtools** (Filters, sorts, and indexes Bowtie alignments from the retrieved reads (removing unmapped/multimapping reads))
- **Bedtools** (Computes read coverage over HOT region intervals from the alignments derived from the downloaded data)
- **genomation** (R/Bioconductor package used to compute log2(IP/control) fold enrichment over HOT regions from the retrieved-experiment coverage)

## Examples

```
curl "https://www.ebi.ac.uk/ena/portal/api/filereport?accession=<RUN_ACCESSION>&result=read_run&fields=run_accession,fastq_ftp,fastq_md5" -o ko_tf_run.tsv
```

## Evaluation signals

- Every accession in Supplementary Table S1 resolves to a non-empty FASTQ pair (one IP, one matching control), yielding 2 × 25 downloaded runs
- ENA run metadata (fastq_ftp paths, experiment/sample names) is consistent with the accession-to-KO-transcription-factor mapping in Table S1
- Retrieved FASTQs trim and align cleanly: bbduk/Bowtie/samtools produce non-empty filtered, sorted, indexed BAMs for each run
- Bedtools coverage over HOT region intervals is computable for all 25 experiments, and per-experiment log2(IP/control) values can be written to ko_hot_enrichment.csv
- Re-running the retrieval yields byte-identical files for the same accession set (reproducibility check)

## Limitations

- A lack of signal in a KO ChIP-seq experiment can also stem from technical conditions such as low number of reads, low library complexity, or unsuccessful IP, so retrieved data may be unusable even when downloaded correctly
- With currently available data it is not possible to estimate the proportion of antibody-specific error from pull-down of non-target proteins vs. direct binding to polynucleotide epitopes, so retrieval alone cannot disambiguate artifact mechanisms
- Accession-to-experiment metadata depends on the accuracy of Supplementary Table S1; the article's own abstract section lacks explicit accession metadata, requiring cross-checking against the table

## Evidence

- [supplementary] Download the KO transcription factor ChIP-seq fastq files (IP and matching control) from ENA using the accession numbers listed in Supplementary Table S1.: [evidence span withheld — anchor status: not_found]
- [intro] Large scale projects such as ENCODE (1) and modENCODE (2) used this technology to find the binding sites of hundreds of proteins in multiple species.: "Large scale projects such as ENCODE (1) and modENCODE (2) used this technology to find the binding sites of hundreds of proteins in multiple species."
- [abstract] Using ChIP-seq data sets for knocked-out transcription factors, we demonstrate presence of false positive signals on HOT regions.: "Using ChIP-seq data sets for knocked-out transcription factors, we demonstrate presence of false positive signals on HOT regions."
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
- [discussion] a lack of signal in a KO ChIP-seq experiment might also be caused by technical conditions such as low number of reads, low library complexity or unsuccessful IP.: "a lack of signal in a KO ChIP-seq experiment might also be caused by technical conditions such as low number of reads, low library complexity or unsuccessful IP."
