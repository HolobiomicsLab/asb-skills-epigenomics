---
name: novel-cpg-site-inclusion
description: Use when post-aligning MethylC-seq reads with Bismark and you need per-CpG
  methylated/unmethylated counts that capture novel CpG sites introduced by SNPs (e.g.,
  rs11348696), not just reference-genome CpGs.
when_to_use_negative:
- If your downstream analysis must be restricted strictly to CpG sites annotated in
  the reference genome (e.g., comparing against a fixed reference CpG catalog)
- If the alignment file was not produced by Bismark or lacks 'XM' methylation strings
  and 'XG' genome designations, which the script requires
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: DMRfinder
  role: Provides extract_CpG_data.py, which extracts merged methylated/unmethylated
    CpG counts and incorporates novel CpG sites
  repo: https://github.com/jsh58/DMRfinder
- name: extract_CpG_data.py
  role: The specific DMRfinder script that converts Bismark aligner output into the
    CpG count table
  repo: https://github.com/jsh58/DMRfinder
- name: Bismark
  role: Aligns bisulfite-treated reads and produces the SAM/BAM input (with XM/XG
    tags) consumed by extract_CpG_data.py
  repo: FelixKrueger/Bismark
- name: Samtools
  role: Pipes the BAM as header-bearing SAM text to extract_CpG_data.py via stdin
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:b2b81a4d4b199a19e2baa5fe9e86d54ebb2b64f6644649e0116562e54586f03a
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
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# novel-cpg-site-inclusion

## Summary

Extracts methylation counts from Bismark SAM alignments with DMRfinder's extract_CpG_data.py in a way that includes CpG sites created by natural variants (not present in the reference genome), unlike Bismark's own methylation extractor.

## When to use

Use when post-aligning MethylC-seq reads with Bismark and you need per-CpG methylated/unmethylated counts that capture novel CpG sites introduced by SNPs (e.g., rs11348696), not just reference-genome CpGs.

## When NOT to use

- If your downstream analysis must be restricted strictly to CpG sites annotated in the reference genome (e.g., comparing against a fixed reference CpG catalog)
- If the alignment file was not produced by Bismark or lacks 'XM' methylation strings and 'XG' genome designations, which the script requires

## Inputs

- Bismark-produced SAM alignment file (with header, 'XM' methylation strings, and 'XG' genome designations), via file or stdin

## Outputs

- Six-column tab-delimited count table (chrom, chromStart, chromEnd, percent methylation, methylated counts, unmethylated counts) at each CpG site, including novel CpG sites, sorted by chromosome (SAM-header order) and 1-based position

## How to apply

Pipe a Bismark BAM through 'samtools view -h' into extract_CpG_data.py (input via '-' for stdin); the SAM must have a header, 'XM' methylation strings, and 'XG' genome designations. The script parses each alignment's XM string, labeling reference cytosines sequenced as thymines as unmethylated and those remaining as cytosines as methylated, with all calls on the forward strand. It merges counts by summing cytosine and guanine tallies for each CpG site and — key to this skill — includes novel CpG sites not present in the reference genome, which Bismark's extractor omits. Apply the minimum coverage filter (-m, default 1) and the output is a six-column tab-delimited table (chrom, chromStart, chromEnd, percent methylation, methylated counts, unmethylated counts), sorted by chromosome order from the SAM header then 1-based position.

## Related tools

- **DMRfinder** (Provides extract_CpG_data.py, which extracts merged methylated/unmethylated CpG counts and incorporates novel CpG sites) — https://github.com/jsh58/DMRfinder
- **extract_CpG_data.py** (The specific DMRfinder script that converts Bismark aligner output into the CpG count table) — https://github.com/jsh58/DMRfinder
- **Bismark** (Aligns bisulfite-treated reads and produces the SAM/BAM input (with XM/XG tags) consumed by extract_CpG_data.py) — FelixKrueger/Bismark
- **Samtools** (Pipes the BAM as header-bearing SAM text to extract_CpG_data.py via stdin)

## Examples

```
samtools view -h C1_bismark_bt2.bam | python extract_CpG_data.py -i - -o C1.cov -v
```

## Evaluation signals

- On a small sample input, the produced count table must match the reference implementation's output exactly (identical rows, columns, and count values)
- Output has exactly six tab-delimited columns: chrom, chromStart, chromEnd, percent methylation, methylated counts, unmethylated counts
- Rows are sorted by chromosome in the order of reference sequence names in the SAM header, then by 1-based cytosine position
- CpG sites created by variants (e.g., a SNP like rs11348696 generating a novel CpG) appear in the output, distinguishing it from Bismark's methylation extractor
- Methylated + unmethylated counts meet the -m minimum coverage threshold (default 1) for every reported site

## Limitations

- Requires Bismark output containing a header, 'XM' methylation strings, and 'XG' genome designations; other aligners' outputs are not supported
- All methylation calls are made with respect to the forward strand, so reverse-strand calls appear on guanine bases
- Extraction speed, while faster than Bismark's extractor (9,378 vs 22,514 sec on IMR90 replicate 1a), still scales with input size

## Evidence

- [methods] The DMRfinder script `extract_CpG_data.py` converts the output from Bismark's aligner into a table of methylated/unmethylated counts at each CpG site: "The DMRfinder script `extract_CpG_data.py` converts the output from Bismark's aligner into a table of methylated/unmethylated counts at each CpG site"
- [readme] SAM input requirements: "SAM alignment file produced by Bismark (must have a header, 'XM' methylation strings, and 'XG' genome designations; can use '-' for stdin)"
- [readme] Merging counts across strands: "The script also merges methylation counts, in that it sums counts for both the cytosine and the guanine of each CpG site"
- [results] Novel CpG site from variant rs11348696: "a known variant (rs11348696, dbSNP build 150) that results in the creation of a novel CpG site"
- [readme] Forward-strand calls: "All methylation calls are made with respect to the forward strand, so for reads that align to the reverse strand of the genome, methylation calls appear on the guanine bases."
