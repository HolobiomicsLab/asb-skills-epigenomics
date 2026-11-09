---
name: intrasample-differential-methylation-region-calling
description: Use when dxm_solveMethylation has reconstructed 1- and 2-subpopulation
  methylation profiles for a heterogeneous bisulfite sequencing sample (eRRBS or Agilent
  Methyl-Seq BED-like input) and you need per-region i-DMRs, e.g., promoter-level
  subclonal heterogeneity counts to relate to DLBCL relapse.
when_to_use_negative:
- Calling differential methylation between separate samples or sorted cell types (between-sample
  DMR calling, e.g., DSS with delta 0.3 on GCB vs monocyte counts) — dxm_callIDMR
  only detects differences among subpopulations inside one sample.
- Regions with known allele-specific methylation or copy-number variants, where two
  solved 'profiles' may reflect parental alleles or copy state rather than true subclones.
- Samples rich in 5-hydroxymethylation (e.g., neuronal cell types), since DXM does
  not consider 5hmC in its model.
edam_topics:
- http://edamontology.org/topic_3295
tools:
- name: DXM
  role: dxm_callIDMR consumes dxm_solveMethylation's *_allVitProb.txt posterior probabilities
    and *_reconstructed_2_subpops.txt profiles to emit <prefix>_DXM_dmrs.txt i-DMRs
  repo: https://github.com/edwardslab-wustl/dxm
- name: bedtools
  role: intersect -wo of methylation.bed against cgi.bed assigns the unique region
    names that define which genomic regions i-DMRs are called within and against which
    DSS/i-DMR overlap is computed
  repo: https://bedtools.readthedocs.io/en/latest/
- name: DSS
  role: 'independent between-sample DMR caller (equal dispersion, delta 0.3, p < 0.05)
    used as the concordance benchmark: ~64% of DSS DMRs were recovered as i-DMRs in
    simulated mixtures'
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:cba290130a536c30b20317cb9895478a42dccacf71c160debc8a57077c375151
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - DSS
  - DXM
  - Docker
  - bedtools
  - numpy
  - python3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# intrasample-differential-methylation-region-calling

## Summary

Calls intrasample differentially methylated regions (i-DMRs) — loci where two subpopulations within one heterogeneous bisulfite-sequenced sample carry distinct methylation profiles — using DXM's dxm_callIDMR on the posterior probabilities and reconstructed two-subpopulation profiles produced by dxm_solveMethylation. Enables per-sample heterogeneity quantification, e.g., promoter i-DMR counts associated with relapse in the 31-DLBCL eRRBS cohort.

## When to use

Use when dxm_solveMethylation has reconstructed 1- and 2-subpopulation methylation profiles for a heterogeneous bisulfite sequencing sample (eRRBS or Agilent Methyl-Seq BED-like input) and you need per-region i-DMRs, e.g., promoter-level subclonal heterogeneity counts to relate to DLBCL relapse.

## When NOT to use

- Calling differential methylation between separate samples or sorted cell types (between-sample DMR calling, e.g., DSS with delta 0.3 on GCB vs monocyte counts) — dxm_callIDMR only detects differences among subpopulations inside one sample.
- Regions with known allele-specific methylation or copy-number variants, where two solved 'profiles' may reflect parental alleles or copy state rather than true subclones.
- Samples rich in 5-hydroxymethylation (e.g., neuronal cell types), since DXM does not consider 5hmC in its model.

## Inputs

- dxm_solveMethylation posterior probability file (<prefix>_allVitProb.txt)
- dxm_solveMethylation two-subpopulation reconstruction file (<prefix>_reconstructed_2_subpops.txt)
- Upstream: tab-delimited BED-like methylation input (chr, pos1, pos2, unique regionName, fractional methylation 0-1, integer coverage) from eRRBS/Methyl-Seq bisulfite data (e.g., converted bismark_methylation_extractor .cov)

## Outputs

- <prefix>_DXM_dmrs.txt: tab-delimited intrasample DMR list with columns chromosome, start coordinate, end coordinate, region name (multiple i-DMRs for the same region share the same region name)

## How to apply

