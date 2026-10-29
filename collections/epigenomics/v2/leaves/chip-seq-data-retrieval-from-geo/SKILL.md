---
name: chip-seq-data-retrieval-from-geo
description: Use when a differential ChIP-seq analysis needs its raw aligned reads
  (BAM) fetched from public repositories, specifically when reproducing the ENCODE
  H1-hESC vs K562 histone mark comparison via GEO sample accessions.
when_to_use_negative:
- The analysis requires unaligned FASTQ reads or a different genome assembly than
  hg19 — the article used only the aligned hg19 BAM files.
- The target samples are not covered by the article's listed GEO accessions (e.g.
  the Roadmap Epigenomics SRA runs SRR019561–SRR029349 or rat rn4 data use different
  accession routes).
- Peak-like (narrow) histone marks or transcription factor ChIP-seq are the target
  — histoneHMM is designed for broad genomic footprints like H3K27me3 and H3K9me3.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0607
tools:
- name: samtools
  role: Removes duplicated reads from the downloaded BAM files prior to histoneHMM
    preprocessing
- name: histoneHMM
  role: Downstream consumer of the retrieved BAM files; its preprocessing functions
    (getBins, countBamInGRanges) bin the genome and count reads
  repo: https://github.com/matthiasheinig/histoneHMM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:ae59f82a13176224bc029a6df49d7991243a67b305154efc6b28b63dbc9bb260
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matthiasheinig/histoneHMM
  tools:
  - Bioconductor
  - C++
  - R
  - histoneHMM
  - samtools
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0607
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chip-seq-data-retrieval-from-geo

## Summary

Retrieves aligned ChIP-seq BAM files for histone marks H3K9me3, H3K79me2, H3K36me3 and H3K27me3 in H1-hESC and K562 from the ENCODE GEO sample accessions cited in the histoneHMM article (hg19 assembly), providing the raw input for downstream bivariate HMM differential analysis.

## When to use

Use when a differential ChIP-seq analysis needs its raw aligned reads (BAM) fetched from public repositories, specifically when reproducing the ENCODE H1-hESC vs K562 histone mark comparison via GEO sample accessions.

## When NOT to use

- The analysis requires unaligned FASTQ reads or a different genome assembly than hg19 — the article used only the aligned hg19 BAM files.
- The target samples are not covered by the article's listed GEO accessions (e.g. the Roadmap Epigenomics SRA runs SRR019561–SRR029349 or rat rn4 data use different accession routes).
- Peak-like (narrow) histone marks or transcription factor ChIP-seq are the target — histoneHMM is designed for broad genomic footprints like H3K27me3 and H3K9me3.

## Inputs

- GEO sample accession numbers (GSM1003585, GSM733748, GSM733725, GSM1003547, GSM733658, GSM733714, GSM733653, GSM733776, GSM1003524, GSM1003576)
- Histone mark and cell line mapping (H3K9me3, H3K79me2, H3K36me3, H3K27me3 in H1-hESC and K562)
- hg19 genome assembly reference

## Outputs

- Aligned ChIP-seq BAM files per sample/mark/cell-line on hg19
- Duplicate-removed BAM files (after samtools deduplication)

## How to apply

Identify the GEO sample accession numbers listed in the article's discussion (e.g. GSM1003585, GSM733748, GSM733725, GSM1003547, GSM733658, GSM733714, GSM733653, GSM733776, GSM1003524, GSM1003576), noting the article lists each accession twice so duplicates must be collapsed. Download the aligned reads (genome assembly hg19) for each sample/mark/cell-line combination from GEO, mapping each accession to its histone mark and cell line (H1-hESC or K562). After download, remove duplicated reads from each BAM file using samtools, since the histoneHMM workflow requires duplicate removal before binning the genome into 1000 bp bins. Validate retrieval by confirming every accession from the article resolves to a BAM file on hg19 and that each of the four histone marks is represented for both cell lines.

## Related tools

- **samtools** (Removes duplicated reads from the downloaded BAM files prior to histoneHMM preprocessing)
- **histoneHMM** (Downstream consumer of the retrieved BAM files; its preprocessing functions (getBins, countBamInGRanges) bin the genome and count reads) — https://github.com/matthiasheinig/histoneHMM

## Evaluation signals

- All non-redundant GEO accessions from the article's discussion resolve to a downloadable aligned BAM file on hg19.
- Each of the four histone marks (H3K9me3, H3K79me2, H3K36me3, H3K27me3) has one sample for H1-hESC and one for K562.
- Duplicate reads are removed from every BAM with samtools before counting, matching the article's preprocessing step.
- Read counts per 1000 bp bin stay bounded after truncation at the upper 0.1 percentile (maxq), indicating correct sample preparation for run.bivariate.hmm.
- Downstream differential-region totals are comparable in magnitude to the published Table 2 values (843.03, 284.58, 324.85, 591.27 Mb).

## Limitations

- The article lists ENCODE GEO accession numbers with duplicates (each accession appears twice), making the exact sample list ambiguous; the curator must deduplicate accessions.
- No version numbers are given for samtools or the competing tools, so exact tool versions used on the retrieved data are not documented.
- Only aligned reads (hg19) are referenced; raw FASTQ retrieval paths are not described in the article.

## Evidence

- [discussion] ENCODE GEO accessions with duplicate listing: "(GEO sample accession numbers: GSM1003585, GSM1003585, GSM733748, GSM733748, GSM733725, GSM733725, GSM1003547, GSM1003547, GSM733658, GSM733658, GSM733714, GSM733714, GSM733653, GSM733653, GSM733776,"
- [discussion] hg19 aligned reads used: "We used the aligned reads (genome assembly hg19)"
- [discussion] Duplicate removal with samtools: "and removed duplicated reads using samtools"
- [readme] README install command for downstream package: "devtools::install_github("matthiasheinig/histoneHMM")"
