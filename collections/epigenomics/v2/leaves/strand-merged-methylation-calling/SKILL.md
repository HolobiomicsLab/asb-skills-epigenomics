---
name: strand-merged-methylation-calling
description: Use when Bismark-aligned MethylC-seq reads (SAM/BAM with XM methylation
  strings and XG genome designations) must be converted into a sorted table of per-CpG
  methylation counts, including novel CpG sites absent from the reference genome.
when_to_use_negative:
- Input is raw FASTQ rather than Bismark-aligned SAM/BAM — reads must first be aligned
  to the reference genome with Bismark/Bowtie2
- The goal is a Bismark-native methylation extractor output restricted to reference-genome
  CpG sites — DMRfinder deliberately includes novel CpG sites created by variants
  (e.g. rs11348696)
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: DMRfinder (extract_CpG_data.py)
  role: Converts Bismark aligner output into a table of methylated/unmethylated counts
    per CpG site, merging strand counts and including novel CpG sites
  repo: https://github.com/jsh58/DMRfinder
- name: Bismark
  role: Aligns bisulfite-converted reads and produces the SAM file with 'XM' methylation
    strings and 'XG' genome designations consumed by extract_CpG_data.py
  repo: https://github.com/FelixKrueger/Bismark
- name: samtools
  role: Optionally used to stream BAM alignments ('samtools view -h') into extract_CpG_data.py
    via stdin
  repo: http://www.htslib.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:eefb9a887bffc4843024d496ebae153e7789aae02f5a12300c23a733d87243c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsh58/DMRfinder
  tools:
  - Bismark
  - DMRfinder
  - DMRfinder (extract_CpG_data.py)
  - Python
  - extract_CpG_data.py
  - samtools
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# strand-merged-methylation-calling

## Summary

Extract per-CpG methylated/unmethylated counts from Bismark SAM alignments using DMRfinder's extract_CpG_data.py, merging cytosine and guanine (strand) counts per CpG site. Unlike Bismark's own methylation extractor, this includes novel CpG sites created by sequence variants.

## When to use

Use when Bismark-aligned MethylC-seq reads (SAM/BAM with XM methylation strings and XG genome designations) must be converted into a sorted table of per-CpG methylation counts, including novel CpG sites absent from the reference genome.

## When NOT to use

- Input is raw FASTQ rather than Bismark-aligned SAM/BAM — reads must first be aligned to the reference genome with Bismark/Bowtie2
- The goal is a Bismark-native methylation extractor output restricted to reference-genome CpG sites — DMRfinder deliberately includes novel CpG sites created by variants (e.g. rs11348696)

## Inputs

- Bismark-produced SAM alignment file (must contain a header, 'XM' methylation strings, and 'XG' genome designations); BAM acceptable via 'samtools view -h' piping to stdin ('-i -')

## Outputs

- Tab-delimited CpG methylation-count table with columns chrom, chromStart, chromEnd, percent methylation, methylated counts, unmethylated counts, sorted by SAM-header chromosome order then 1-based position
- Optional ordered-chromosomes file via -e; optional strand column via -s

## How to apply

Pipe the Bismark alignment through 'samtools view -h' into extract_CpG_data.py, which reads a SAM file with a header, 'XM' methylation strings, and 'XG' genome designations from a file or stdin ('-i -'). The script parses each alignment's methylation calls — labeling reference cytosines sequenced as thymines as unmethylated and those remaining as cytosines as methylated, with all calls made on the forward strand. It merges counts by summing the cytosine and guanine tallies of each CpG site (needed because reads align to different strands) and includes novel CpG sites not present in the reference, which Bismark's own extraction omits. Apply the minimum coverage filter (-m, default 1) and emit the six-column tab-delimited table (chrom, chromStart, chromEnd, percent methylation, methylated counts, unmethylated counts), sorted by chromosome order from the SAM header then by 1-based cytosine position.

## Related tools

- **DMRfinder (extract_CpG_data.py)** (Converts Bismark aligner output into a table of methylated/unmethylated counts per CpG site, merging strand counts and including novel CpG sites) — https://github.com/jsh58/DMRfinder
- **Bismark** (Aligns bisulfite-converted reads and produces the SAM file with 'XM' methylation strings and 'XG' genome designations consumed by extract_CpG_data.py) — https://github.com/FelixKrueger/Bismark
- **samtools** (Optionally used to stream BAM alignments ('samtools view -h') into extract_CpG_data.py via stdin) — http://www.htslib.org/

## Examples

```
samtools view -h C1_bismark_bt2.bam | python extract_CpG_data.py -i - -o C1.cov -v
```

## Evaluation signals

- On a small sample input, the produced count table matches the reference implementation's output (identical rows, columns, and count values)
- Output is a six-column table (chrom, chromStart, chromEnd, percent methylation, methylated counts, unmethylated counts)
- Output is sorted by chromosome order from the SAM header, then by 1-based position of the CpG cytosine
- Cytosine and guanine counts of each CpG site are summed (strand-merged), not reported separately
- Positions filtered by the -m minimum coverage threshold (default 1) are absent from the output

## Limitations

- Input SAM must contain a header, 'XM' methylation strings, and 'XG' genome designations — older Bismark versions lacking these tags are incompatible
- Methylation calls are made with respect to the forward strand; for reverse-strand reads, calls appear on the guanine bases, so counts must be merged to recover per-CpG totals
- Because novel CpG sites (e.g. from variants like rs11348696) are included, output will differ from Bismark's methylation extractor and positions may not exist in the reference genome

## Evidence

- [methods] The DMRfinder script `extract_CpG_data.py` converts the output from Bismark's aligner into a table of methylated/unmethylated counts at each CpG site: "The DMRfinder script `extract_CpG_data.py` converts the output from Bismark's aligner into a table of methylated/unmethylated counts at each CpG site"
- [readme] Reference cytosines unmethylated vs methylated: "reference cytosines that were sequenced as thymines are labeled unmethylated, and those that remained as cytosines are labeled methylated"
- [readme] The script also merges methylation counts: "The script also merges methylation counts, in that it sums counts for both the cytosine and the guanine of each CpG site"
- [readme] Sort order and novel CpG sites: "The table is sorted by chromosome (based on the order of reference sequence names in the header of the SAM) and position, which is the 1-based coordinate of the cytosine of the CpG"
