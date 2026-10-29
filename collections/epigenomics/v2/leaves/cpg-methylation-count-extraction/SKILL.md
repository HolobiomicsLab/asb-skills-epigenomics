---
name: cpg-methylation-count-extraction
description: Use when starting from a Bismark-aligned MethylC-seq SAM/BAM file (with
  XM methylation strings and XG genome designations) and per-CpG methylation count
  tables (*.cov) are needed for DMR analysis or linkage analysis.
when_to_use_negative:
- You need counts restricted strictly to CpG sites present in the reference genome
  (extract_CpG_data.py also reports novel CpG sites created by variants, unlike Bismark's
  methylation extractor)
- The alignment file lacks a SAM header, 'XM' methylation strings, or 'XG' genome
  designations, or was produced by an aligner other than Bismark
- You only need raw per-read methylation calls without merging cytosine/guanine counts
  per CpG site
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: DMRfinder (extract_CpG_data.py)
  role: Converts Bismark alignment output into a merged, sorted table of methylated/unmethylated
    counts at each CpG site; also supports linked methylation collection via -n
  repo: https://github.com/jsh58/DMRfinder
- name: Bismark
  role: Produces the required SAM alignments with XM/XG tags after in silico bisulfite
    conversion; version 0.15.0 recommended
  repo: https://github.com/FelixKrueger/Bismark
- name: Samtools
  role: Converts Bismark BAM output to headered SAM via `samtools view -h` for piping
    into extract_CpG_data.py
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_001
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:4bb126548f2785d2949e54c0706f0d396c136072fbc329e11411e5e4dc3fe082
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsh58/DMRfinder
  tools:
  - Bismark
  - Bowtie2
  - DMRfinder
  - DMRfinder (extract_CpG_data.py)
  - DSS
  - Python
  - R
  - Samtools
  - bsseq
  - combine_CpG_sites.py
  - extract_CpG_data.py
  - findDMRs.r
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# cpg-methylation-count-extraction

## Summary

Extracts per-CpG methylated/unmethylated counts from Bismark SAM alignments using DMRfinder's extract_CpG_data.py, which merges cytosine and guanine tallies per CpG site and, unlike Bismark, also includes novel CpG sites created by sequence variants.

## When to use

Use when starting from a Bismark-aligned MethylC-seq SAM/BAM file (with XM methylation strings and XG genome designations) and per-CpG methylation count tables (*.cov) are needed for DMR analysis or linkage analysis.

## When NOT to use

- You need counts restricted strictly to CpG sites present in the reference genome (extract_CpG_data.py also reports novel CpG sites created by variants, unlike Bismark's methylation extractor)
- The alignment file lacks a SAM header, 'XM' methylation strings, or 'XG' genome designations, or was produced by an aligner other than Bismark
- You only need raw per-read methylation calls without merging cytosine/guanine counts per CpG site

## Inputs

- Headered SAM alignment file from Bismark containing 'XM' methylation strings and 'XG' genome designations (or a BAM converted via `samtools view -h`, optionally piped via stdin '-')
- Optional BED file of genomic regions of interest (chromosome, 1-based start, end, unique region name) for linked methylation collection via -n

## Outputs

- Six-column tab-delimited methylation count table (*.cov): chrom, chromStart, chromEnd, percent methylation, methylated counts, unmethylated counts, sorted by SAM-header chromosome order then 1-based position
- Optional <file>_linked.txt file encoding per-read, per-region linkage (0 = unmethylated CpG, 1 = methylated, - = no data) for each BED region

## How to apply

Pipe a headered SAM through stdin or pass the file directly: `samtools view -h sample_bismark_bt2.bam | python extract_CpG_data.py -i - -o sample.cov -v`. The script parses each alignment's XM methylation string, labeling reference cytosines sequenced as thymines as unmethylated and those remaining as cytosines as methylated, with all calls made on the forward strand. It merges counts by summing the cytosine and guanine tallies of each CpG site (this occurs when reads align to different strands), applies a minimum coverage filter (-m, default 1), and emits a six-column tab-delimited table (chrom, chromStart, chromEnd, percent methylation, methylated counts, unmethylated counts) sorted by chromosome order from the SAM header then by 1-based position. Unlike Bismark's methylation extractor, extract_CpG_data.py incorporates novel CpG sites created by natural variants (e.g., rs11348696, dbSNP build 150). Optionally pass a BED file via -n to collect spatially-linked methylation data for reads covering at least one CpG in each region, writing a <file>_linked.txt with 0/1/- encoding (unmethylated/methylated/no data) per read.

## Related tools

- **DMRfinder (extract_CpG_data.py)** (Converts Bismark alignment output into a merged, sorted table of methylated/unmethylated counts at each CpG site; also supports linked methylation collection via -n) — https://github.com/jsh58/DMRfinder
- **Bismark** (Produces the required SAM alignments with XM/XG tags after in silico bisulfite conversion; version 0.15.0 recommended) — https://github.com/FelixKrueger/Bismark
- **Samtools** (Converts Bismark BAM output to headered SAM via `samtools view -h` for piping into extract_CpG_data.py)

## Examples

```
samtools view -h C1_bismark_bt2.bam | python extract_CpG_data.py -i - -o C1.cov -v
```

## Evaluation signals

- Output table is sorted by chromosome order from the SAM header, then by 1-based position of the cytosine of the CpG
- For each CpG site, methylated and unmethylated counts are the sum of cytosine and guanine tallies from both strands
- Every reported site meets the minimum coverage threshold (-m, default 1)
- On a small sample input, the produced count table matches the reference implementation's output (identical rows, columns, and count values)
- A regenerated <file>_linked.txt from a BED region file reproduces the per-region read-level 0/1/- encoding format

## Limitations

- Novel CpG sites created by natural variants are included in the output, so results are not limited to reference-genome CpG sites
- All methylation calls are made with respect to the forward strand, so for reads aligning to the reverse strand, calls appear on the guanine bases
- extract_CpG_data.py requires a headered SAM with 'XM' and 'XG' tags; BAM must first be converted with `samtools view -h`
- A Bismark bug related to ambiguous read alignments affects versions after 0.16.0; version 0.15.0 is recommended until fixed

## Evidence

- [methods] The DMRfinder script `extract_CpG_data.py` converts the output from Bismark's aligner into a table of methylated/unmethylated counts at each CpG site: "The DMRfinder script `extract_CpG_data.py` converts the output from Bismark's aligner into a table of methylated/unmethylated counts at each CpG site"
- [readme] The script also merges methylation counts, summing cytosine and guanine tallies: "The script also merges methylation counts, in that it sums counts for both the cytosine and the guanine of each CpG site"
- [results] Novel CpG sites are included, unlike Bismark: "a known variant (rs11348696, dbSNP build 150) that results in the creation of a novel CpG site"
- [readme] Required SAM input must have header, XM strings, and XG designations: "SAM alignment file produced by Bismark (must have
                    a header, 'XM' methylation strings, and 'XG'
                    genome designations; can use '-' for stdin)"
- [readme] Reads aligning to the reverse strand have methylation calls on guanine bases: "All methylation calls are made with respect to the forward strand, so for reads that align to the reverse strand of the genome, methylation calls appear on the guanine bases"
