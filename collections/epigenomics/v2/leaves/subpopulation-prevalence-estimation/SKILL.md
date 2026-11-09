---
name: subpopulation-prevalence-estimation
description: Use when bisulfite sequencing (Methyl-Seq or eRRBS) data from a heterogeneous
  sample must yield relative subpopulation proportions comparable to FACS- or flow-sorted
  cell fractions, and cell-type composition is unknown a priori.
when_to_use_negative:
- dxm_estimateFracs is INCOMPATIBLE with dxm_solveMethylation — do not use dxm_estimateFracs
  to obtain prevalences after a dxm_solveMethylation run; use dxm_solveMethylation's
  own fractional prevalence solution instead
- The sample has extreme methylation changes (e.g., DNMT3A mutation or demethylating
  agent treatment) without retuned DXM parameters, or contains regions with allele-specific
  methylation or copy number variants, which confound subpopulation interpretation
- The data are rich in 5-hydroxymethylation (e.g., neuronal cell types), which DXM
  does not consider
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0654
tools:
- name: DXM
  role: Deconvolves methylation profiles over user-defined regions via dxm_solveMethylation
    and provides the fractional prevalence solution for underlying subpopulations
  repo: https://github.com/edwardslab-wustl/dxm
- name: bedtools
  role: Assigns unique region names (e.g., promoter/CGI identifiers) to methylation
    BED files via 'bedtools intersect -wo' before DXM input
- name: python3
  role: Runtime for DXM (requires Python 3.5+) with numpy as a required package
  repo: https://github.com/edwardslab-wustl/dxm
- name: Docker
  role: Containerized distribution of DXM for reproducible deconvolution runs
  repo: https://hub.docker.com/r/edwardslab/dxm
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:5b8912f21fd5d5704443f5f5321abf42f1533570a0534c4b9fe8aea093584847
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - DXM
  - Docker
  - bedtools
  - numpy
  - python3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# subpopulation-prevalence-estimation

## Summary

Estimate the fractional prevalences of underlying subpopulations (e.g., CD4+ T and CD19+ B cells in DLBCL lymph node biopsies) from heterogeneous bisulfite sequencing data using DXM's deconvolution with its built-in prevalence solution, and validate the estimates against independently measured FACS/flow cytometry fractions.

## When to use

Use when bisulfite sequencing (Methyl-Seq or eRRBS) data from a heterogeneous sample must yield relative subpopulation proportions comparable to FACS- or flow-sorted cell fractions, and cell-type composition is unknown a priori.

## When NOT to use

- dxm_estimateFracs is INCOMPATIBLE with dxm_solveMethylation — do not use dxm_estimateFracs to obtain prevalences after a dxm_solveMethylation run; use dxm_solveMethylation's own fractional prevalence solution instead
- The sample has extreme methylation changes (e.g., DNMT3A mutation or demethylating agent treatment) without retuned DXM parameters, or contains regions with allele-specific methylation or copy number variants, which confound subpopulation interpretation
- The data are rich in 5-hydroxymethylation (e.g., neuronal cell types), which DXM does not consider

## Inputs

- Bisulfite sequencing methylation data (e.g., Agilent Methyl-Seq from four DLBCL lymph node biopsies, GEO accession GSE130556) as a tab-delimited BED-like file with columns chromosome, position1, position2, unique regionName, fractional methylation (0-1), integer coverage
- CGI (CpG island) bed file from the UCSC Genome Browser Table Browser (or refGene_hg19_10kb_fixed.txt.gz from the DXM example_data directory) for region-name assignment via bedtools intersect
- Independent FACS/clinical flow cytometry fractions of sorted CD4+ T and CD19+ B cells for validation (optional benchmark)

## Outputs

- Per-sample fractional prevalence estimates of the underlying subpopulations (from dxm_solveMethylation's built-in prevalence solution)
- <prefix>_allVitProb.txt posterior probability file
- <prefix>_reconstructed_1_subpops.txt and <prefix>_reconstructed_2_subpops.txt methylation profile files

## How to apply

Convert methylation calls to a tab-delimited BED-like file (chromosome, position1, position2, unique regionName, fractional methylation 0–1, integer coverage), collapsing across strands and filtering positions below the maximum expected coverage; assign unique region names (e.g., promoter/CGI IDs) via 'bedtools intersect -wo' against a CGI bed. Run 'dxm_solveMethylation -c 400' per sample (~2 hours per 2-subpopulation sample at 60x average coverage) and obtain fractional prevalences from dxm_solveMethylation's own built-in prevalence solution — dxm_estimateFracs is explicitly incompatible with this step. Compare DXM-estimated prevalences against independently determined sorted CD4+ T and CD19+ B cell FACS/flow values, expecting fractional agreement within ~0.0875 (B-cells shown in Figure 6C of the DLBCL validation).

## Related tools

- **DXM** (Deconvolves methylation profiles over user-defined regions via dxm_solveMethylation and provides the fractional prevalence solution for underlying subpopulations) — https://github.com/edwardslab-wustl/dxm
- **bedtools** (Assigns unique region names (e.g., promoter/CGI identifiers) to methylation BED files via 'bedtools intersect -wo' before DXM input)
- **python3** (Runtime for DXM (requires Python 3.5+) with numpy as a required package) — https://github.com/edwardslab-wustl/dxm
- **Docker** (Containerized distribution of DXM for reproducible deconvolution runs) — https://hub.docker.com/r/edwardslab/dxm

## Examples

```
dxm_solveMethylation -i sampleInput.bed -c 400 -o testSample
```

## Evaluation signals

- DXM-estimated fractional prevalences agree with FACS-sorted CD4+ T and CD19+ B cell fractions within ~0.0875 on average
- Per-sample intrasample DMR (i-DMR) counts from dxm_callIDMR fall in the reported 3785–6790 promoter range
- Deconvolved methylation profile accuracies reproduce the reported ~91% for major and ~86% for minor subpopulations
- Input coverage is below the maximum expected sequencing coverage set via the -c flag (e.g., 400), and data were collapsed across strands before running DXM
- Every region carries a unique regionName (e.g., CGI ID) so DXM can compute across all CpGs of each region

## Limitations

- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpreting results in samples with high levels of 5hmC
- Care must be taken in interpreting DXM results in regions with allele-specific methylation or copy number variants
- Additional future testing is required to determine whether a methylation-based strategy using DXM detects cell prevalences more accurately than flow cytometry and FACS
- Retuning DXM parameters (e.g., transition probabilities) may be needed for individual datasets or samples with extreme methylation changes such as DNMT3A mutation or demethylating agent treatment

## Evidence

- [readme] dxm_estimateFracs incompatibility: "Note that this utility is INCOMPATIBLE with dxm_solveMethylation, which has its own fractional prevalence solution call."
- [readme] runtime benchmark: "DXM solved eRRBS and Methyl-Seq samples for 2 subpopulations at average coverage of 60x in ~2 hours."
- [abstract] DLBCL validation: "We further experimentally validate DXM predicted allelic subpopulation-methylation profiles in four Diffuse Large B-Cell Lymphomas (DLBCLs)"
- [discussion] sorted cell validation: "we have validated that subpopulation methylation profiles predicted by DXM are recapitulated in relevant sorted cell subpopulations in DLBCL samples."
- [discussion] flow cytometry comparison caveat: "additional future testing is required to determine whether a methylation-based strategy using DXM detects cell prevalences more accurately than these standard approaches."
