---
name: geo-chipseq-dataset-retrieval
description: Use when a reproduction task cites specific GEO accessions (e.g. GSM2098958
  H3K4me3 mESC, GSM2645501 MLL2 WT) and the raw FASTQ files, mapped BAMs, or BED peak
  lists must be fetched from NCBI GEO before analysis can begin.
when_to_use_negative:
- Data are already locally available as processed BAM/BED files and no accession resolution
  is needed
- The dataset of interest is not deposited in NCBI GEO (e.g. hosted on ENCODE/DDBJ
  or SRA-only without GEO mapping)
- The task concerns visualization or peak-overlap analysis itself rather than acquiring
  the source data
tools:
- name: SeqCode
  role: 'Downstream consumer of the retrieved samples: expects per-sample BAM/SAM
    read files (read via integrated SamTools/HTSlib) and BED peak lists from the GEO
    accessions'
  repo: https://github.com/eblancoga/seqcode
- name: Bowtie
  role: Maps the retrieved FASTQ reads over the genome (mm9 and mm10) after download,
    producing the BAM files SeqCode requires
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:256a5cbe2d993233f43447be0716c498cb0703614a2ccdc2a4c7fb433dca0d75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Bowtie
  - R
  - SAMtools
  - SeqCode
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# geo-chipseq-dataset-retrieval

## Summary

Retrieves the raw high-throughput sequencing data (FASTQ) and derived peak lists behind a publication's ChIP-seq examples by resolving the GEO accessions listed in the article's data table (e.g. Table 4), so downstream SeqCode-based visualization and classification pipelines have the exact inputs used in the paper.

## When to use

Use when a reproduction task cites specific GEO accessions (e.g. GSM2098958 H3K4me3 mESC, GSM2645501 MLL2 WT) and the raw FASTQ files, mapped BAMs, or BED peak lists must be fetched from NCBI GEO before analysis can begin.

## When NOT to use

- Data are already locally available as processed BAM/BED files and no accession resolution is needed
- The dataset of interest is not deposited in NCBI GEO (e.g. hosted on ENCODE/DDBJ or SRA-only without GEO mapping)
- The task concerns visualization or peak-overlap analysis itself rather than acquiring the source data

## Inputs

- GEO accession identifiers (GSMxxxxxxx / SRXxxxxxxx) from the article's data table (e.g. Table 4)
- NCBI GEO repository records for ChIP-seq experiments (mESC H3K4me3, H3K27me3, MLL2 WT/KO)
- Resulting raw FASTQ files per sample, or pre-computed peak lists in BED format (chromosome, initial position, ending position, optional name)

## Outputs

- Per-sample raw ChIP-seq FASTQ files organized by accession and condition
- Mapped BAM files per sample (unaligned reads excluded) for SeqCode input
- BED-format peak lists (e.g. MACS output) for downstream matchpeaks overlap classification

## How to apply

Locate the publication's data table (Table 4 in the SeqCode article) which lists each example experiment's accession (GSM/SRX identifiers, e.g. SRX367147 for H3K4me3 in mESCs from Tee et al., GSM1562339 for RNA-seq from Jacinto et al.) and retrieve the raw data from the NCBI GEO repository. For ChIP-seq reproduction tasks, fetch the FASTQ for each condition replicate pair (e.g. GSM2645501 MLL2 WT vs GSM2645502 MLL2 KO; GSM2645495 H3K4me3 WT vs GSM2645496 H3K4me3 KO from Mas et al.), then map with Bowtie to the appropriate genome assembly (mm9/mm10 for mouse) and exclude unaligned reads from the resulting BAM files, since SeqCode commands read BAM/SAM internally via integrated SamTools/HTSlib. When only peak classification is needed (e.g. the H3K4me3/H3K27me3 active/bivalent/silent task), obtain the peak lists directly in BED format (chromosome, initial position, ending position, optional name) for input to SeqCode's matchpeaks. The rationale is that all SeqCode examples in the paper are defined by these exact accessions, so faithful retrieval guarantees comparability with the published figures.

## Related tools

- **SeqCode** (Downstream consumer of the retrieved samples: expects per-sample BAM/SAM read files (read via integrated SamTools/HTSlib) and BED peak lists from the GEO accessions) — https://github.com/eblancoga/seqcode
- **Bowtie** (Maps the retrieved FASTQ reads over the genome (mm9 and mm10) after download, producing the BAM files SeqCode requires)

## Evaluation signals

- Every accession cited in the reproduction task (e.g. GSM2098958, GSM2098952, GSM2645501-02, GSM2645495-96) resolves to a downloaded FASTQ or BAM file for the correct condition
- Retrieved peak files are in the required BED format with chromosome, initial position, ending position, and optional name columns, usable by SeqCode matchpeaks
- BAM files contain only aligned reads (unaligned reads excluded) and are readable by SeqCode's SamTools/HTSlib-based commands
- Reads map to the correct genome assembly (mm9 or mm10 for mouse mESC samples) matching the article's methods
- Downstream SeqCode outputs (e.g. recoverChIPlevels counts, meta-plots) reproduce the qualitative patterns of the published figures

## Limitations

- The article documents accessions only through its data table (Table 4); supplementary or unreleased datasets may not be retrievable from GEO
- Retrieval alone does not guarantee correct downstream use — the genome assembly choice (mm9 vs mm10) must match the sample, or peak coordinates will be inconsistent
- GEO-hosted data quality and supplementary file layouts vary per submitter, so file naming (FASTQ vs BAM vs BED) must be checked per accession

## Evidence

- [methods] Raw data of the examples introduced throughout the article (Table 4) were retrieved from the NCBI GEO repository74.: "Raw data of the examples introduced throughout the article (Table 4) were retrieved from the NCBI GEO repository"
- [methods] Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3).: "Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3)."
- [readme] Commands working with sequencing data require the user to provide the mapped reads of the experiment (BAM or SAM format).: "Commands working with sequencing data require the user to provide the mapped reads of the experiment (BAM or SAM format)."
- [methods] Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands.: "Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands."
