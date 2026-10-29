---
name: methylc-seq-alignment-processing
description: Use when raw FASTQ or Bismark BAM/SAM alignments from a bisulfite-sequencing
  run (e.g. IMR90/H1 MethylC-seq) must be converted to per-CpG methylated/unmethylated
  count tables or region-linked methylation strings.
when_to_use_negative:
- Input is a table of CpG counts already extracted (use combine_CpG_sites.py for clustering
  instead)
- The goal is differential testing of regions rather than alignment/count extraction
  (use findDMRs.r)
- The analysis requires modeling sets of regions, quantitative traits, or batch effects,
  which DMRfinder does not support
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: Bismark
  role: Aligns bisulfite-converted reads via in silico conversion and Bowtie2, making
    methylation calls with XM/XG tags in CpG/CHG/CHH contexts
  repo: https://github.com/FelixKrueger/Bismark
- name: extract_CpG_data.py
  role: DMRfinder script converting Bismark SAM output into merged, sorted methylated/unmethylated
    CpG count tables, optionally collecting linked methylation for BED regions via
    -n
  repo: https://github.com/jsh58/DMRfinder
- name: samtools
  role: Converts Bismark BAM to headered SAM (`samtools view -h`) and pipes it to
    extract_CpG_data.py via stdin
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:589830e187e850eb4f2a0b10ae3053e5f8e6fcbd21d9bc0c31a8d821c7781099
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
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# methylc-seq-alignment-processing

## Summary

Align MethylC-seq reads with Bismark (in silico bisulfite conversion, XM/XG tags) and convert alignments into merged, sorted per-CpG methylation count tables with DMRfinder's extract_CpG_data.py, optionally collecting spatially linked read-level methylation for BED-defined regions.

## When to use

Use when raw FASTQ or Bismark BAM/SAM alignments from a bisulfite-sequencing run (e.g. IMR90/H1 MethylC-seq) must be converted to per-CpG methylated/unmethylated count tables or region-linked methylation strings.

## When NOT to use

- Input is a table of CpG counts already extracted (use combine_CpG_sites.py for clustering instead)
- The goal is differential testing of regions rather than alignment/count extraction (use findDMRs.r)
- The analysis requires modeling sets of regions, quantitative traits, or batch effects, which DMRfinder does not support

## Inputs

- Single-end or paired-end MethylC-seq FASTQ files (e.g. C1.fastq, C2.fastq)
- Bismark BAM/SAM alignment files with XM methylation strings and XG genome designations
- Tab-delimited BED file of regions of interest (chromosome, 1-based start, end, unique region name) for -n linked-methylation mode
- Bismark-prepared reference genome directory

## Outputs

- Merged, sorted table of methylated/unmethylated counts at each CpG site (.cov), with 1-based cytosine position; optional strand column via -s
- Ordered chromosome list file (-e output)
- <file>_linked.txt with per-region read-level methylation strings encoding 0 (unmethylated CpG), 1 (methylated), and - (no data) at each CpG column

## How to apply

Align reads to a reference genome with Bismark, which performs in silico bisulfite conversion and labels reference cytosines sequenced as thymines unmethylated and those remaining as cytosines methylated, using XM methylation strings and XG genome designations; note all calls are made relative to the forward strand. Convert BAM to headered SAM via `samtools view -h` because extract_CpG_data.py requires a headered SAM with XM and XG tags, and pipe it directly into `python extract_CpG_data.py -i - -o <sample>.cov -v`. The script merges counts by summing cytosine and guanine counts at each CpG and sorts output by chromosome (header order) and 1-based cytosine position. For methylation linkage analysis, prepare a tab-delimited BED file (chromosome, 1-based start, end, unique region name) and rerun extract_CpG_data.py with `-n <BED>` to collect read-level methylation patterns for reads covering at least one CpG in each region. Use `-r` to set minimum coverage per CpG (extract step default 1) and `-b` as a memory-saving option for coordinate-sorted SAM. Prefer Bismark 0.15.0 over 0.16.0 due to an ambiguous-alignment bug, and avoid Bismark's deduplication script, which discards reads regardless of methylation information.

## Related tools

- **Bismark** (Aligns bisulfite-converted reads via in silico conversion and Bowtie2, making methylation calls with XM/XG tags in CpG/CHG/CHH contexts) — https://github.com/FelixKrueger/Bismark
- **extract_CpG_data.py** (DMRfinder script converting Bismark SAM output into merged, sorted methylated/unmethylated CpG count tables, optionally collecting linked methylation for BED regions via -n) — https://github.com/jsh58/DMRfinder
- **samtools** (Converts Bismark BAM to headered SAM (`samtools view -h`) and pipes it to extract_CpG_data.py via stdin)

## Examples

```
samtools view -h C1_bismark_bt2.bam | python extract_CpG_data.py -i - -o C1.cov -v
```

## Evaluation signals

- The .cov output is sorted by chromosome (in SAM-header reference-name order) and 1-based cytosine position, with cytosine and guanine counts summed per CpG
- Running with -n BED regenerates a <file>_linked.txt whose Region/Sites/read lines reproduce the per-region 0/1/- encoding reported in the methylation linkage additional file
- Alignment logs show single optimal alignments retained and methylation calls labeled by CpG/CHG/CHH context via XM strings
- Per-sample .cov files are produced for every sample in the run (e.g. C1.cov, C2.cov) with coverage no lower than the -m/-r minimum used

## Limitations

- Bismark version 0.16.0 contains an ambiguous-read-alignment bug; version 0.15.0 is recommended until fixed
- Bismark's deduplication script keeps the first read at each position regardless of sequence or methylation information, so it is not recommended for PCR duplicate removal
- extract_CpG_data.py requires a headered SAM with XM methylation strings and XG genome designations; BAM must be converted with `samtools view -h` first

## Evidence

- [readme] Bismark performs an in silico bisulfite conversion of both the reads and the reference genome, and then uses Bowtie2 to align the reads to the reference.: [evidence span withheld — anchor status: not_found]
- [methods] The DMRfinder script `extract_CpG_data.py` converts the output from Bismark's aligner into a table of methylated/unmethylated counts at each CpG site: "The DMRfinder script `extract_CpG_data.py` converts the output from Bismark's aligner into a table of methylated/unmethylated counts at each CpG site"
- [readme] The script also merges methylation counts, in that it sums counts for both the cytosine and the guanine of each CpG site: "The script also merges methylation counts, in that it sums counts for both the cytosine and the guanine of each CpG site"
- [readme] we recommend [version 0.15.0](https://github.com/FelixKrueger/Bismark/releases/tag/v0.15.0).: "we recommend [version 0.15.0](https://github.com/FelixKrueger/Bismark/releases/tag/v0.15.0)."
- [supplementary] The methylation linkage analysis can determine if methylation differences occur in a nonrandom pattern, which may reflect an alteration in the cellular populations being studied: [evidence span withheld — anchor status: not_found]