First prepare the per-sample input as a tab-delimited BED-like file (chromosome, position1, position2, unique regionName, fractional methylation 0-1, integer coverage), collapsing across strands and filtering positions below the maximum expected coverage; assign unique region names by overlapping regions of interest (e.g., UCSC CGI bed) via 'bedtools intersect -wo -a methylation.bed -b cgi.bed' with awk column selection ($1,$2,$3,$9,$4,$5). Run 'dxm_solveMethylation -i <sample>.bed -c <max coverage, e.g., 400> -o <prefix>' to produce the <prefix>_allVitProb.txt posterior file and <prefix>_reconstructed_2_subpops.txt — dxm_callIDMR requires both, and dxm_estimateFracs output is incompatible with this workflow. Then run 'dxm_callIDMR -v <prefix>_allVitProb.txt -m <prefix>_reconstructed_2_subpops.txt -o putative', which writes <prefix>_DXM_dmrs.txt with columns chromosome, start coordinate, end coordinate, region name; multiple putative i-DMRs in the same region share the same region name. Judge the result against reported scale: per-sample promoter i-DMR counts of 3708-18200 (average 10826) in the 31-DLBCL eRRBS cohort, 3785-6790 promoters in the four DLBCL Methyl-Seq biopsies, and ~64% concordance with DSS-called DMRs on simulated GCB-monocyte mixtures.

## Related tools

- **DXM** (dxm_callIDMR consumes dxm_solveMethylation's *_allVitProb.txt posterior probabilities and *_reconstructed_2_subpops.txt profiles to emit <prefix>_DXM_dmrs.txt i-DMRs) — https://github.com/edwardslab-wustl/dxm
- **bedtools** (intersect -wo of methylation.bed against cgi.bed assigns the unique region names that define which genomic regions i-DMRs are called within and against which DSS/i-DMR overlap is computed) — https://bedtools.readthedocs.io/en/latest/
- **DSS** (independent between-sample DMR caller (equal dispersion, delta 0.3, p < 0.05) used as the concordance benchmark: ~64% of DSS DMRs were recovered as i-DMRs in simulated mixtures)

## Examples

```
dxm_callIDMR -v testSample_allVitProb.txt -m testSample_reconstructed_2_subpops.txt -o putative
```

## Evaluation signals

- Output file <prefix>_DXM_dmrs.txt exists and has exactly four tab-delimited columns: chromosome, start coordinate, end coordinate, region name.
- Multiple putative i-DMRs falling in the same input region carry the same region name (expected per the README).
- For the 31-DLBCL eRRBS cohort, per-sample promoter i-DMR counts fall in the reported 3708-18200 range (average ~10826); for the four DLBCL Methyl-Seq biopsies, counts fall in the 3785-6790 promoter range.
- bedtools intersect of DXM i-DMRs against DSS-called DMRs on simulated GCB-monocyte mixtures reproduces ~64% i-DMR/DMR concordance (and ~86% recovery of DSS-'missed' DMRs under refined region definitions).
- dxm_callIDMR was only run after dxm_solveMethylation produced both *_allVitProb.txt and *_reconstructed_2_subpops.txt — not with dxm_estimateFracs output, which is incompatible.

## Limitations

- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpreting i-DMRs in samples with high 5hmC levels such as neuronal cell types.
- Care must be taken in regions with allele-specific methylation or copy-number variants, which can be mistaken for two subpopulations.
- Region size affects sensitivity: as regions get larger and include more CpGs, more CpGs must change methylation for DXM to solve a distinct profile, altering i-DMR yield.
- i-DMR/DMR agreement with between-sample callers such as DSS is only ~64%, similar to variability observed between different DMR-callers, so i-DMRs should not be treated as exhaustive between-group DMRs.
- Samples with extreme methylation changes (e.g., DNMT3A mutation or demethylating agent treatment) may require retuning DXM parameters such as transition probabilities.

## Evidence

- [methods] Call intrasample differentially methylated regions from solved methylation profiles.: "Call intrasample differentially methylated regions from solved methylation profiles."
- [readme] dxm_callIDMR -v testSample_allVitProb.txt -m testSample_reconstructed_2_subpops.txt -o putative: "dxm_callIDMR -v testSample_allVitProb.txt -m testSample_reconstructed_2_subpops.txt -o putative"
- [readme] If there are multiple putative iDMRs for the same region, they will have the same corresponding region name.: "If there are multiple putative iDMRs for the same region, they will have the same corresponding region name."
- [readme] All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation.: "All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation."
- [discussion] The differences between DSS-called DMRs and i-DMRs in artificial mixtures are similar to those observed between different DMR-callers (62).: "The differences between DSS-called DMRs and i-DMRs in artificial mixtures are similar to those observed between different DMR-callers (62)."
- [discussion] care must be taken in interpreting DXM results in regions with allele-specific methylation or copy number variants.: "care must be taken in interpreting DXM results in regions with allele-specific methylation or copy number variants."
