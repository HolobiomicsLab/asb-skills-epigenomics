---
name: differential-methylation-region-testing
description: Use when Bismark-aligned MethylC-seq (bisulfite) data in SAM/BAM format
  needs post-alignment processing to identify differentially methylated regions (DMRs)
  between two or more sample groups, e.g., IMR90 vs H1.
when_to_use_negative:
- The analysis needs modeling of sets of regions or quantitative traits — DMRfinder
  only identifies individual differentially methylated regions
- The data contain batch effects that must be accounted for — DMRfinder cannot correct
  for batch effects
- Input is raw methylation calls from a non-Bismark aligner lacking 'XM' methylation
  strings and 'XG' genome designations in the SAM output
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: DMRfinder
  role: 'Core pipeline: extract_CpG_data.py extracts counts, combine_CpG_sites.py
    clusters CpGs into regions, findDMRs.r performs the DMR tests'
  repo: https://github.com/jsh58/DMRfinder
- name: Bismark
  role: Aligns bisulfite-treated reads to the reference genome and produces the SAM/BAM
    input with methylation strings; recommended version 0.15.0
  repo: https://github.com/FelixKrueger/Bismark
- name: Bowtie2
  role: Underlying aligner used by Bismark to map the reads to the in-silico bisulfite-converted
    reference
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: DSS
  role: Bioconductor package whose statistical framework (beta-binomial hierarchical
    modeling and Wald tests) is used by findDMRs.r
  repo: http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- name: Samtools
  role: Converts Bismark BAM to SAM via `samtools view -h` to stream into extract_CpG_data.py
  repo: http://www.htslib.org/
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:38a7e4f0111b4eae817908c440a33b9ea0e19011c20b624dc66366f565c2cc7e
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
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# differential-methylation-region-testing

## Summary

A three-script DMRfinder pipeline that extracts per-CpG methylation counts from Bismark SAM/BAM alignments, clusters CpG sites into genomic regions via modified single-linkage clustering, and tests regions for differential methylation using DSS beta-binomial hierarchical modeling and Wald tests.

## When to use

Use when Bismark-aligned MethylC-seq (bisulfite) data in SAM/BAM format needs post-alignment processing to identify differentially methylated regions (DMRs) between two or more sample groups, e.g., IMR90 vs H1.

## When NOT to use

- The analysis needs modeling of sets of regions or quantitative traits — DMRfinder only identifies individual differentially methylated regions
- The data contain batch effects that must be accounted for — DMRfinder cannot correct for batch effects
- Input is raw methylation calls from a non-Bismark aligner lacking 'XM' methylation strings and 'XG' genome designations in the SAM output

## Inputs

- Bismark SAM/BAM alignment files (with header, 'XM' methylation strings, and 'XG' genome designations), e.g. Lister et al. IMR90 and H1 MethylC-seq reads
- Bismark reference genome prepared with bismark_genome_preparation
- FASTQ sequence files for samples in each group

## Outputs

- Per-sample merged, sorted CpG methylation count tables (*.cov)
- combined.csv: clustered CpG regions with summed methylation counts per sample
- results.csv: differentially methylated regions with Wald test statistics

## How to apply

Run `samtools view -h` on each Bismark BAM and pipe into `extract_CpG_data.py -i - -o <sample>.cov` to obtain merged, sorted per-CpG methylated/unmethylated count tables (counts for the cytosine and guanine of each CpG are summed). Then run `combine_CpG_sites.py -v -o combined.csv <sample>.cov ...` with default clustering parameters (-r 3 min counts per CpG, -s 1 min samples, -d 100 max CpG spacing, -c 3 min CpGs per region, -x 500 bp max region length, -m 20 min total counts per region); clustering does not consider methylation levels, so it is not biased toward finding DMRs, and the -x 500 bp split limits the chaining effect of single-linkage clustering. Finally run `Rscript findDMRs.r -i combined.csv -o results.csv -v -n <group_names> <groupList1> <groupList2>`, which applies DSS beta-binomial hierarchical modeling with Wald tests and default significance filters -d 0.10 (minimum methylation difference) and -p 0.05 (maximum p-value). Use Bismark v0.15.0 for alignment, since a bug related to ambiguous read alignments was introduced in v0.16.0. The statistical framework is that of the Bioconductor package DSS (tested with DSS 2.14.0 and Rscript 3.3.1; Python 2.7.12/3.5.2).

