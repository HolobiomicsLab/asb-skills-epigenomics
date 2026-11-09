---
name: genomic-region-bed-interval-handling
description: Use when you need read-level methylation linkage (phase) information
  within defined genomic intervals from MethylC-seq/Bismark alignments, e.g. to test
  whether methylation differences occur in a nonrandom pattern.
when_to_use_negative:
- You only need aggregate per-CpG or per-region methylation counts for DMR calling
  — use extract_CpG_data.py -i plus combine_CpG_sites.py and findDMRs.r instead of
  the -n linked-data mode
- The input alignment lacks a header, XM methylation strings, or XG genome designations
  (e.g. a non-Bismark SAM) — extract_CpG_data.py will not parse it
- You need to model sets of regions, quantitative traits, or batch effects — the article
  notes DMRfinder does not support these
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: extract_CpG_data.py
  role: DMRfinder script that, with the -n BED-file option, collects spatially-linked
    methylation data for reads covering at least one CpG in each region from Bismark
    SAM input
  repo: https://github.com/jsh58/DMRfinder
- name: Bismark
  role: Performs in silico bisulfite conversion and alignment, producing the SAM/BAM
    with XM/XG tags required for methylation extraction
  repo: https://github.com/FelixKrueger/Bismark
- name: samtools
  role: Converts BAM alignments to headered SAM via `samtools view -h` for piping
    into extract_CpG_data.py
- name: DMRfinder
  role: Package providing extract_CpG_data.py and the methylation linkage analysis
    workflow
  repo: https://github.com/jsh58/DMRfinder
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:f872614a6a20a113f47b6885c07f6d5708d35f6e62f3d43adf78c055c9351952
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsh58/DMRfinder
  tools:
  - Bismark
  - DMRfinder
  - Python
  - extract_CpG_data.py
  - samtools
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# genomic-region-bed-interval-handling

## Summary

Prepare tab-delimited BED files of genomic regions of interest and use DMRfinder's extract_CpG_data.py -n option to collect read-level, spatially-linked methylation patterns within each region from Bismark SAM alignments.

## When to use

Use when you need read-level methylation linkage (phase) information within defined genomic intervals from MethylC-seq/Bismark alignments, e.g. to test whether methylation differences occur in a nonrandom pattern.

## When NOT to use

- You only need aggregate per-CpG or per-region methylation counts for DMR calling — use extract_CpG_data.py -i plus combine_CpG_sites.py and findDMRs.r instead of the -n linked-data mode
- The input alignment lacks a header, XM methylation strings, or XG genome designations (e.g. a non-Bismark SAM) — extract_CpG_data.py will not parse it
- You need to model sets of regions, quantitative traits, or batch effects — the article notes DMRfinder does not support these

## Inputs

- Headered SAM alignment file from Bismark (with XM methylation strings and XG genome designations), optionally piped from `samtools view -h` on a BAM
- Tab-delimited BED file of genomic regions of interest: chromosome, 1-based start, end, unique region name

## Outputs

- <file>_linked.txt file of linked methylation data per BED region: Region and Sites header lines, then one line per read with 0 (unmethylated CpG), 1 (methylated CpG), or - (no data) at each CpG column
- Merged, sorted table of methylated/unmethylated counts per CpG site (when run with -i -o without -n)

## How to apply

First align MethylC-seq reads (e.g. IMR90 and H1 samples from Lister et al.) with Bismark, which labels methylation via XM methylation strings and XG genome designations. Convert BAM to headered SAM with `samtools view -h`, since extract_CpG_data.py requires a headered SAM input. Prepare a BED file in tab-delimited form with four columns: chromosome, 1-based start coordinate, end coordinate, and a unique region name. Run `python extract_CpG_data.py` first with `-i` on the SAM file, then re-run it with the `-n <file>` option pointing to the BED file so the script collects spatially-linked methylation data for reads covering at least one CpG in each region. The output `<file>_linked.txt` encodes, per BED region, 0 for unmethylated CpG, 1 for methylated, and - (no data) at each CpG column for each read; this linkage analysis can determine if methylation differences occur in a nonrandom pattern, which may reflect an alteration in the cellular populations being studied.

## Related tools

- **extract_CpG_data.py** (DMRfinder script that, with the -n BED-file option, collects spatially-linked methylation data for reads covering at least one CpG in each region from Bismark SAM input) — https://github.com/jsh58/DMRfinder
- **Bismark** (Performs in silico bisulfite conversion and alignment, producing the SAM/BAM with XM/XG tags required for methylation extraction) — https://github.com/FelixKrueger/Bismark
- **samtools** (Converts BAM alignments to headered SAM via `samtools view -h` for piping into extract_CpG_data.py)
- **DMRfinder** (Package providing extract_CpG_data.py and the methylation linkage analysis workflow) — https://github.com/jsh58/DMRfinder

## Examples

```
samtools view -h sample_bismark_bt2.bam | python extract_CpG_data.py -i - -o sample.cov -v; python extract_CpG_data.py -i sample_bismark_bt2.sam -n regions.bed -o sample_linked.txt
```

## Evaluation signals

- The regenerated <file>_linked.txt output reproduces the per-region read-level linkage results and format reported in the 'Methylation linkage analysis' additional file
- Each region block contains a Region line and a Sites line (CpG coordinates), followed by one line per read
- Read-level methylation characters are restricted to 0 (unmethylated CpG), 1 (methylated CpG), and - (no data at that CpG)
- The linked output only covers reads that span at least one CpG within each BED region listed in the -n file
- The underlying per-CpG counts table (from the -i run) is merged, summing cytosine and guanine counts per CpG site, and sorted by chromosome and 1-based cytosine position

## Limitations

- extract_CpG_data.py requires SAM input with a header, 'XM' methylation strings, and 'XG' genome designations — non-Bismark alignments cannot be processed
- The linkage output provides per-read methylation strings but no statistical testing; DMRfinder does not consider sets of regions or quantitative traits and cannot account for batch effects
- Bed-file regions must be supplied with 1-based coordinates and a unique region name in tab-delimited form; Bismark version matters — a bug affecting ambiguous read alignments was introduced in Bismark 0.16.0 and version 0.15.0 was recommended at the time

## Evidence

- [readme] -n <file>     BED file listing regions for which to collect linked methylation data: "-n <file>     BED file listing regions for which to collect
                    linked methylation data"
- [supplementary] The methylation linkage analysis can determine if methylation differences occur in a nonrandom pattern: [evidence span withheld — anchor status: not_found]
- [readme] -i <input>    SAM alignment file produced by Bismark (must have a header, 'XM' methylation strings, and 'XG' genome designations; can use '-' for stdin): "-i <input>    SAM alignment file produced by Bismark (must have
                    a header, 'XM' methylation strings, and 'XG'
                    genome designations; can use '-' for stdin)"
- [readme] Bismark performs an in silico bisulfite conversion of both the reads and the reference genome, and then uses Bowtie2 to align the reads to the reference: "Bismark [1] performs an in silico bisulfite conversion of both the reads and the reference genome, and then uses Bowtie2 [[3](#ref3)] to align the reads to the reference."
