---
name: clinical-correlate-association-analysis
description: Use when per-sample subclonal methylation feature counts (e.g. i-DMRs
  from DXM on bisulfite data) must be related to clinical outcomes such as relapse
  or patient covariates like age.
when_to_use_negative:
- 'When only 18 patients are available: the article notes initial findings based on
  18 patients need validation in a larger cohort, so avoid clinical association claims
  on that sample size.'
- When the methylation calls are cross-sample DMRs (e.g. DSS-called) rather than intrasample
  i-DMRs — the findings concern DXM i-DMR counts, and the article notes i-DMRs differ
  from DSS-called DMRs.
- When samples have extreme methylation changes (e.g. DNMT3A mutation or demethylating-agent
  treatment) without retuning DXM parameters, since downstream i-DMR counts would
  be unreliable.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_2640
tools:
- name: DXM
  role: Deconvolves each sample's eRRBS bisulfite data into two-subpopulation methylation
    profiles and calls the per-sample i-DMRs that are counted and correlated with
    clinical outcomes
  repo: https://github.com/edwardslab-wustl/dxm
- name: Python3
  role: Used (with numpy) to tabulate per-sample i-DMR counts and compute the age
    correlation and relapse association
- name: numpy
  role: Required Python package supporting the DXM install and downstream tabulation/correlation
    computations
- name: bedtools
  role: Its 'intersect' command assigns unique promoter/CGI region names so i-DMR
    counts can be restricted to gene promoters
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:3fa2e50f06926b932f39e78edfa0802a2b4c66bf54a0b6c8d526ab37f42373a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - DXM
  - Python3
  - bedtools
  - numpy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# clinical-correlate-association-analysis

## Summary

Tabulates per-sample counts of DXM-called intrasample DMRs (i-DMRs) from a 31-DLBCL eRRBS cohort and tests whether fewer i-DMR-bearing gene promoters at diagnosis associate with patient relapse, plus correlation of counts with patient age.

## When to use

Use when per-sample subclonal methylation feature counts (e.g. i-DMRs from DXM on bisulfite data) must be related to clinical outcomes such as relapse or patient covariates like age.

## When NOT to use

- When only 18 patients are available: the article notes initial findings based on 18 patients need validation in a larger cohort, so avoid clinical association claims on that sample size.
- When the methylation calls are cross-sample DMRs (e.g. DSS-called) rather than intrasample i-DMRs — the findings concern DXM i-DMR counts, and the article notes i-DMRs differ from DSS-called DMRs.
- When samples have extreme methylation changes (e.g. DNMT3A mutation or demethylating-agent treatment) without retuning DXM parameters, since downstream i-DMR counts would be unreliable.

## Inputs

- Per-sample DXM output files: <prefix>_allVitProb.txt, <prefix>_reconstructed_2_subpops.txt, and <prefix>_DXM_dmrs.txt from 31 DLBCL eRRBS samples (GSE130556)
- Clinical metadata for the 31 DLBCL patients (relapse status, age at diagnosis)
- CpG island annotations (cgi.bed from UCSC Table Browser) used to assign promoter region names

## Outputs

- dlbcl_idmr_counts.csv — per-sample i-DMR promoter counts joined with relapse status and age
- Association statistics: correlation of i-DMR counts with age (expected r^2 ≈ 0.19) and relapse-rate comparison between low- and high-i-DMR groups

## How to apply

After running dxm_solveMethylation (-c set to the maximum coverage, e.g. 400) and dxm_callIDMR on each of the 31 GSE130556 DLBCL samples, count i-DMRs per sample from the <prefix>_DXM_dmrs.txt files, restricting to gene promoters (e.g. CGI-annotated regions from the UCSC cgi.bed intersect). Sanity-check counts against the reported range of 3708-18200 i-DMRs per sample (average 10826). Then correlate per-sample i-DMR counts with patient age (expecting the reported weak r^2 = 0.19) and test the association between fewer i-DMR-bearing promoters at diagnosis and relapse, writing the summary table to dlbcl_idmr_counts.csv (e.g. with Python3/numpy).

## Related tools

- **DXM** (Deconvolves each sample's eRRBS bisulfite data into two-subpopulation methylation profiles and calls the per-sample i-DMRs that are counted and correlated with clinical outcomes) — https://github.com/edwardslab-wustl/dxm
- **Python3** (Used (with numpy) to tabulate per-sample i-DMR counts and compute the age correlation and relapse association)
- **numpy** (Required Python package supporting the DXM install and downstream tabulation/correlation computations)
- **bedtools** (Its 'intersect' command assigns unique promoter/CGI region names so i-DMR counts can be restricted to gene promoters)

## Examples

```
dxm_callIDMR -v testSample_allVitProb.txt -m testSample_reconstructed_2_subpops.txt -o putative_<sample>
```

## Evaluation signals

- Per-sample i-DMR counts fall within the reported range of 3708-18200 with an average of about 10826.
- Age vs. i-DMR-count correlation reproduces the reported weak association, r^2 = 0.19.
- The relapse test reproduces the direction of the finding: patients with fewer i-DMR-bearing gene promoters at diagnosis have higher relapse rates.
- dlbcl_idmr_counts.csv contains 31 rows, one per DLBCL sample, with count, age, and relapse columns populated.
- Every input sample produced both <prefix>_reconstructed_2_subpops.txt and <prefix>_allVitProb.txt before dxm_callIDMR was run.

## Limitations

- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpreting results in samples with high 5hmC levels such as neuronal cell types.
- Care must be taken interpreting DXM results in regions with allele-specific methylation or copy number variants, which can confound i-DMR counts.
- The initial clinical finding was based on 18 patients and needs validation in a larger cohort; the 31-sample cohort is the validation set.
- Retuning DXM parameters (e.g. transition probabilities) may be needed for samples with extreme methylation changes such as DNMT3A mutation or demethylating agent treatment.

## Evidence

- [abstract] we apply DXM to a cohort of 31 DLBCLs and relate allelic subpopulation methylation profiles to relapse: "we apply DXM to a cohort of 31 DLBCLs and relate allelic subpopulation methylation profiles to relapse"
- [discussion] While our initial findings based on 18 patients need to be validated in a larger cohort: "While our initial findings based on 18 patients need to be validated in a larger cohort, they illustrate how DXM could be applied to analyze subpopulation methylation profiles in cancer samples."
- [methods] Call intrasample differentially methylated regions from solved methylation profiles.: "Call intrasample differentially methylated regions from solved methylation profiles."
- [readme] The output of dxm_callIDMR is <prefix>_DXM_dmrs.txt: "The output of dxm_callIDMR is \<prefix\>_DXM_dmrs.txt (e.g. putative_DXMdmrs.txt). Its format is tab-delimited:"
- [supplementary] All bisulfite sequencing data has been made available as a GEO accession GSE130556.: "All bisulfite sequencing data has been made available as a GEO accession GSE130556."
