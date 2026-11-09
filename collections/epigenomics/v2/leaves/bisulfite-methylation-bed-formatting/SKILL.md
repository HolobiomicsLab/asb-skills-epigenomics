---
name: bisulfite-methylation-bed-formatting
description: Use when bisulfite sequencing methylation data (e.g. Bismark .cov, RRBS/MethylC-seq
  calls) must be reformatted for DXM tools such as dxm_solveMethylation or dxm_estimateFracs.
when_to_use_negative:
- Methylation values are on a 0–100 scale and have not yet been divided by 100 — DXM
  requires fractional methylation between 0 and 1
- Region names are not unique per tested region (DXM computes across all CpGs of a
  given region keyed by regionName)
- Data are allele-specific methylation or copy-number-variant regions where DXM interpretation
  requires special care
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: DXM
  role: Target tool whose BED-like input format (chr, position1, position2, regionName,
    fractionalMethylation, coverage) the conversion must satisfy
  repo: https://github.com/edwardslab-wustl/dxm
- name: bedtools
  role: Intersects methylation data with CGI/promoter bed files and supplies region
    names for the DXM input columns
- name: TrimGalore
  role: Listed as related repository for consistent quality and adapter trimming of
    bisulfite sequencing reads, with special handling for RRBS libraries, upstream
    of methylation calling
  repo: FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:6c74b1f4b3599025cf9c2335b068d04c6b71f3407233958bb6313952af73e38a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - DSS
  - DXM
  - TrimGalore
  - bedtools
  - methylPurify
  - numpy
  - python3
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# bisulfite-methylation-bed-formatting

## Summary

Converts processed bisulfite sequencing methylation data (e.g. Bismark .cov output) into the tab-delimited BED-like DXM input format (chromosome, position1, position2, regionName, fractionalMethylation, coverage), with strand collapsing and coverage filtering as recommended pre-steps.

## When to use

Use when bisulfite sequencing methylation data (e.g. Bismark .cov, RRBS/MethylC-seq calls) must be reformatted for DXM tools such as dxm_solveMethylation or dxm_estimateFracs.

## When NOT to use

- Methylation values are on a 0–100 scale and have not yet been divided by 100 — DXM requires fractional methylation between 0 and 1
- Region names are not unique per tested region (DXM computes across all CpGs of a given region keyed by regionName)
- Data are allele-specific methylation or copy-number-variant regions where DXM interpretation requires special care

## Inputs

- Bismark .cov file from bismark_methylation_extractor (chr pos1 pos2 methylation meth_cov unmeth_cov)
- CGI or promoter-region bed file (e.g. UCSC Table Browser cgi.bed, or refGene_hg19_10kb_fixed.txt.gz from the DXM example_data directory)
- Other aligner methylation outputs convertible to BED-like methylation format

## Outputs

- Tab-delimited BED-like DXM input file (dxm_in.bed) with columns chromosome, position1, position2, regionName, fractionalMethylation (0–1), integer coverage

## How to apply

Rescale methylation fractions to the 0–1 range (divide by 100 if the aligner outputs 0–100) and convert each site/region to the six-column tab-delimited format: chromosome, position1, position2, regionName, fractionalMethylation, integer coverage. Collapse data across strands and filter so every site's coverage stays below the maximum expected sequencing coverage that will be passed via the -c/-m flag in dxm_solveMethylation. Assign a unique regionName to each tested region (e.g. CGI ID, gene, enhancer) — DXM computes across all CpGs of a region keyed by that name — by downloading a CGI or refGene bed file and running bedtools intersect to project methylation data onto regions. For Bismark .cov files (chr pos1 pos2 methylation meth_cov unmeth_cov), compute coverage as meth_cov + unmeth_cov with awk, then intersect with the region bed and print the columns in DXM order (e.g. awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed).

## Related tools

- **DXM** (Target tool whose BED-like input format (chr, position1, position2, regionName, fractionalMethylation, coverage) the conversion must satisfy) — https://github.com/edwardslab-wustl/dxm
- **bedtools** (Intersects methylation data with CGI/promoter bed files and supplies region names for the DXM input columns)
- **TrimGalore** (Listed as related repository for consistent quality and adapter trimming of bisulfite sequencing reads, with special handling for RRBS libraries, upstream of methylation calling) — FelixKrueger/TrimGalore

## Examples

```
bedtools intersect -wo -a methylation.bed -b cgi.bed | awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed
```

## Evaluation signals

- Output file has exactly six tab-delimited columns in the order chromosome, position1, position2, regionName, fractionalMethylation, coverage
- All fractionalMethylation values lie between 0 (fully unmethylated) and 1 (fully methylated)
- All coverage values are integers strictly below the maximum expected sequencing coverage (-c/-m flag, e.g. 400 default, 20 or 40 for simulated mixtures)
- Every region name is unique per tested region (e.g. CGI ID, enhancer ID) and position fields are integers
- bedtools intersect output correctly selects column $9 (region name) and methylation/coverage columns from the awk projection

## Limitations

- DXM does not explicitly use chromosome or position2 fields (they are required only for BED compatibility), so relative coordinates are permissible but region names must be unique
- Data should be collapsed across strands prior to running DXM per the authors' recommendation
- Coverage must be filtered below the maximum expected sequencing coverage or -c (e.g. default 400) will needlessly increase memory usage

## Evidence

- [readme] The methylation data input file should be in a tab-delimited bed-like format: <chr> <position1> <position2> <regionName> <fractionalMethylation> <coverage>: "The methylation data input file should be in a tab-delimited bed-like format: <chr> <position1> <position2> <regionName> <fractionalMethylation> <coverage>"
- [readme] fractionalMethylation - values should be between 0 (fully unmethylated) and 1 (fully methylated), if the program you use for preprocessing outputs values between 0 and 100 please divide by 100 first.: "fractionalMethylation - values should be between 0 (fully unmethylated) and 1 (fully methylated), if the program you use for preprocessing outputs values between 0 and 100 please divide by 100 first."
- [methods] Convert your methylation data to a tab-delimited BED-like file with these columns: chromosome position1 position2 methylation_level coverage.: "Convert your methylation data to a tab-delimited BED-like file with these columns: chromosome position1 position2 methylation_level coverage."
- [methods] We generally recommend collapsing data across strands prior to running DXM.: "We generally recommend collapsing data across strands prior to running DXM."
- [methods] All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation.: "All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation."
