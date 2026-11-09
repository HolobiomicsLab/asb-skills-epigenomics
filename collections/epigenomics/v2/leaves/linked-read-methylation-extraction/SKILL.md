---
name: linked-read-methylation-extraction
description: Use when you need read-level methylation linkage (0/1/- encoding per
  CpG column) across spatially linked CpGs in defined BED regions from Bismark SAM
  alignments, e.g. to test whether methylation changes reflect altered cellular populations.
when_to_use_negative:
- Input alignments were produced by a non-bisulfite aligner and lack Bismark XM/XG
  tags or a SAM header — extract_CpG_data.py requires these for methylation calling
- The goal is group-level differential methylation testing rather than read-level
  linkage — use combine_CpG_sites.py and findDMRs.r downstream instead
- Input is raw FASTQ without alignment — Bismark alignment must be performed first,
  and Bismark 0.16.0 should be avoided in favor of 0.15.0 due to an ambiguous-alignment
  bug
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0622
tools:
- name: extract_CpG_data.py
  role: DMRfinder script that converts Bismark aligner output into a table of methylated/unmethylated
    counts at each CpG site; with -n it collects spatially-linked methylation data
    for reads covering CpGs in BED regions
  repo: https://github.com/jsh58/DMRfinder
- name: Bismark
  role: Aligns MethylC-seq reads via in silico bisulfite conversion and Bowtie2, making
    methylation calls via XM/XG tags
  repo: https://github.com/FelixKrueger/Bismark
- name: samtools
  role: Converts Bismark BAM alignments to headered SAM (`samtools view -h`) for input
    to extract_CpG_data.py
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:9ea36fffecc1b67b13e9047c3da277eeebb1a134935ccdcd5ef62ce8749d80fc
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# linked-read-methylation-extraction

## Summary

Extract per-read, spatially-linked methylation states at CpG sites within user-defined genomic regions from Bismark-aligned MethylC-seq data using DMRfinder's extract_CpG_data.py with the -n BED option, enabling assessment of whether methylation differences occur in a nonrandom pattern across linked CpGs.

## When to use

Use when you need read-level methylation linkage (0/1/- encoding per CpG column) across spatially linked CpGs in defined BED regions from Bismark SAM alignments, e.g. to test whether methylation changes reflect altered cellular populations.

## When NOT to use

- Input alignments were produced by a non-bisulfite aligner and lack Bismark XM/XG tags or a SAM header — extract_CpG_data.py requires these for methylation calling
- The goal is group-level differential methylation testing rather than read-level linkage — use combine_CpG_sites.py and findDMRs.r downstream instead
- Input is raw FASTQ without alignment — Bismark alignment must be performed first, and Bismark 0.16.0 should be avoided in favor of 0.15.0 due to an ambiguous-alignment bug

## Inputs

- Bismark SAM alignment file with header, XM methylation strings and XG genome designations (pipe BAM through `samtools view -h`)
- Tab-delimited BED file of genomic regions of interest (chromosome, 1-based start, end, unique region name)
- Optional: paired-end or single-end FASTQ reads from MethylC-seq (bisulfite-treated) samples for upstream Bismark alignment

## Outputs

- Merged, sorted table of methylated/unmethylated counts at each CpG site (cytosine and guanine counts summed), e.g. sample.cov
- Linked methylation file (<file>_linked.txt) with Region/Sites/read-lines encoding 0 (unmethylated), 1 (methylated), '-' (no data) per CpG column per read for each BED region

## How to apply

Align MethylC-seq reads (e.g. Lister et al. IMR90 and H1 samples) with Bismark, which performs in silico bisulfite conversion and labels methylation via XM methylation strings and XG genome designations, with all calls made on the forward strand (reverse-strand reads call methylation on guanines). Convert BAM to a headered SAM with `samtools view -h` because extract_CpG_data.py requires a SAM with header, XM and XG tags. First run extract_CpG_data.py with -i on the SAM to produce a merged, sorted table of methylated/unmethylated counts per CpG, summing cytosine and guanine counts. Prepare a tab-delimited BED file (chromosome, 1-based start, end, unique region name), then re-run extract_CpG_data.py with -n pointing to the BED file so the script collects linked methylation data for reads covering at least one CpG in each region. Confirm the regenerated <file>_linked.txt output encodes, per BED region, 0 for unmethylated CpG, 1 for methylated, and '-' for no data at each CpG column for each read.

## Related tools

- **extract_CpG_data.py** (DMRfinder script that converts Bismark aligner output into a table of methylated/unmethylated counts at each CpG site; with -n it collects spatially-linked methylation data for reads covering CpGs in BED regions) — https://github.com/jsh58/DMRfinder
- **Bismark** (Aligns MethylC-seq reads via in silico bisulfite conversion and Bowtie2, making methylation calls via XM/XG tags) — https://github.com/FelixKrueger/Bismark
- **samtools** (Converts Bismark BAM alignments to headered SAM (`samtools view -h`) for input to extract_CpG_data.py)

## Examples

```
samtools view -h sample_bismark_bt2.bam | python extract_CpG_data.py -i - -o sample.cov -v -n regions.bed
```

## Evaluation signals

- Regenerated <file>_linked.txt reproduces the per-region read-level linkage results and format (Region/Sites/read-lines with 0/1/- encoding) reported in the methylation linkage analysis additional file
- Every read line in the linked output covers at least one CpG in each BED region
- CpG counts in the .cov output are merged (cytosine and guanine counts summed) and sorted by chromosome (per SAM header order) and 1-based cytosine position
- Methylation calls are reported with respect to the forward strand, so reverse-strand reads contribute counts at guanine positions
- Each BED region appears in the linked output under its unique region name with the expected number of CpG columns matching the region's CpG content

## Limitations

- DMRfinder does not allow modeling of sets of regions or quantitative traits — it identifies individual regions that are differentially methylated
- DMRfinder cannot account for batch effects
- extract_CpG_data.py requires a headered SAM with XM and XG tags; BAM files must be piped through `samtools view -h` first
- Bismark version 0.16.0 introduced a bug related to ambiguous read alignments; version 0.15.0 is recommended until fixed

## Evidence

- [methods] The DMRfinder script `extract_CpG_data.py` converts the output from Bismark's aligner into a table of methylated/unmethylated counts at each CpG site: "The DMRfinder script `extract_CpG_data.py` converts the output from Bismark's aligner into a table of methylated/unmethylated counts at each CpG site"
- [readme] Linked methylation data option: "-n <file>     BED file listing regions for which to collect
                    linked methylation data"
- [readme] SAM input requirements: "-i <input>    SAM alignment file produced by Bismark (must have
                    a header, 'XM' methylation strings, and 'XG'
                    genome designations; can use '-' for stdin)"
- [readme] Merged counts and forward-strand calling: "The script also merges methylation counts, in that it sums counts for both the cytosine and the guanine of each CpG site"
- [readme] Forward-strand methylation calls with Bismark: "All methylation calls are made with respect to the forward strand, so for reads that align to the reverse strand of the genome, methylation calls appear on the guanine bases."
- [supplementary] Nonrandom methylation linkage finding: [evidence span withheld — anchor status: not_found]
