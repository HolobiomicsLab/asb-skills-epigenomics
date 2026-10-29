---
name: bisulfite-read-alignment
description: Use when raw MethylC-seq or bisulfite-treated sequencing reads (e.g.
  FASTQ from the Lister et al. IMR90/H1 cell lines) must be mapped to a reference
  genome before methylation count extraction and DMR identification.
when_to_use_negative:
- The input is already a Bismark-aligned SAM/BAM file — proceed directly to methylation
  count extraction with extract_CpG_data.py instead of re-aligning.
- The reads are not from bisulfite-treated sequencing (e.g. standard WGS or RNA-seq)
  — in silico bisulfite conversion in Bismark would be inappropriate.
- The goal is DMR testing on existing merged CpG count tables (*.cov) — skip alignment
  and go to combine_CpG_sites.py / findDMRs.r.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3675
tools:
- name: Bismark
  role: Performs in silico bisulfite conversion of reads and reference genome, then
    bisulfite mapping and methylation calling in a single step
  repo: https://github.com/jsh58/DMRfinder
- name: Bowtie2
  role: Short-read aligner used by Bismark to align bisulfite-converted reads to the
    reference
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: Samtools
  role: Optional; converts Bismark BAM output to SAM (or SAM to BAM) when piping into
    extract_CpG_data.py
- name: DMRfinder
  role: 'Downstream consumer of the aligned SAM/BAM: extract_CpG_data.py requires
    the header, ''XM'' and ''XG'' fields Bismark produces'
  repo: https://github.com/jsh58/DMRfinder
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:49c706a7d99a32ff260139e254dbd1e9b5c5cc4936ec0febece9cf956cfab235
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsh58/DMRfinder
  tools:
  - Bismark
  - Bowtie2
  - DMRfinder
  - DSS
  - Python
  - R
  - Samtools
  - bsseq
  - combine_CpG_sites.py
  - extract_CpG_data.py
  - findDMRs.r
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3675
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# bisulfite-read-alignment

## Summary

Aligns high-throughput bisulfite-treated MethylC-seq reads to a reference genome using Bismark (Bowtie2 backend), producing SAM/BAM alignments with per-cytosine methylation strings that downstream DMRfinder steps consume.

## When to use

Use when raw MethylC-seq or bisulfite-treated sequencing reads (e.g. FASTQ from the Lister et al. IMR90/H1 cell lines) must be mapped to a reference genome before methylation count extraction and DMR identification.

## When NOT to use

- The input is already a Bismark-aligned SAM/BAM file — proceed directly to methylation count extraction with extract_CpG_data.py instead of re-aligning.
- The reads are not from bisulfite-treated sequencing (e.g. standard WGS or RNA-seq) — in silico bisulfite conversion in Bismark would be inappropriate.
- The goal is DMR testing on existing merged CpG count tables (*.cov) — skip alignment and go to combine_CpG_sites.py / findDMRs.r.

## Inputs

- Single-end or paired-end FASTQ reads from a bisulfite-treated (MethylC-seq) run, e.g. C1.fastq, C2.fastq
- Reference genome directory/path for Bismark index preparation

## Outputs

- Bismark SAM/BAM alignment files (e.g. C1_bismark_bt2.bam) containing SAM headers, 'XM' methylation strings and 'XG' genome designations
- Per-read methylation calls classified by sequence context (CpG/CHG/CHH), reported with respect to the forward strand

## How to apply

First run `bismark_genome_preparation` on the reference genome, then align reads with Bismark, which performs an in silico bisulfite conversion of both reads and genome before calling Bowtie2. Use Bismark v0.15.0 as recommended, because version 0.16.0 introduced a bug affecting ambiguous read alignments. Keep the alignment output as SAM/BAM carrying the header, 'XM' methylation strings and 'XG' genome designations, since DMRfinder's extract_CpG_data.py requires these fields. Do not apply Bismark's deduplication script, which the authors discourage because it keeps only the first read at a position regardless of sequence or methylation information; if samtools is installed, convert output to BAM via `samtools view -h` when piping into extraction.

## Related tools

- **Bismark** (Performs in silico bisulfite conversion of reads and reference genome, then bisulfite mapping and methylation calling in a single step) — https://github.com/jsh58/DMRfinder
- **Bowtie2** (Short-read aligner used by Bismark to align bisulfite-converted reads to the reference) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **Samtools** (Optional; converts Bismark BAM output to SAM (or SAM to BAM) when piping into extract_CpG_data.py)
- **DMRfinder** (Downstream consumer of the aligned SAM/BAM: extract_CpG_data.py requires the header, 'XM' and 'XG' fields Bismark produces) — https://github.com/jsh58/DMRfinder

## Examples

```
bismark_genome_preparation /path/to/genome; bismark /path/to/genome C1.fastq
```

## Evaluation signals

- Alignment files carry a SAM header plus per-read 'XM' methylation strings and 'XG' genome designations, as required by extract_CpG_data.py
- Bismark version is 0.15.0 (the 0.16.0 ambiguous-alignment bug is avoided)
- Only reads with a single optimal alignment are retained for methylation calling
- Methylation calls are classified into CpG/CHG/CHH contexts and reported with respect to the forward strand (reverse-strand calls appear on guanine bases)
- Downstream extraction produces merged, sorted *.cov tables consistent with the alignment-derived counts (e.g. ~9,378 s extraction time on IMR90 replicate 1a rather than Bismark's 22,514 s)

## Limitations

- Bismark v0.16.0 contains a bug related to ambiguous read alignments; v0.15.0 is recommended until fixed.
- Bismark's built-in deduplication script is discouraged — it arbitrarily keeps the first read at a position regardless of sequence or methylation information.
- If Samtools is not installed, Bismark output will be gzip-compressed rather than BAM, though DMRfinder can analyze either format.

## Evidence

- [readme] Bismark performs an in silico bisulfite conversion of both the reads and the reference genome, and then uses Bowtie2 to align the reads to the reference.: "Bismark [[1](#ref1)] performs an in silico bisulfite conversion of both the reads and the reference genome, and then uses Bowtie2 [[3](#ref3)] to align the reads to the reference."
- [readme] A bug related to ambiguous read alignments crept into version 0.16.0 of Bismark; version 0.15.0 is recommended until fixed.: "A [bug](https://github.com/FelixKrueger/Bismark/issues/56) related to ambiguous read alignments crept into version 0.16.0 of Bismark. [Until this is fixed](https://github.com/FelixKrueger/Bismark/pull"
- [readme] The input SAM alignment file must have a header, 'XM' methylation strings and 'XG' genome designations.: "-i <input>    SAM alignment file produced by Bismark (must have
                    a header, 'XM' methylation strings, and 'XG'
                    genome designations; can use '-' for stdin)"
- [methods] The high-throughput sequence reads from a MethylC-seq run must first be aligned to a reference genome.: "the high-throughput sequence reads from a MethylC-seq run first must be aligned to a reference genome"
- [readme] Reads with a single optimal alignment are further analyzed for methylation status.: "Reads with a single optimal alignment are further analyzed for methylation status; reference cytosines that were sequenced as thymines are labeled unmethylated, and those that remained as cytosines"
