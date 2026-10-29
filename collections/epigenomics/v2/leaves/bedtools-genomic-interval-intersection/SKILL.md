---
name: bedtools-genomic-interval-intersection
description: 'Use when preparing per-cytosine bismark/bsmap/biscuit methylation output
  for DXM: each CpG needs a unique regionName from an overlapping feature BED (CGIs,
  promoters, imprinted DMRs) before dxm_solveMethylation can run.'
when_to_use_negative:
- The methylation file already carries unique region names per tested region and is
  in DXM's six-column format — re-intersecting would be redundant
- The goal is deconvolving methylation across a whole chromosome or an unannotated
  region with no feature BED, since DXM requires unique region names per region of
  interest and large regions with many CpGs need more CpGs with methylation change
  to be solved as distinct profiles
- The feature BED's genome build does not match the methylation data's build — intersect
  would mislabel coordinates
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: bedtools
  role: Overlaps per-CpG methylation BED files with CGI/promoter/imprinted-DMR interval
    BEDs to assign unique region names for DXM input
  repo: https://bedtools.readthedocs.io/en/latest/
- name: DXM
  role: Downstream consumer of the intersect-produced region-named methylation BED
    for subpopulation deconvolution
  repo: https://github.com/edwardslab-wustl/dxm
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:8d6dd5504797e1b47e04ba5c63076ccf6ca7df4822e7c062301b438df27d13c7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DXM
  - Docker
  - bedtools
  - numpy
  - python3
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# bedtools-genomic-interval-intersection

## Summary

Use bedtools intersect (-wo) to assign unique region names (e.g. CpG island or imprinted DMR identifiers) to per-CpG bisulfite methylation BED files, producing the DXM-compatible six-column input format required before subpopulation deconvolution.

## When to use

Use when preparing per-cytosine bismark/bsmap/biscuit methylation output for DXM: each CpG needs a unique regionName from an overlapping feature BED (CGIs, promoters, imprinted DMRs) before dxm_solveMethylation can run.

## When NOT to use

- The methylation file already carries unique region names per tested region and is in DXM's six-column format — re-intersecting would be redundant
- The goal is deconvolving methylation across a whole chromosome or an unannotated region with no feature BED, since DXM requires unique region names per region of interest and large regions with many CpGs need more CpGs with methylation change to be solved as distinct profiles
- The feature BED's genome build does not match the methylation data's build — intersect would mislabel coordinates

## Inputs

- Per-cytosine methylation file from bismark_methylation_extractor (.cov: chr pos1 pos2 methylation% meth_cov unmeth_cov) or equivalent bsmap/biscuit output
- Feature BED file of genomic intervals (e.g. cgi.bed from UCSC Table Browser, refGene_hg19_10kb_fixed.txt.gz promoter regions, or imprinted DMR BED)
- bedtools installed on the command line

## Outputs

- DXM input BED-like file (e.g. dxm_in.bed) with columns: chromosome, position1, position2, regionName, fractionalMethylation (0-1), integer coverage

## How to apply

First convert aligner methylation calls to a tab-delimited BED-like file with columns chromosome, position1, position2, methylation_level, coverage, collapsing data across strands and filtering positions whose coverage exceeds the maximum expected sequencing coverage (set later as the -c flag of dxm_solveMethylation). Download a feature BED (e.g. cgi.bed from the UCSC Genome Browser Table Browser for the matching genome build, refGene_hg19_10kb_fixed.txt.gz for hg19 TSS ±5kb, or an imprinted DMR BED) and run 'bedtools intersect -wo -a methylation.bed -b cgi.bed', then select columns with awk so the output carries chr, pos1, pos2, the feature name as regionName, fractional methylation (0-1), and integer coverage. The -wo flag writes all overlapping pairs so each CpG is labeled with the feature it falls in; the awk column indices may need adjusting for a custom feature BED. The resulting region-named file (e.g. dxm_in.bed) restricts DXM's deconvolution to the chosen genomic compartments, since DXM computes across all CpGs sharing a region name.

## Related tools

- **bedtools** (Overlaps per-CpG methylation BED files with CGI/promoter/imprinted-DMR interval BEDs to assign unique region names for DXM input) — https://bedtools.readthedocs.io/en/latest/
- **DXM** (Downstream consumer of the intersect-produced region-named methylation BED for subpopulation deconvolution) — https://github.com/edwardslab-wustl/dxm

## Examples

```
bedtools intersect -wo -a methylation.bed -b cgi.bed | awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed
```

## Evaluation signals

- Output file has exactly six tab-delimited columns: chromosome, position1, position2, regionName, fractionalMethylation (between 0 and 1), integer coverage
- Every row's regionName is non-empty and traceable to an entry in the feature BED (e.g. a CGI ID from cgi.bed)
- All coverage values are at or below the maximum expected sequencing coverage that will be passed as -c to dxm_solveMethylation (e.g. 400)
- Fractional methylation values are in [0,1]; values on a 0-100 scale indicate the aligner conversion step (e.g. divide by 100) was skipped
- The region-restricted analysis behaves as expected downstream: ~92% of female vs ~14% of male X-linked CGIs called differentially methylated, and ~59% of imprinted loci solved as two subpopulations in primed WIBR3 ES cells

## Limitations

- DXM does not use the chromosome or position2 columns internally, so mislabeled coordinates would not error out but would corrupt any later coordinate-based interpretation
- Column selection in the awk statement depends on the feature BED layout; using a custom BED without adjusting the printed columns yields wrong regionName/methylation assignment
- Larger feature regions (more CpGs) require more CpGs with methylation change to be solved as a distinct profile by DXM, so interval choice affects sensitivity

## Evidence

- [methods] bedtools intersect -wo -a methylation.bed -b cgi.bed | awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed: "bedtools intersect -wo -a methylation.bed -b cgi.bed | awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed"
- [readme] The methylation data input file should be in a tab-delimited bed-like format: <chr> <position1> <position2> <regionName> <fractionalMethylation> <coverage>: "The methylation data input file should be in a tab-delimited bed-like format:

<chr> <position1> <position2> <regionName> <fractionalMethylation> <coverage>"
- [readme] We recommend adding region names with utilities such as the 'intersect' command from bedtools: "We recommend adding region names with utilities such as the 'intersect' command from [bedtools](https://bedtools.readthedocs.io/en/latest/)."
- [readme] We generally recommend collapsing data across strands prior to running DXM.: "We generally recommend collapsing data across strands prior to running DXM."
- [readme] All data should be filtered such that the coverage is below the maximum expected sequencing coverage: "All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation."
