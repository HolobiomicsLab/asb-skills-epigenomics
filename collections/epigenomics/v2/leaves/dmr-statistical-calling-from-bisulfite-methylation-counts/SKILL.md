---
name: dmr-statistical-calling-from-bisulfite-methylation-counts
description: Use when comparing DXM i-DMRs from heterogeneous bisulfite data against
  an independently called DMR set between sorted reference cell types, or when a DMR
  caller with explicit count-based statistics is needed on sorted methylation counts.
when_to_use_negative:
- When the sample is homogeneous (single major subpopulation) — DXM will return 1-subpopulation
  reconstructions and there are no i-DMRs to compare against DSS DMRs
- When interpreting DXM results in regions with allele-specific methylation or copy
  number variants — i-DMRs there reflect ASM/CNV, not subpopulation differences, confounding
  the concordance comparison
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DSS
  role: Statistical DMR caller run on sorted GCB vs monocyte methylation counts with
    equal dispersion, delta=0.3, p<0.05
- name: DXM
  role: Deconvolves heterogeneous methylation data (dxm_solveMethylation -c max coverage)
    and calls i-DMRs (dxm_callIDMR) to benchmark against DSS DMRs
  repo: https://github.com/edwardslab-wustl/dxm
- name: bedtools
  role: bedtools intersect to overlap DSS DMR coordinates with DXM i-DMR coordinates
    and compute concordance fractions
- name: python3
  role: Runs DXM (requires python3 3.5+, numpy, cython) to process mixture methylation
    data before comparison
  repo: https://github.com/edwardslab-wustl/dxm
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:ebb0ac48c3a1e2de1dd0b85e88281ecd03ca3855d1e99cb191f1060f98c887c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - DSS
  - DXM
  - bedtools
  - python3
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# dmr-statistical-calling-from-bisulfite-methylation-counts

## Summary

Statistically call differentially methylated regions (DMRs) between sorted cell types from bisulfite methylation counts using DSS (equal dispersion, delta=0.3, p<0.05), then quantify concordance with DXM intrasample DMRs (i-DMRs) via bedtools overlap. Provides an independent caller benchmark for DXM deconvolution (~64% concordance reported).

## When to use

Use when comparing DXM i-DMRs from heterogeneous bisulfite data against an independently called DMR set between sorted reference cell types, or when a DMR caller with explicit count-based statistics is needed on sorted methylation counts.

## When NOT to use

- When the sample is homogeneous (single major subpopulation) — DXM will return 1-subpopulation reconstructions and there are no i-DMRs to compare against DSS DMRs
- When interpreting DXM results in regions with allele-specific methylation or copy number variants — i-DMRs there reflect ASM/CNV, not subpopulation differences, confounding the concordance comparison

## Inputs

- Sorted cell type bisulfite sequencing methylation counts (e.g., sorted GCB and monocyte), strand-collapsed
- Tab-delimited BED-like methylation file: chromosome, position1, position2, regionName, fractionalMethylation (0-1), integer coverage
- DXM outputs: *_allVitProb.txt posterior probabilities and *_reconstructed_2_subpops.txt reconstructed subpopulation profiles

## Outputs

- DSS-called DMR list between the sorted cell types (delta >= 0.3, p-value < 0.05)
- DXM i-DMR file (*_DXM_dmrs.txt: chromosome, start, end, region name)
- Per-region overlap table and summary concordance fractions written to dss_idmr_overlap.tsv (reported ~64% i-DMR/DMR concordance; ~86% recovery of DSS-'missed' DMRs under refined region definitions)

## How to apply

Convert sorted-cell bisulfite sequencing data into the DXM tab-delimited bed-like format (chromosome, position1, position2, regionName, fractionalMethylation 0-1, integer coverage), collapsing across strands as recommended. Run DSS on the sorted GCB vs monocyte methylation counts with equal dispersion enabled, a minimum methylation difference (delta) of 0.3, and a DMR significance threshold of p-value < 0.05 to obtain a DMR list. In parallel, run dxm_solveMethylation (with -c set to the maximum coverage in the input) and dxm_callIDMR on the *_allVitProb.txt and *_reconstructed_2_subpops.txt outputs to produce the *_DXM_dmrs.txt i-DMR file. Intersect DSS DMR coordinates with DXM i-DMR coordinates using bedtools intersect and compute the fraction of i-DMRs overlapping a DSS DMR and the recovery of DSS-'missed' DMRs under refined region definitions, writing the per-region table and summary fractions to dss_idmr_overlap.tsv. The article notes differences between DSS-called DMRs and i-DMRs are of the same magnitude as those between different DMR-callers, so partial overlap — not exact identity — is the expected outcome.

## Related tools

- **DSS** (Statistical DMR caller run on sorted GCB vs monocyte methylation counts with equal dispersion, delta=0.3, p<0.05)
- **DXM** (Deconvolves heterogeneous methylation data (dxm_solveMethylation -c max coverage) and calls i-DMRs (dxm_callIDMR) to benchmark against DSS DMRs) — https://github.com/edwardslab-wustl/dxm
- **bedtools** (bedtools intersect to overlap DSS DMR coordinates with DXM i-DMR coordinates and compute concordance fractions)
- **python3** (Runs DXM (requires python3 3.5+, numpy, cython) to process mixture methylation data before comparison) — https://github.com/edwardslab-wustl/dxm

## Examples

```
bedtools intersect -wo -a dss_dmrs.bed -b dxm_idmrs.bed > dss_idmr_overlap.tsv
```

## Evaluation signals

- Computed i-DMR/DMR concordance should reproduce the reported ~64% agreement between DSS-called DMRs and DXM i-DMRs before accepting the result
- Recovery of ~86% of DSS-'missed' DMRs under refined region definitions should be confirmed
- Every input position has integer coverage below the maximum expected sequencing coverage (the -c/-m flag of dxm_solveMethylation); increasing -c above max coverage only wastes memory
- Fractional methylation values are in [0,1] (divide by 100 if the preprocessor outputs 0-100) and region names are unique per region
- dss_idmr_overlap.tsv contains a per-region overlap table plus summary concordance fractions, with overlaps derived from bedtools intersect of the two coordinate sets

## Limitations

- Only about 64% of DSS-called DMRs were recovered as i-DMRs on average — this overlap is consistent with known variability across DMR callers, so DSS and DXM should not be expected to agree region-for-region
- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpreting results in samples with high levels of 5hmC
- Care must be taken interpreting DXM results in regions with allele-specific methylation or copy number variants
- As regions get larger and include more CpGs, the number of CpGs that must have a change in methylation to be solved as a distinct profile by DXM increases, affecting region-level overlap

## Evidence

- [other] Run DSS on the sorted GCB vs monocyte methylation counts with equal dispersion enabled, a minimum methylation difference (delta) of 0.3, and a DMR significance threshold of p-value < 0.05: [evidence span withheld — anchor status: not_found]
- [discussion] Differences similar to different DMR-callers: "The differences between DSS-called DMRs and i-DMRs in artificial mixtures are similar to those observed between different DMR-callers"
- [other] 64% concordance finding: [evidence span withheld — anchor status: not_found]
- [readme] BED-like input format: "Convert your methylation data to a tab-delimited BED-like file with these columns: chromosome position1 position2 methylation_level coverage."
- [readme] Strand collapsing recommendation: "We generally recommend collapsing data across strands prior to running DXM."
