---
name: flow-cytometry-prevalence-comparison
description: Use when methylation-based deconvolution prevalence estimates (e.g. from
  DXM) must be experimentally validated against independently sorted cell populations
  such as FACS-derived CD4+ T and CD19+ B cell fractions.
when_to_use_negative:
- Prevalence estimates come from dxm_estimateFracs, which the README states is INCOMPATIBLE
  with dxm_solveMethylation and its own prevalence solution
- The sample lacks matched FACS or flow cytometry measurements of sorted cell subpopulations,
  leaving no independent ground truth
- The goal is assessing whether DXM outperforms flow cytometry in accuracy — the article
  says additional future testing is required for that claim, so this comparison supports
  only agreement, not superiority
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_2640
tools:
- name: DXM
  role: Deconvolves bisulfite sequencing methylation data via dxm_solveMethylation
    to obtain subpopulation fractional prevalences for comparison against FACS values
  repo: https://github.com/edwardslab-wustl/dxm
- name: Docker
  role: Alternative deployment route for the DXM container used to run the deconvolution
    on the DLBCL samples
  repo: https://hub.docker.com/repository/docker/edwardslab/dxm
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:1439ba711ea8e40752dafb50e9d84b138ee40823c52507fadb90b434e70e502c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - DXM
  - Docker
  - numpy
  - python3
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# flow-cytometry-prevalence-comparison

## Summary

Validates DXM-solved subpopulation fractional prevalences from bisulfite sequencing data against independently measured cell proportions from FACS/clinical flow cytometry (sorted CD4+ T and CD19+ B cells) in primary DLBCL lymph node biopsies.

## When to use

Use when methylation-based deconvolution prevalence estimates (e.g. from DXM) must be experimentally validated against independently sorted cell populations such as FACS-derived CD4+ T and CD19+ B cell fractions.

## When NOT to use

- Prevalence estimates come from dxm_estimateFracs, which the README states is INCOMPATIBLE with dxm_solveMethylation and its own prevalence solution
- The sample lacks matched FACS or flow cytometry measurements of sorted cell subpopulations, leaving no independent ground truth
- The goal is assessing whether DXM outperforms flow cytometry in accuracy — the article says additional future testing is required for that claim, so this comparison supports only agreement, not superiority

## Inputs

- DXM-solved fractional prevalences from dxm_solveMethylation output (allVitProb and reconstructed subpopulation files) for four DLBCL lymph node biopsies (GSE130556)
- FACS/clinical flow cytometry cell proportions for sorted CD4+ T cells and CD19+ B cells from the same DLBCL samples

## Outputs

- Per-sample comparison of DXM-estimated vs FACS-measured T and B-cell fractional prevalences (average fractional difference within ~0.0875)
- Validation verdict on DXM's ability to recover subpopulation composition in primary DLBCL samples

## How to apply

For each DLBCL sample, run dxm_solveMethylation and take the fractional prevalences of the underlying subpopulations from DXM's built-in prevalence solution within dxm_solveMethylation (dxm_estimateFracs is incompatible with this step). Then compare the DXM-estimated T and B-cell prevalences against the sorted CD4+ T and CD19+ B cell flow/FACS values for the four samples, requiring fractional agreement within ~0.0875. Corroborate the validation by checking that per-sample i-DMR counts fall in the reported 3785-6790 promoter range and that deconvolved profile accuracies are approximately 91% for major and 86% for minor subpopulations. Note that the article cautions additional future testing is required to determine whether the methylation-based strategy detects cell prevalences more accurately than flow cytometry and FACS themselves.

## Related tools

- **DXM** (Deconvolves bisulfite sequencing methylation data via dxm_solveMethylation to obtain subpopulation fractional prevalences for comparison against FACS values) — https://github.com/edwardslab-wustl/dxm
- **Docker** (Alternative deployment route for the DXM container used to run the deconvolution on the DLBCL samples) — https://hub.docker.com/repository/docker/edwardslab/dxm

## Examples

```
dxm_solveMethylation -i sample.bed -c 400 -o dlbc_sample; dxm_callIDMR -v dlbc_sample_allVitProb.txt -m dlbc_sample_reconstructed_2_subpops.txt -o putative
```

## Evaluation signals

- DXM-estimated and FACS-sorted fractional prevalences agree within ~0.0875 on average across the four DLBCL samples (B-cells shown in Figure 6C)
- Per-sample i-DMR counts fall in the reported 3785-6790 promoter range
- Deconvolved profile accuracies reproduce ~91% for major and ~86% for minor subpopulations
- Prevalences are sourced from dxm_solveMethylation's built-in prevalence solution, not dxm_estimateFracs

## Limitations

- Additional future testing is required to determine whether a methylation-based strategy using DXM detects cell prevalences more accurately than flow cytometry and FACS
- Initial findings based on 18 patients need to be validated in a larger cohort
- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpreting results in samples with high levels of 5hmC

## Evidence

- [results] DXM-estimated prevalences agreed well with FACS results: [evidence span withheld — anchor status: not_found]
- [readme] built-in prevalence solution, not dxm_estimateFracs: "Note that this utility is INCOMPATIBLE with dxm_solveMethylation, which has its own fractional prevalence solution call."
- [intro] validation in sorted CD4+ T and CD19+ B cells: "We validated that DXM predictions for subpopulation methylation profiles were recapitulated in relevant sorted CD4+ T and CD19+ B cells from these samples."
- [discussion] methylation strategy vs flow cytometry accuracy: "additional future testing is required to determine whether a methylation-based strategy using DXM detects cell prevalences more accurately than these standard approaches."
