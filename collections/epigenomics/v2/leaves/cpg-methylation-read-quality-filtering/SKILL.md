---
name: cpg-methylation-read-quality-filtering
description: Use when bisulfite-sequencing FASTA reads (e.g. DDO1/DAO amplicons from
  PRJEB28662) must be quality-filtered prior to epiallele-frequency or methylated-core
  extraction with ampliMethProfiler or MethCoresProfiler.
when_to_use_negative:
- The input is already a table of epiallele configurations or MethCoresProfiler output
  (Tab_Epialleles / ComposCore) — filtering has already been applied upstream.
- The data are not bisulfite-sequencing reads of targeted CpG-containing amplicons
  (e.g. single-CpG methylation arrays), where per-molecule phased configurations are
  unavailable.
- Reads cannot be aligned to a reference with known CpG positions and primer sequences,
  making the ±50% length, 80% primer-similarity and 60% alignment criteria inapplicable.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3674
tools:
- name: ampliMethProfiler
  role: Applies the quality filters (length ±50%, ≥80% primer similarity, ≥98% bisulfite
    efficiency, ≥60% base alignment, removal of ambiguous CpG calls) and extracts
    mCpG configurations in single DNA molecules from FASTA reads
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: PEAR
  role: Merges paired-end FASTQ reads with a minimum 40-nucleotide overlapping region
    before filtering
- name: PRINSEQ
  role: Converts merged, quality-filtered reads to FASTA format and supports mean-Phred/length
    filtering
- name: MethCoresProfiler
  role: Downstream consumer of the filtered reads; its MethCores_Extractor module
    requires the binary EpiHaplotypes file produced after filtering
  repo: https://github.com/84AP/MethCoresProfiler/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:e529ecefaa45ebaa64ca94f026a64a1f204d0f616e38d989ae71727781f7c49f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/84AP/MethCoresProfiler
  tools:
  - EBA
  - MethCoresProfiler
  - PEAR
  - PRINSEQ
  - R
  - ampliMethProfiler
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cpg-methylation-read-quality-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the ampliMethProfiler read-quality filter chain (length ±50% of reference, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment, removal of ambiguous CpG calls) before extracting single-molecule mCpG configurations from bisulfite sequencing reads, so that downstream epiallele and methylated-core frequency estimates are not corrupted by low-confidence methylation calls.

## When to use

Use when bisulfite-sequencing FASTA reads (e.g. DDO1/DAO amplicons from PRJEB28662) must be quality-filtered prior to epiallele-frequency or methylated-core extraction with ampliMethProfiler or MethCoresProfiler.

## When NOT to use

- The input is already a table of epiallele configurations or MethCoresProfiler output (Tab_Epialleles / ComposCore) — filtering has already been applied upstream.
- The data are not bisulfite-sequencing reads of targeted CpG-containing amplicons (e.g. single-CpG methylation arrays), where per-molecule phased configurations are unavailable.
- Reads cannot be aligned to a reference with known CpG positions and primer sequences, making the ±50% length, 80% primer-similarity and 60% alignment criteria inapplicable.

## Inputs

- Paired-end bisulfite-sequencing FASTQ reads (ENA accession PRJEB28662)
- PEAR-merged and PRINSEQ-converted FASTA reads for target regions (e.g. DDO1 −488 to −44 bp from TSS with 6 CpGs; DAO +7 to +334 bp with 4 CpGs)
- Reference sequences including primer regions for the target amplicons

## Outputs

- Filtered set of bisulfite reads retaining only high-confidence methylation calls per molecule
- Single-molecule mCpG configuration (epiallele) calls per sample, suitable as tab-delimited binary EpiHaplotype input to MethCoresProfiler

## How to apply

First merge paired-end FASTQ reads with PEAR (minimum 40 nucleotides overlap) and convert to FASTA with PRINSEQ, retaining reads with mean Phred >33 and length 400–500 nt. Then run ampliMethProfiler on the FASTA reads applying its five quality filters: (i) read length within ±50% of the reference length, (ii) at least 80% sequence similarity of the primer with the corresponding gene, (iii) at least 98% bisulfite efficiency, (iv) alignment of at least 60% of bases with the reference sequences, and (v) removal of reads with ambiguous calls (gaps, A or G) at CpG dinucleotides. The rationale is that ambiguous or inefficiently converted reads would be scored as false non-methylated CpGs, distorting single-epiallele frequencies and the stochastic-vs-deterministic separation later assessed by chi-square tests (P-value ≤ 10−9). Decision point: reads failing any single criterion are discarded entirely rather than trimmed, because the mCpG configuration of each molecule must be complete for MethCoresProfiler's Module 1 (MethCores_Extractor) to compute depth (b4) and two-mCpG co-occurrence statistics.

## Related tools

- **ampliMethProfiler** (Applies the quality filters (length ±50%, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment, removal of ambiguous CpG calls) and extracts mCpG configurations in single DNA molecules from FASTA reads) — https://sourceforge.net/projects/amplimethprofiler/
- **PEAR** (Merges paired-end FASTQ reads with a minimum 40-nucleotide overlapping region before filtering)
- **PRINSEQ** (Converts merged, quality-filtered reads to FASTA format and supports mean-Phred/length filtering)
- **MethCoresProfiler** (Downstream consumer of the filtered reads; its MethCores_Extractor module requires the binary EpiHaplotypes file produced after filtering) — https://github.com/84AP/MethCoresProfiler/

## Examples

```
pear -f sample_R1.fastq -r sample_R2.fastq -o merged && prinseq-lite.pl -fastq merged.fastq -out_format 1 -min_qual_mean 33 -min_len 400 -max_len 500 && ampliMethProfiler --input merged.fastA --reference DDO1_ref.fa --filters length:0.5,primer:0.8,bisulfite:0.98,alignment:0.6
```

## Evaluation signals

- All retained reads satisfy every filter: length within ±50% of the reference length, ≥80% primer similarity, ≥98% bisulfite conversion efficiency, ≥60% bases aligned to the reference, and no ambiguous calls (gaps, A or G) at CpG dinucleotides.
- No retained read contains an unresolved base at any CpG position in the target region (e.g. the 6 CpGs of DDO1 or 4 CpGs of DAO), since such reads must have been removed.
- The filtered output converts losslessly into the tab-delimited binary EpiHaplotype format required by 1_MethCores_Extractor.R, with a matching cgpos file and sample metadata (#SampleID, Tissue, Description, Group, Rep, ID).
- Downstream per-sample epiallele-frequency distributions computed from the filtered reads reproduce the paper's behavior: frequent individual epialleles are found but R1/R2 random and Cond 4 deterministic epialleles show comparable frequency/entropy values in single-epiallele analyses (Supplementary Figure S5C), while only core-level analysis separates them.
- Mean read quality and length distributions of the retained set fall within the stated preprocessing window (mean Phred >33, length 400–500 nt after PEAR merging).

## Limitations

- The filter chain discards reads entirely rather than rescuing partial methylation information; in heterogeneous populations stochastic mCpGs (e.g. hydroxymethylated cytosines) survive filtering and still dilute stable methylated molecules, so single-epiallele frequencies remain unable to isolate stable cores.
- Filters depend on a reference with known primer and CpG positions; ambiguous CpG calls (gaps, A or G) are removed, which can reduce depth below what is needed for significant chi-square annotation (P-value ≤ 10−9).
- ampliMethProfiler itself cannot distinguish deterministic from stochastic epialleles — MethCoresProfiler outperforms EBA, ampliMethProfiler and methclone by separating Cond 4 from R1 and R2 epialleles only after core extraction.

## Evidence
