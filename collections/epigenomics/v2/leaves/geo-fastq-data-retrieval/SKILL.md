---
name: geo-fastq-data-retrieval
description: 'Use when a spikChIP-style spike-in ChIP-seq analysis needs its starting
  raw data: GEO accessions such as GSE60104 must be resolved to per-sample FASTQ files
  (Jurkat_K79 0–100% spike-in fractions) before any mapping with Bowtie can begin.'
when_to_use_negative:
- The FASTQ or BAM files are already available locally (the skill only covers retrieval
  from GEO, not mapping or normalization)
- Only normalized outputs are needed and the pre-computed spikChIP tracks deposited
  in the UCSC session (https://genome.ucsc.edu/s/DiCroceLab/spikChIP_NAR%2DGB_2021)
  can be downloaded instead
- The experiment uses a GEO accession outside the article's deposited datasets (GSE60104,
  GSE64243, GSE102882), so sample composition and spike-in fractions cannot be verified
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3169
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:5de5ad1d39e65a7279b443f717778d6f843b81709ad2055adc77ff46cbf17683
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
  - Samtools
  - SeqCode
  - gawk
  - spikChIP
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# geo-fastq-data-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves the raw ChIP-seq FASTQ reads and reference genomes needed by the spikChIP pipeline by downloading the H3K79me2 Jurkat libraries (GSM1465004–GSM1465008) from GEO accession GSE60104, together with the hg19 (human sample) and dm3 (fruit fly spike-in) reference sequences that must be combined into the synthetic human+fly Bowtie index.

## When to use

Use when a spikChIP-style spike-in ChIP-seq analysis needs its starting raw data: GEO accessions such as GSE60104 must be resolved to per-sample FASTQ files (Jurkat_K79 0–100% spike-in fractions) before any mapping with Bowtie can begin.

## When NOT to use

- The FASTQ or BAM files are already available locally (the skill only covers retrieval from GEO, not mapping or normalization)
- Only normalized outputs are needed and the pre-computed spikChIP tracks deposited in the UCSC session (https://genome.ucsc.edu/s/DiCroceLab/spikChIP_NAR%2DGB_2021) can be downloaded instead
- The experiment uses a GEO accession outside the article's deposited datasets (GSE60104, GSE64243, GSE102882), so sample composition and spike-in fractions cannot be verified

## Inputs

- GEO study accession (e.g. GSE60104) with member sample accessions GSM1465004–GSM1465008
- GSM sample pages (https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM1465004 ... GSM1465008)
- hg19 reference genome sequence (UCSC)
- dm3 reference genome sequence (UCSC)

## Outputs

- Raw ChIP-seq FASTQ files for the H3K79me2 Jurkat libraries at 0/25/50/75/100% spike-in fractions
- hg19 and dm3 FASTA reference sequences ready for concatenation into the synthetic human+FLY genome index

## How to apply

Resolve the study-level accession (GSE60104) on the NCBI GEO website and enumerate its member samples, which for this workflow are the H3K79me2 Jurkat libraries with 0%, 25%, 50%, 75% and 100% spike-in (GSM1465004, GSM1465005, GSM1465006, GSM1465007, GSM1465008), each identified by sample names like [Jurkat_K79_25%_R1]. Download the raw FASTQ files for every sample from each GSM's acc.cgi page (or via the GEO download endpoints / SRA links provided there). Also download the hg19 and dm3 reference genome sequences from UCSC, since spikChIP requires a combined human+fly genome: the dm3 chromosomes must be renamed with the mandatory 'FLY' suffix when building the chromInfo file and Bowtie indexes. Before running spikChIP, confirm that the downloaded FASTQs correspond to the intended conditions, because the spikChIP README states that BAM files must be generated with a mapping tool prior to running the program by the user (i.e. retrieval and mapping are user-supplied prerequisites).

## Examples

```
wget https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM1465005
```

## Evaluation signals

- Each GSM accession (GSM1465004–GSM1465008) resolves to a downloadable FASTQ with a sample name matching the expected condition (e.g. Jurkat_K79_25%_R1)
- The number of retrieved FASTQ files equals the five spike-in mixture fractions (0%, 25%, 50%, 75%, 100%) described for the H3K79me2 Jurkat experiment
- Downloaded hg19 and dm3 FASTAs yield chromosome sizes consistent with the ChromInfo.txt file, including dm3 chromosomes renamed with the 'FLY' suffix
- Read counts per FASTQ are sufficient to pass the downstream Bowtie mapping step, given spikChIP's guidance that spike-in reads should reach at least one million reads (~2%–5% of the experimental genome) and that BAM files with <1M reads require the -d flag

## Limitations

- The final amount of spike-in reads retrieved depends on the mixture ratio, antibody quality and target abundance, so some samples may fall below the recommended ~1M spike-in reads and require the spikChIP -d option
- Retrieval is a user prerequisite: the spikChIP README notes that BAM files must be generated with a mapping tool (Bowtie, multi-locus reads discarded) prior to running the program, so FASTQs alone are not consumable by spikChIP directly

## Evidence
