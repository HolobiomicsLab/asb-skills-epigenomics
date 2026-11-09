---
name: bisulfite-methylation-data-deconvolution
description: Use when bisulfite sequencing (e.g. Methyl-Seq, RRBS/WGBS) data from
  a heterogeneous sample must be resolved into two subpopulation methylation profiles
  and fractional prevalences over defined regions.
when_to_use_negative:
- Regions with pre-existing allele-specific methylation or copy number variants, where
  two methylation states reflect allelic or CNV effects rather than subclonal subpopulations
- Samples with high 5-hydroxymethylation (e.g. neuronal cell types), since DXM does
  not consider 5hmC
- Solving a single global methylation profile across an entire chromosome, which is
  impractical given assembly gaps and large repetitive DNA sequences
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: DXM
  role: 'Core deconvolution tool: dxm_solveMethylation solves 1- and 2-subpopulation
    methylation profiles and prevalences; dxm_callIDMR calls intrasample differentially
    methylated regions; dxm_estimateFracs estimates subpopulation prevalences'
  repo: https://github.com/edwardslab-wustl/dxm
- name: bedtools
  role: Overlaps methylation BED with region annotations (cgi.bed, refGene promoters)
    to assign unique region names for DXM input
- name: numpy
  role: Required Python package for DXM execution
- name: Bismark
  role: Bisulfite aligner whose bismark_methylation_extractor .cov output is converted
    (via awk) into the DXM input BED-like format
- name: Docker
  role: Alternative installation route via the edwardslab/dxm container image
  repo: https://hub.docker.com/r/edwardslab/dxm
- name: methylPurify
  role: Benchmarked comparator for subclonal methylation deconvolution; requires code
    edits to handle reads longer than 100 bp
provenance:
  source_task_ids:
  - task_005
  - task_003
  - task_006
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:de6e72b23f4d39ac2249a3e5c00b15cf97bc36b98cbde6bfa24daeee2e61871f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - Bismark
  - DXM
  - Docker
  - bedtools
  - methylPurify
  - numpy
  - python3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# bisulfite-methylation-data-deconvolution

## Summary

Deconvolve methylation profiles of major allelic subpopulations from heterogeneous bisulfite sequencing data using DXM, without prior knowledge of the number of subpopulations or cell types. Applies to CpG island, promoter, X-linked CGI, imprinted DMR, and DLBCL tumor datasets.

## When to use

Use when bisulfite sequencing (e.g. Methyl-Seq, RRBS/WGBS) data from a heterogeneous sample must be resolved into two subpopulation methylation profiles and fractional prevalences over defined regions.

## When NOT to use

- Regions with pre-existing allele-specific methylation or copy number variants, where two methylation states reflect allelic or CNV effects rather than subclonal subpopulations
- Samples with high 5-hydroxymethylation (e.g. neuronal cell types), since DXM does not consider 5hmC
- Solving a single global methylation profile across an entire chromosome, which is impractical given assembly gaps and large repetitive DNA sequences

## Inputs

- Tab-delimited BED-like methylation file: chr, position1, position2, regionName, fractionalMethylation (0-1), integer coverage
- Bisulfite sequencing methylation callsets (bismark .cov from bismark_methylation_extractor, bsmap, biscuit output)
- Region annotation BED (cgi.bed from UCSC Table Browser, refGene_hg19_10kb_fixed.txt.gz for TSS +/-5kb, imprinted DMR BED)
- GEO GSE130556 bisulfite sequencing data (DLBCL biopsies, sorted GCB/monocyte, REP/BEP somatic samples)
- WIBR3 primed vs naive ES cell data from GSE75868

## Outputs

- <prefix>_reconstructed_1_subpops.txt (regions with 1 major methylation profile)
- <prefix>_reconstructed_2_subpops.txt (minor and major subpopulation methylation states per region)
- <prefix>_allVitProb.txt (relative posterior probabilities)
- <prefix>_solvedPrevalences.txt from dxm_estimateFracs (fractional prevalence per subpopulation)
- <prefix>_DXM_dmrs.txt iDMR calls from dxm_callIDMR (chr, start, end, regionName)
- Benchmark/validation tables (iDMR_rates_summary.tsv, benchmark_accuracy.csv)

## How to apply

