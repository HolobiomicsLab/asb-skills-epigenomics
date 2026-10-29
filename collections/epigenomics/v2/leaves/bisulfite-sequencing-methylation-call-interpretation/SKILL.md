---
name: bisulfite-sequencing-methylation-call-interpretation
description: Use when working with Bismark-aligned MethylC-seq reads (e.g., IMR90/H1
  data) and needing to interpret per-CpG methylation calls, extract methylated/unmethylated
  counts, or examine spatially-linked methylation patterns across reads.
when_to_use_negative:
- When the goal is region-level differential methylation testing rather than read-level
  spatially-linked methylation interpretation — use the combine_CpG_sites.py and findDMRs.r
  pipeline instead
- When alignment output lacks Bismark XM/XG tags (e.g., alignments from other bisulfite
  aligners), since extract_CpG_data.py requires these designations
- When modeling sets of regions, quantitative traits, or batch effects — DMRfinder
  cannot account for these
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: Bismark
  role: Aligns bisulfite-converted reads with in silico conversion and makes methylation
    calls via XM/XG tags in CpG/CHG/CHH contexts
  repo: FelixKrueger/Bismark
- name: extract_CpG_data.py
  role: Converts Bismark alignment output into merged, sorted methylated/unmethylated
    CpG count tables and collects spatially-linked methylation data for BED regions
  repo: https://github.com/jsh58/DMRfinder
- name: samtools
  role: Converts Bismark BAM output to headered SAM (samtools view -h) required as
    extract_CpG_data.py input
- name: DMRfinder
  role: Package providing extract_CpG_data.py and the methylation linkage analysis
    framework
  repo: https://github.com/jsh58/DMRfinder
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:88ba271c0a4c3024c99edab319025e138bfaba1751284fc8cf9cd48d6585cd35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FelixKrueger/Bismark
  tools:
  - Bismark
  - DMRfinder
  - Python
  - extract_CpG_data.py
  - samtools
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# bisulfite-sequencing-methylation-call-interpretation

## Summary

Interprets Bismark methylation calls from MethylC-seq alignments — reading XM methylation strings and XG strand designations in SAM/BAM output — so that per-read, per-CpG methylation states (methylated/unmethylated/no-data) can be correctly extracted, merged across strands, and used for region-level linkage analysis.

## When to use

Use when working with Bismark-aligned MethylC-seq reads (e.g., IMR90/H1 data) and needing to interpret per-CpG methylation calls, extract methylated/unmethylated counts, or examine spatially-linked methylation patterns across reads.

## When NOT to use

- When the goal is region-level differential methylation testing rather than read-level spatially-linked methylation interpretation — use the combine_CpG_sites.py and findDMRs.r pipeline instead
- When alignment output lacks Bismark XM/XG tags (e.g., alignments from other bisulfite aligners), since extract_CpG_data.py requires these designations
- When modeling sets of regions, quantitative traits, or batch effects — DMRfinder cannot account for these

## Inputs

- Bismark SAM/BAM alignment files with header, XM methylation strings, and XG genome designations (e.g., from Lister et al. IMR90 and H1 MethylC-seq reads)
- Tab-delimited BED file of genomic regions of interest (chromosome, 1-based start, end, unique region name) for linked methylation collection

## Outputs

- Merged, sorted table of methylated/unmethylated counts at each CpG site (cytosine and guanine counts summed) from extract_CpG_data.py
- <file>_linked.txt file encoding per-region, per-read methylation linkage (0 = unmethylated CpG, 1 = methylated, - = no data)

## How to apply

Align MethylC-seq reads with Bismark, which performs in silico bisulfite conversion of reads and reference and labels reference cytosines sequenced as thymines as unmethylated and those remaining as cytosines as methylated; all calls are made with respect to the forward strand, so reverse-strand reads record methylation on the guanine bases. The alignments must carry a header, 'XM' methylation strings, and 'XG' genome designations for downstream extraction. Convert BAM to SAM (samtools view -h) and pipe into extract_CpG_data.py, which merges counts by summing cytosine and guanine counts of each CpG site and sorts by chromosome and 1-based cytosine position. For methylation linkage analysis, supply a tab-delimited BED file of regions (chromosome, 1-based start, end, unique name) via the -n option, producing a <file>_linked.txt output where each read covering at least one CpG in the region encodes 0 = unmethylated, 1 = methylated, and - = no data at each CpG column. This read-level linkage encoding lets you determine whether methylation differences occur in a nonrandom pattern that may reflect an alteration in the cellular populations being studied.

## Related tools

- **Bismark** (Aligns bisulfite-converted reads with in silico conversion and makes methylation calls via XM/XG tags in CpG/CHG/CHH contexts) — FelixKrueger/Bismark
- **extract_CpG_data.py** (Converts Bismark alignment output into merged, sorted methylated/unmethylated CpG count tables and collects spatially-linked methylation data for BED regions) — https://github.com/jsh58/DMRfinder
- **samtools** (Converts Bismark BAM output to headered SAM (samtools view -h) required as extract_CpG_data.py input)
- **DMRfinder** (Package providing extract_CpG_data.py and the methylation linkage analysis framework) — https://github.com/jsh58/DMRfinder

## Examples

```
samtools view -h sample_bismark_bt2.bam | python extract_CpG_data.py -i - -o sample.cov -v -n regions.bed
```

## Evaluation signals

- The regenerated <file>_linked.txt reproduces the per-region read-level linkage results with Region/Sites/read-lines and 0/1/- encoding matching the Additional file 2 methylation linkage analysis
- extract_CpG_data.py output is sorted by chromosome (in reference header order) and by 1-based cytosine position, with cytosine and guanine counts summed per CpG site
- Alignment input retains a SAM header with XM methylation strings and XG genome designations before extraction
- For reverse-strand-aligned reads, methylation calls appear on the guanine bases of CpGs, consistent with Bismark's forward-strand call convention

## Limitations

- Bismark version 0.16.0 contains a bug related to ambiguous read alignments; version 0.15.0 is recommended until the fix is merged
- Bismark's deduplication script simply keeps the first read at a given position regardless of sequence or methylation information and is not recommended
- DMRfinder identifies individual differentially methylated regions but does not consider sets of regions or quantitative traits, and cannot account for batch effects

## Evidence

- [readme] Bismark performs an in silico bisulfite conversion of both the reads and the reference genome: "Bismark performs an in silico bisulfite conversion of both the reads and the reference genome, and then uses Bowtie2 [[3](#ref3)] to align the reads"
- [readme] reference cytosines that were sequenced as thymines are labeled unmethylated, and those that remained as cytosines are labeled methylated: "reference cytosines that were sequenced as thymines are labeled unmethylated, and those that remained as cytosines are labeled methylated."
- [readme] All methylation calls are made with respect to the forward strand, so for reads that align to the reverse strand of the genome, methylation calls appear on the guanine bases: "All methylation calls are made with respect to the forward strand, so for reads that align to the reverse strand of the genome, methylation calls appear on the guanine bases."
- [readme] it sums counts for both the cytosine and the guanine of each CpG site: "The script also merges methylation counts, in that it sums counts for both the cytosine and the guanine of each CpG site"
- [supplementary] The methylation linkage analysis can determine if methylation differences occur in a nonrandom pattern, which may reflect an alteration in the cellular populations being studied: [evidence span withheld — anchor status: not_found]
- [supplementary] 0 for unmethylated CpG, 1 for methylated, and - for no data at each CpG column per read: [evidence span withheld — anchor status: not_found]
