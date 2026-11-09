---
name: methylation-count-table-validation
description: Use when reconstructing or testing the methylation-count extraction step
  of a MethylC-seq pipeline and confirming that output tables match a reference implementation,
  including CpG sites absent from the reference genome.
when_to_use_negative:
- Input alignments were not produced by Bismark (the 'XM'/'XG' tags and in silico
  bisulfite SAM format are required)
- The goal is downstream DMR testing or region clustering (combine_CpG_sites.py /
  findDMRs.r), not per-site count extraction
- You only need methylation counts at reference-genome CpG sites and want to exclude
  variant-created novel CpG sites (that is Bismark's own behavior, not DMRfinder's)
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: DMRfinder
  role: Provides extract_CpG_data.py, the reference script whose methylation-count
    extraction behavior and novel-CpG inclusion are being validated
  repo: https://github.com/jsh58/DMRfinder
- name: Bismark
  role: Produces the SAM alignments (with 'XM' methylation strings and 'XG' genome
    designations) that are the input to the extraction step
  repo: https://github.com/FelixKrueger/Bismark
- name: Python
  role: Language of extract_CpG_data.py; the reimplementation under test (tested with
    Python 2.7.12 and 3.5.2)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:b98e3825042719d8d9ae4e4f18b30d1b9a0622759cb7ee67c5768955ea970997
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsh58/DMRfinder
  tools:
  - Bismark
  - DMRfinder
  - Python
  - extract_CpG_data.py
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# methylation-count-table-validation

## Summary

Validates a reimplementation of DMRfinder's extract_CpG_data.py step, which converts Bismark SAM alignments into a merged, sorted table of methylated/unmethylated counts at each CpG site, explicitly including novel CpG sites created by variants (e.g. rs11348696) that Bismark omits.

## When to use

Use when reconstructing or testing the methylation-count extraction step of a MethylC-seq pipeline and confirming that output tables match a reference implementation, including CpG sites absent from the reference genome.

## When NOT to use

- Input alignments were not produced by Bismark (the 'XM'/'XG' tags and in silico bisulfite SAM format are required)
- The goal is downstream DMR testing or region clustering (combine_CpG_sites.py / findDMRs.r), not per-site count extraction
- You only need methylation counts at reference-genome CpG sites and want to exclude variant-created novel CpG sites (that is Bismark's own behavior, not DMRfinder's)

## Inputs

- Bismark SAM alignment file (with header, 'XM' methylation strings, and 'XG' genome designations), via file or stdin ('-'), e.g. piped from 'samtools view -h C1_bismark_bt2.bam'

## Outputs

- Tab-delimited six-column CpG count table (chrom, chromStart, chromEnd, percent methylation, methylated counts, unmethylated counts), merged and sorted by SAM-header chromosome order then 1-based position
- Optional ordered-chromosome list via the -e option

## How to apply

Read a Bismark-produced SAM file (must contain a header, 'XM' methylation strings, and 'XG' genome designations), accepting it via stdin as in 'samtools view -h ... | python extract_CpG_data.py -i - -o out.cov'. Parse each alignment's 'XM' string, labeling reference cytosines sequenced as thymines as unmethylated and those remaining as cytosines as methylated, with all calls made on the forward strand (reverse-strand reads carry calls on the guanine bases). Merge counts by summing the cytosine and guanine tallies of each CpG site, and include novel CpG sites created by natural variants, which Bismark excludes. Apply the minimum-coverage filter (-m, default 1) and emit a six-column tab-delimited table (chrom, chromStart, chromEnd, percent methylation, methylated counts, unmethylated counts), sorted by chromosome order from the SAM header and then by 1-based position. Validation is judged by diffing against the reference implementation: on a small sample input the count table must match the reference rows, columns, and count values exactly.

## Related tools

- **DMRfinder** (Provides extract_CpG_data.py, the reference script whose methylation-count extraction behavior and novel-CpG inclusion are being validated) — https://github.com/jsh58/DMRfinder
- **Bismark** (Produces the SAM alignments (with 'XM' methylation strings and 'XG' genome designations) that are the input to the extraction step) — https://github.com/FelixKrueger/Bismark
- **Python** (Language of extract_CpG_data.py; the reimplementation under test (tested with Python 2.7.12 and 3.5.2))

## Examples

```
samtools view -h C1_bismark_bt2.bam | python extract_CpG_data.py -i - -o C1.cov -v
```

## Evaluation signals

- On a small sample SAM input, the produced count table must match the reference implementation's output: identical rows, columns, and count values
- Counts for both the cytosine and the guanine of each CpG site are summed (merged) into a single row
- A CpG site created by a natural variant (e.g. rs11348696, dbSNP build 150) present in the reads but absent from the reference genome appears in the output
- Rows are sorted by chromosome order taken from the SAM header, then by 1-based cytosine position
- Sites with fewer than -m methylation counts (default 1) are filtered out of the output

## Limitations

- The SAM input must contain a header, 'XM' methylation strings, and 'XG' genome designations; older or non-Bismark SAM files are not parseable
- Unlike Bismark, DMRfinder includes novel CpG sites created by variants, so outputs will legitimately differ from Bismark's own methylation extractor
- DMRfinder itself does not model sets of regions, quantitative traits, or batch effects; this skill covers only the per-site count-extraction step

## Evidence

- [methods] The DMRfinder script `extract_CpG_data.py` converts the output from Bismark's aligner into a table of methylated/unmethylated counts at each CpG site: "The DMRfinder script `extract_CpG_data.py` converts the output from Bismark's aligner into a table of methylated/unmethylated counts at each CpG site"
- [results] Novel CpG sites are included, unlike Bismark: "a known variant (rs11348696, dbSNP build 150) that results in the creation of a novel CpG site"
- [readme] The script also merges methylation counts, in that it sums counts for both the cytosine and the guanine of each CpG site: "The script also merges methylation counts, in that it sums counts for both the cytosine and the guanine of each CpG site"
- [readme] SAM input requirements stated in README usage: "SAM alignment file produced by Bismark (must have
                    a header, 'XM' methylation strings, and 'XG'
                    genome designations; can use '-' for stdin)"
- [readme] All methylation calls are made with respect to the forward strand: "All methylation calls are made with respect to the forward strand, so for reads that align to the reverse strand of the genome, methylation calls appear on the guanine bases."