Convert aligner methylation calls (bismark .cov, bsmap, biscuit) into a tab-delimited BED-like file with columns chromosome, position1, position2, regionName, fractionalMethylation (0-1), integer coverage, collapsing across strands and filtering coverage below the maximum expected sequencing coverage. Assign unique region names by overlapping the methylation file with a region set (cgi.bed from UCSC Table Browser, or refGene_hg19_10kb_fixed.txt.gz for TSS +/-5kb promoters) using 'bedtools intersect -wo' with awk column selection. Run dxm_solveMethylation with -c set to the maximum coverage in the input (e.g. 55 for 55x mixtures, 400 for Methyl-Seq DLBCL samples), which solves 1- and 2-subpopulation profiles and reports fractional prevalences built into the solver. Optionally call intrasample DMRs with dxm_callIDMR using the allVitProb posterior file and the reconstructed_2_subpops output. Validate against ground truth or flow/FACS-sorted references (e.g. CD4+ T and CD19+ B cells in DLBCL), expecting per-CpG accuracies near 98.5%/96.2%/87.1% for single/major/minor profiles and prevalence agreement within ~0.0875.

## Related tools

- **DXM** (Core deconvolution tool: dxm_solveMethylation solves 1- and 2-subpopulation methylation profiles and prevalences; dxm_callIDMR calls intrasample differentially methylated regions; dxm_estimateFracs estimates subpopulation prevalences) — https://github.com/edwardslab-wustl/dxm
- **bedtools** (Overlaps methylation BED with region annotations (cgi.bed, refGene promoters) to assign unique region names for DXM input)
- **numpy** (Required Python package for DXM execution)
- **Bismark** (Bisulfite aligner whose bismark_methylation_extractor .cov output is converted (via awk) into the DXM input BED-like format)
- **Docker** (Alternative installation route via the edwardslab/dxm container image) — https://hub.docker.com/r/edwardslab/dxm
- **methylPurify** (Benchmarked comparator for subclonal methylation deconvolution; requires code edits to handle reads longer than 100 bp)

## Examples

```
dxm_solveMethylation -i dxm_in.bed -c 400 -o testSample && dxm_callIDMR -v testSample_allVitProb.txt -m testSample_reconstructed_2_subpops.txt -o putative
```

## Evaluation signals

- Per-CpG reconstruction accuracy reaches reported tiers: 98.5% single-pattern promoters, 96.2% major profiles, 87.1% minor profiles in simulated 55x GCB:monocyte mixtures
- DXM-estimated T/B-cell fractional prevalences agree with FACS/flow values within ~0.0875 in the four DLBCL samples (Figure 6C)
- Per-sample i-DMR counts fall within the reported 3785-6790 promoter range
- X-linked CGI differential methylation rates match reported ~92% female vs ~14% male, and imprinted two-subpopulation call rates match 59% (primed) vs <10% (naive) WIBR3
- Input validation: fractionalMethylation in [0,1], integer coverage, coverage below the -c/-m maximum, and unique regionName per region

## Limitations

- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpretation in samples with high 5hmC such as neuronal cell types
- Care must be taken interpreting results in regions with allele-specific methylation or copy number variants
- Larger regions require more CpGs with methylation change to be solved as a distinct profile; parameter retuning (e.g. transition probabilities) may be needed for extreme methylation changes such as DNMT3A mutation or demethylating agent treatment
- dxm_estimateFracs is INCOMPATIBLE with dxm_solveMethylation, which has its own fractional prevalence solution call
- Initial DLBCL findings based on 18 patients need validation in a larger cohort

## Evidence

- [abstract] DXM does not require prior knowledge of the number of subpopulations or cell types: "DXM does not require prior knowledge of the number of subpopulations or types of cells to expect"
- [readme] Input format and region naming: "The methylation data input file should be in a tab-delimited bed-like format: <chr> <position1> <position2> <regionName> <fractionalMethylation> <coverage>"
- [methods] Region overlap command: "bedtools intersect -wo -a methylation.bed -b cgi.bed | awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed"
- [methods] Coverage filtering: "All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation."
- [readme] Computation time expectation: "DXM solved eRRBS and Methyl-Seq samples for 2 subpopulations at average coverage of 60x in ~2 hours."
- [discussion] 5hmC caveat: "DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpreting results in samples with high levels of 5hmC"
