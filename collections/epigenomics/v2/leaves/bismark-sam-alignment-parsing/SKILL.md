---
name: bismark-sam-alignment-parsing
description: Use when Bismark-produced SAM/BAM alignments from MethylC-seq data must
  be converted into per-CpG methylated/unmethylated count tables, especially when
  novel CpG sites from variants (e.g. rs11348696) should be retained in the output.
when_to_use_negative:
- The input is not a Bismark alignment file with 'XM' and 'XG' tags (e.g. raw FASTQ
  or alignments from a non-bisulfite aligner) — the script requires these SAM fields
- You want methylation counts restricted strictly to CpG sites present in the reference
  genome; DMRfinder intentionally includes novel CpG sites created by variants
- You need to model sets of regions, quantitative traits, or batch effects — these
  are outside DMRfinder's scope
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: DMRfinder
  role: Package providing extract_CpG_data.py, the script that converts Bismark aligner
    output into a table of methylated/unmethylated counts at each CpG site
  repo: https://github.com/jsh58/DMRfinder
- name: extract_CpG_data.py
  role: The specific script executed in this skill; parses SAM alignments, merges
    CpG counts, applies -m minimum coverage (def. 1), and writes the sorted count
    table
  repo: https://github.com/jsh58/DMRfinder
- name: Bismark
  role: Upstream aligner whose in silico bisulfite alignments (BAM/SAM output) are
    the input to this extraction step
  repo: FelixKrueger/Bismark
- name: Python
  role: Runtime for extract_CpG_data.py (tested with Python 2.7.12 and 3.5.2)
- name: Samtools
  role: Optional; used to stream BAM alignments to the script via 'samtools view -h'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:5762688e75388612de0020c6a4d1bf6f7028038f9e2349f8c07a30df3ac4352b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsh58/DMRfinder
  tools:
  - Bismark
  - DMRfinder
  - Python
  - Samtools
  - extract_CpG_data.py
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# bismark-sam-alignment-parsing

## Summary

Reconstructs the first step of the DMRfinder pipeline — the extract_CpG_data.py script that converts Bismark SAM alignments into a sorted, merged table of methylated/unmethylated counts per CpG site, including novel CpG sites absent from the reference genome (unlike Bismark's own methylation extractor).

## When to use

Use when Bismark-produced SAM/BAM alignments from MethylC-seq data must be converted into per-CpG methylated/unmethylated count tables, especially when novel CpG sites from variants (e.g. rs11348696) should be retained in the output.

## When NOT to use

- The input is not a Bismark alignment file with 'XM' and 'XG' tags (e.g. raw FASTQ or alignments from a non-bisulfite aligner) — the script requires these SAM fields
- You want methylation counts restricted strictly to CpG sites present in the reference genome; DMRfinder intentionally includes novel CpG sites created by variants
- You need to model sets of regions, quantitative traits, or batch effects — these are outside DMRfinder's scope

## Inputs

- Bismark-produced SAM alignment file (with header, 'XM' methylation strings, 'XG' genome designations; can use '-' for stdin, e.g. piped from 'samtools view -h' on a BAM file)

## Outputs

- Six-column tab-delimited CpG count table (chrom, chromStart, chromEnd, percent methylation, methylated counts, unmethylated counts), merged and sorted by chromosome order from the SAM header and 1-based position

## How to apply

Read a Bismark SAM alignment file (it must contain a header, 'XM' methylation strings, and 'XG' genome designations), accepting it via stdin (e.g. piped from 'samtools view -h') or as a file. Parse each alignment's 'XM' methylation calls: reference cytosines sequenced as thymines are labeled unmethylated and those remaining as cytosines are labeled methylated, with all calls made on the forward strand (reverse-strand reads carry calls on the guanine bases). Merge counts by summing the cytosine and guanine tallies of each CpG site, and include novel CpG sites not present in the reference — a key difference from Bismark's extractor, which DMRfinder deliberately does not replicate. Apply the minimum coverage filter (-m, default 1) and emit a six-column tab-delimited table (chrom, chromStart, chromEnd, percent methylation, methylated counts, unmethylated counts), sorted by chromosome order from the SAM header then by 1-based cytosine position.

## Related tools

- **DMRfinder** (Package providing extract_CpG_data.py, the script that converts Bismark aligner output into a table of methylated/unmethylated counts at each CpG site) — https://github.com/jsh58/DMRfinder
- **extract_CpG_data.py** (The specific script executed in this skill; parses SAM alignments, merges CpG counts, applies -m minimum coverage (def. 1), and writes the sorted count table) — https://github.com/jsh58/DMRfinder
- **Bismark** (Upstream aligner whose in silico bisulfite alignments (BAM/SAM output) are the input to this extraction step) — FelixKrueger/Bismark
- **Python** (Runtime for extract_CpG_data.py (tested with Python 2.7.12 and 3.5.2))
- **Samtools** (Optional; used to stream BAM alignments to the script via 'samtools view -h')

## Examples

```
samtools view -h C1_bismark_bt2.bam | python extract_CpG_data.py -i - -o C1.cov -v
```

## Evaluation signals

- On a small sample SAM input, the produced count table must match the reference implementation's output exactly (identical rows, columns, and count values)
- Output has six columns per row: chrom, chromStart, chromEnd, percent methylation, methylated counts, unmethylated counts
- Rows are sorted first by chromosome in the order of reference sequence names in the SAM header, then by 1-based cytosine position
- Counts for the cytosine and the guanine of each CpG site are summed into a single row per site
- CpG sites with fewer than -m counts (default 1) are absent from the output

## Limitations

- The SAM input must contain a header, 'XM' methylation strings, and 'XG' genome designations; alignments lacking these fields cannot be parsed
- All methylation calls are made with respect to the forward strand, so reverse-strand reads carry calls on the guanine bases
- DMRfinder does not model sets of regions or quantitative traits and cannot account for batch effects
- A Bismark bug affecting ambiguous read alignments existed in version 0.16.0; version 0.15.0 was recommended until fixed

## Evidence

- [methods] extract_CpG_data.py converts Bismark output: "The DMRfinder script `extract_CpG_data.py` converts the output from Bismark's aligner into a table of methylated/unmethylated counts at each CpG site"
- [readme] README usage and stdin example: "$ samtools view -h C1_bismark_bt2.bam  |  python extract_CpG_data.py  -i -  -o C1.cov  -v"
- [readme] SAM input requirements: "-i <input>    SAM alignment file produced by Bismark (must have
                    a header, 'XM' methylation strings, and 'XG'
                    genome designations; can use '-' for stdin)"
- [readme] Merging and sorting behavior: "The script also merges methylation counts, in that it sums counts for both the cytosine and the guanine of each CpG site"
- [results] Novel CpG sites finding: "a known variant (rs11348696, dbSNP build 150) that results in the creation of a novel CpG site"
- [readme] Forward-strand calling convention: "All methylation calls are made with respect to the forward strand, so for reads that align to the reverse strand of the genome, methylation calls appear on the guanine bases."