## Related tools

- **DMRfinder** (Core pipeline: extract_CpG_data.py extracts counts, combine_CpG_sites.py clusters CpGs into regions, findDMRs.r performs the DMR tests) — https://github.com/jsh58/DMRfinder
- **Bismark** (Aligns bisulfite-treated reads to the reference genome and produces the SAM/BAM input with methylation strings; recommended version 0.15.0) — https://github.com/FelixKrueger/Bismark
- **Bowtie2** (Underlying aligner used by Bismark to map the reads to the in-silico bisulfite-converted reference) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **DSS** (Bioconductor package whose statistical framework (beta-binomial hierarchical modeling and Wald tests) is used by findDMRs.r) — http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- **Samtools** (Converts Bismark BAM to SAM via `samtools view -h` to stream into extract_CpG_data.py) — http://www.htslib.org/

## Examples

```
samtools view -h C1_bismark_bt2.bam | python extract_CpG_data.py -i - -o C1.cov -v; python combine_CpG_sites.py -v -o combined.csv C1.cov C2.cov E1.cov E2.cov; Rscript findDMRs.r -i combined.csv -o results.csv -v -n Control,Exptl C1,C2 E1,E2
```

## Evaluation signals

- The *.cov outputs are sorted by chromosome (per the reference sequence-name order in the SAM header) and 1-based cytosine position, with counts merged across the cytosine and guanine of each CpG
- combined.csv reflects the three-step clustering with defaults: CpG sites ≤100 bp apart, regions with ≥3 CpGs, ≤500 bp long, and ≥20 total counts, with ≥3 counts in at least 1 sample
- results.csv only contains regions passing -d 0.10 methylation difference and -p 0.05 p-value thresholds
- A same-sample replicate self-contrast should yield minimal DMRs (e.g., DMRfinder reported two, DSS zero) — a substantial number of false positives indicates misconfiguration
- Runtime should be far below alternative tools: e.g., extracting counts from IMR90 replicate 1a took 9,378 s with DMRfinder vs 22,514 s with Bismark, using 193 times less disk space

## Limitations

- DMRfinder identifies individual regions that are differentially methylated but does not consider sets of regions or quantitative traits
- DMRfinder cannot account for batch effects; these are noted as areas for potential further improvements
- Bismark v0.16.0 introduced a bug related to ambiguous read alignments, so v0.15.0 is recommended until fixed
- Bismark's deduplication script is not recommended; it keeps the first read at a position regardless of sequence or methylation information

## Evidence

- [abstract] performs a modified single-linkage clustering of methylation sites into genomic regions: "performs a modified single-linkage clustering of methylation sites into genomic regions"
- [abstract] DMRfinder tests regions with beta-binomial modeling and Wald tests: "It then compares methylation levels using beta-binomial hierarchical modeling and Wald tests."
- [readme] README quick start commands for the four-step pipeline: "$ samtools view -h C1_bismark_bt2.bam  |  python extract_CpG_data.py  -i -  -o C1.cov  -v"
- [readme] README documents DMRfinder's statistical framework: "It then performs pairwise comparisons to identify differentially methylated regions (DMRs) using the statistical framework of the Bioconductor package DSS"
- [methods] Article methods on clustering parameter defaults: "-d <int>    Max. distance between CpG sites (def. 100)"
- [methods] Article methods on findDMRs.r significance thresholds: "-d <float>    Min. methylation difference between sample groups
                    ([0-1]; def. 0.10)"
- [readme] README warning on Bismark version: "A [bug](https://github.com/FelixKrueger/Bismark/issues/56) related to ambiguous read alignments crept into version 0.16.0 of Bismark."
- [results] Article results on runtime benchmark: [evidence span withheld — anchor status: not_found]
