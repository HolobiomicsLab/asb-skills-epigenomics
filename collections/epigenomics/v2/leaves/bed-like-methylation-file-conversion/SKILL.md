---
name: bed-like-methylation-file-conversion
description: Use when preparing bisulfite sequencing methylation callsets (e.g. bismark_methylation_extractor
  .cov files) as input for DXM's dxm_solveMethylation, which requires a tab-delimited
  BED-like file with a unique regionName per region.
when_to_use_negative:
- Your data is already in the DXM 6-column BED-like format with unique region names
  — reconversion or a second bedtools intersect would duplicate region-name assignment.
- You need methylation profiles solved genome-wide rather than over defined regions
  — DXM requires user-defined regions with unique names, and a whole-chromosome profile
  is impractical given assembly gaps and repetitive DNA.
- Fractional methylation values are on a 0–100 scale and would be passed unconverted
  — DXM requires values between 0 (fully unmethylated) and 1 (fully methylated).
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: DXM
  role: Target tool whose required tab-delimited BED-like input format (chr position1
    position2 regionName fractionalMethylation coverage) this conversion produces
  repo: https://github.com/edwardslab-wustl/dxm
- name: bedtools
  role: Overlaps methylation positions with region annotations (cgi.bed or refGene_hg19_10kb_fixed.txt.gz)
    via 'intersect -wo' to assign unique region names
- name: Bismark
  role: Upstream bisulfite aligner whose bismark_methylation_extractor .cov output
    is the starting point for awk-based conversion
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:82d6db474c6a3f8664875de2d0f7d8ae9fe12ef9f186e73edaa9c9837425c8e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - Bismark
  - DXM
  - bedtools
  - numpy
  - python3
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# bed-like-methylation-file-conversion

## Summary

Convert aligner methylation-extractor output (e.g. Bismark .cov) into the tab-delimited BED-like 6-column format required by DXM (chr, position1, position2, regionName, fractionalMethylation, coverage), assigning unique region names via bedtools intersect with an annotation bed.

## When to use

Use when preparing bisulfite sequencing methylation callsets (e.g. bismark_methylation_extractor .cov files) as input for DXM's dxm_solveMethylation, which requires a tab-delimited BED-like file with a unique regionName per region.

## When NOT to use

- Your data is already in the DXM 6-column BED-like format with unique region names — reconversion or a second bedtools intersect would duplicate region-name assignment.
- You need methylation profiles solved genome-wide rather than over defined regions — DXM requires user-defined regions with unique names, and a whole-chromosome profile is impractical given assembly gaps and repetitive DNA.
- Fractional methylation values are on a 0–100 scale and would be passed unconverted — DXM requires values between 0 (fully unmethylated) and 1 (fully methylated).

## Inputs

- bismark_methylation_extractor .cov file (chr pos1 pos2 methylation meth_cov unmeth_cov)
- Region annotation bed file (e.g. UCSC cgi.bed or refGene_hg19_10kb_fixed.txt.gz with TSS +/-5kb promoters, hg19)

## Outputs

- Tab-delimited BED-like DXM input file with columns: chromosome, position1, position2, regionName, fractionalMethylation, coverage

## How to apply

Start from the .cov output of bismark_methylation_extractor (chr pos1 pos2 methylation meth_cov unmeth_cov) and compute coverage as the sum of methylated plus unmethylated reads with awk, printing fractional methylation values between 0 and 1 (divide by 100 if your preprocessor outputs 0–100). Collapse data across strands before running DXM, and filter positions so coverage stays below the maximum expected sequencing coverage passed via the -c flag of dxm_solveMethylation (e.g. 55 for simulated mixtures, default 400). Assign unique region names per region by overlapping the methylation bed with a features file using bedtools intersect -wo — either a UCSC Table Browser cgi.bed or the refGene_hg19_10kb_fixed.txt.gz file (TSS ±5kb promoters) from DXM's example_data — then use awk to project the correct columns into the 6-column DXM format. DXM ignores the chr and position2 columns internally but requires them for BED compatibility, so they must still be present.

## Related tools

- **DXM** (Target tool whose required tab-delimited BED-like input format (chr position1 position2 regionName fractionalMethylation coverage) this conversion produces) — https://github.com/edwardslab-wustl/dxm
- **bedtools** (Overlaps methylation positions with region annotations (cgi.bed or refGene_hg19_10kb_fixed.txt.gz) via 'intersect -wo' to assign unique region names)
- **Bismark** (Upstream bisulfite aligner whose bismark_methylation_extractor .cov output is the starting point for awk-based conversion)

## Examples

```
bedtools intersect -wo -a methylation.bed -b refGene_hg19_10kb_fixed.txt | awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed
```

## Evaluation signals

- Output file has exactly 6 tab-delimited columns in the order: chromosome, position1, position2, regionName, fractionalMethylation, coverage.
- All fractionalMethylation values fall between 0 and 1 (not 0–100).
- Every position carries a non-empty, unique regionName inherited from the intersected annotation (e.g. one name per promoter).
- Coverage values are integers and all fall below the maximum expected sequencing coverage supplied as the -c flag of dxm_solveMethylation (e.g. 55 for 55x simulated mixtures, default 400).
- Data has been collapsed across strands prior to conversion, as recommended by the DXM authors.

## Limitations

- DXM does not explicitly use the chromosome or position2 columns, so annotations must be genome-version-matched (e.g. hg19 refGene_hg19_10kb_fixed.txt.gz) even though coordinates are only used for region naming.
- The awk column indices in the bedtools intersect pipeline may need adjustment if a custom bed file of features replaces cgi.bed.
- Coverage must be filtered below the maximum expected sequencing coverage; setting -c higher than the true maximum needlessly increases memory usage.

## Evidence

- [readme] The methylation data input file should be in a tab-delimited bed-like format: "The methylation data input file should be in a tab-delimited bed-like format:

<chr> <position1> <position2> <regionName> <fractionalMethylation> <coverage>"
- [methods] Convert your methylation data to a tab-delimited BED-like file: "Convert your methylation data to a tab-delimited BED-like file with these columns: chromosome position1 position2 methylation_level coverage."
- [methods] bedtools intersect command with awk projection: "bedtools intersect -wo -a methylation.bed -b cgi.bed | awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed"
- [readme] Bismark .cov as starting point for conversion: "If you are using bismark, start with the .cov produced by bismark_methylation_extractor (chr pos1 pos2 methylation meth_cov unmeth_cov)."
- [readme] Collapse across strands and filter coverage below maximum: "All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation.
We generally recommend collapsing data across strands"
