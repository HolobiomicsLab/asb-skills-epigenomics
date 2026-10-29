---
name: x-inactivation-cgi-methylation-analysis
description: Use when bisulfite sequencing data (bismark/bsmap/biscuit .cov or similar)
  should be tested for allele-specific methylation at X-linked CGIs or imprinted DMRs,
  e.g. female-vs-male X-inactivation skews or primed-vs-naive imprinting loss.
when_to_use_negative:
- Samples or regions with high 5-hydroxymethylation (e.g. neuronal cell types), since
  DXM does not consider 5hmC
- Regions with known copy-number variants or extreme methylation changes (e.g. DNMT3A
  mutation or demethylating-agent treatment) without retuning DXM parameters, where
  interpretation is unreliable
- Whole-chromosome-scale methylation profiling, which is impractical given assembly
  gaps and repetitive sequence — DXM is designed for user-defined regions
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: DXM
  role: Deconvolves region-restricted methylation data into major allelic subpopulation
    profiles (dxm_solveMethylation) and calls iDMRs (dxm_callIDMR)
  repo: https://github.com/edwardslab-wustl/dxm
- name: bedtools
  role: Intersects the methylation BED file with CGI or imprinted-DMR BED files and
    extracts the correct columns for the DXM input
- name: Python3
  role: Runtime for DXM (requires python 3.5+, numpy, cython >=0.24); also used to
    tally iDMR rates into the summary table
  repo: https://github.com/edwardslab-wustl/dxm
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:9ce72824c8d30b98ca192adaeb6d50a8a667bd4023c68451118ec3f7d10936e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - DXM
  - Python3
  - bedtools
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# x-inactivation-cgi-methylation-analysis

## Summary

Applies DXM deconvolution to bisulfite sequencing data restricted to X-linked CpG islands and imprinted DMRs to detect allele-specific methylation signatures of X-inactivation and imprinting, including state-dependent imprinting loss in primed vs naive ES cells.

## When to use

Use when bisulfite sequencing data (bismark/bsmap/biscuit .cov or similar) should be tested for allele-specific methylation at X-linked CGIs or imprinted DMRs, e.g. female-vs-male X-inactivation skews or primed-vs-naive imprinting loss.

## When NOT to use

- Samples or regions with high 5-hydroxymethylation (e.g. neuronal cell types), since DXM does not consider 5hmC
- Regions with known copy-number variants or extreme methylation changes (e.g. DNMT3A mutation or demethylating-agent treatment) without retuning DXM parameters, where interpretation is unreliable
- Whole-chromosome-scale methylation profiling, which is impractical given assembly gaps and repetitive sequence — DXM is designed for user-defined regions

## Inputs

- Bisulfite sequencing methylation calls (bismark .cov / bsmap / biscuit output) from REP/BEP somatic samples (GEO GSE130556) and WIBR3 primed vs naive ES cells (GEO GSE75868)
- A BED file of X-linked CpG islands (e.g. from the UCSC Genome Browser Table Browser, matching genome version)
- A BED file of known imprinted DMRs
- bedtools for region intersection

## Outputs

- dxm_in.bed — filtered, region-restricted methylation input in DXM BED-like format
- <prefix>_reconstructed_1_subpops.txt and <prefix>_reconstructed_2_subpops.txt — solved methylation profiles
- <prefix>_DXM_dmrs.txt — intrasample differentially methylated region (iDMR) calls
- iDMR_rates_summary.tsv — per-dataset, per-region-class rates of differential methylation compared with reported reference ranges

## How to apply

Convert aligner output to the DXM BED-like format (chromosome, position1, position2, regionName, fractionalMethylation 0-1, integer coverage), collapsing strands first. Restrict to regions of interest with 'bedtools intersect -wo -a methylation.bed -b cgi.bed' plus awk column extraction, substituting an imprinted-DMR bed for cgi.bed when analyzing imprinting. Run dxm_solveMethylation with -c set to the maximum coverage in the input (setting it higher needlessly increases memory usage), then run dxm_callIDMR with the posterior-probability file (-v) and the 2-subpopulation reconstruction (-m) to obtain iDMR calls. Tally, per region class and condition, the fraction of X-linked CGIs called differentially methylated stratified by sex, and the fraction of imprinted DMRs solved as two subpopulations — expect ~92% female vs ~14% male for X-linked CGIs, and 59% of imprinted loci in primed WIBR3 vs <10% after naive conversion, reflecting loss of imprints.

## Related tools

- **DXM** (Deconvolves region-restricted methylation data into major allelic subpopulation profiles (dxm_solveMethylation) and calls iDMRs (dxm_callIDMR)) — https://github.com/edwardslab-wustl/dxm
- **bedtools** (Intersects the methylation BED file with CGI or imprinted-DMR BED files and extracts the correct columns for the DXM input)
- **Python3** (Runtime for DXM (requires python 3.5+, numpy, cython >=0.24); also used to tally iDMR rates into the summary table) — https://github.com/edwardslab-wustl/dxm

## Examples

```
bedtools intersect -wo -a methylation.bed -b cgi.bed | awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed && dxm_solveMethylation -i dxm_in.bed -c 400 -o sample && dxm_callIDMR -v sample_allVitProb.txt -m sample_reconstructed_2_subpops.txt -o putative
```

## Evaluation signals

- Computed fraction of female X-linked CGIs called differentially methylated falls near the reported ~92%, with male samples near ~14%
- Fraction of imprinted DMRs solved as two subpopulations falls in the reported 99.0%/99.3% range
- Primed WIBR3 shows ~59% of imprinted loci with two subpopulations versus <10% in naive-state WIBR3, consistent with expected loss of imprints
- Output file iDMR_rates_summary.tsv contains observed rates alongside reported values and passes value_in_range / output_matches_reference checks
- Input coverage never exceeds the -c value supplied to dxm_solveMethylation, and all fractional methylation values lie in [0,1]

## Limitations

- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpretation in samples with high levels of 5hmC
- Care must be taken interpreting DXM results in regions with allele-specific methylation or copy number variants
- Region size affects results: as regions get larger and include more CpGs, more CpGs must change methylation to be solved as a distinct profile
- Data should be collapsed across strands prior to running DXM, and coverage filtered below the maximum expected sequencing coverage

## Evidence

- [results] DXM detected multiple methylation patterns at 59% of imprinted loci in primed WIBR3 ES cells but at less than 10% of imprinted loci after conversion to the naive state, consistent with expected loss of imprints.: [evidence span withheld — anchor status: not_found]
- [methods] bedtools intersect workflow: "bedtools intersect -wo -a methylation.bed -b cgi.bed | awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed"
- [readme] coverage filtering note: "Set the -c flag to the maximum coverage in the input file.  Increasing -c above the maximum coverage will needlessly increase memory usage."
- [discussion] 5hmC limitation: "DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpreting results in samples with high levels of 5hmC"
- [discussion] region-size caveat: "As regions get larger and include more CpGs, the number of CpGs that must have a change in methylation in order to be solved as a distinct methylation profile by DXM will also increase."
