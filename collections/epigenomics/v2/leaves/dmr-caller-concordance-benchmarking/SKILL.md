---
name: dmr-caller-concordance-benchmarking
description: Use when you have simulated or known-composition heterogeneous bisulfite
  sequencing mixtures plus methylation calls from the sorted underlying cell types,
  and need to quantify how many independently called DMRs a deconvolution method recovers
  as i-DMRs.
when_to_use_negative:
- The input is a single homogeneous sample with no sorted comparator cell types available
  to call reference DMRs against
- You want to evaluate subpopulation prevalence/fraction estimation accuracy rather
  than DMR-level concordance
- The regions of interest contain allele-specific methylation or copy number variants,
  where DXM results require special care and would confound concordance interpretation
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: DSS
  role: Independent DMR caller run on sorted GCB vs monocyte methylation counts (equal
    dispersion, delta 0.3, p < 0.05) to provide the reference DMR set
- name: DXM
  role: Deconvolves the heterogeneous mixture (dxm_solveMethylation) and calls i-DMRs
    (dxm_callIDMR) to produce the subpopulation DMR set being benchmarked
  repo: https://github.com/edwardslab-wustl/dxm
- name: bedtools
  role: Intersects DSS DMR coordinates with DXM i-DMR coordinates and supports adding
    region names via intersect when building the input bed
- name: python3
  role: Runs the DXM pipeline (requires python3 3.5+, numpy, cython) and computes
    overlap fractions for the summary table
  repo: https://github.com/edwardslab-wustl/dxm
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:2175c3912016b5b2821728eaef0ac30f67726b1fa01ca5f02a2a208a99059f3b
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
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# dmr-caller-concordance-benchmarking

## Summary

Benchmarks DXM intrasample DMR (i-DMR) calls against DMRs called by an independent caller (DSS) on sorted cell types, quantifying overlap with bedtools. This measures how well deconvolved subpopulation methylation profiles recapitulate between-sample differential methylation, at a level of agreement consistent with known variability across DMR callers.

## When to use

Use when you have simulated or known-composition heterogeneous bisulfite sequencing mixtures plus methylation calls from the sorted underlying cell types, and need to quantify how many independently called DMRs a deconvolution method recovers as i-DMRs.

## When NOT to use

- The input is a single homogeneous sample with no sorted comparator cell types available to call reference DMRs against
- You want to evaluate subpopulation prevalence/fraction estimation accuracy rather than DMR-level concordance
- The regions of interest contain allele-specific methylation or copy number variants, where DXM results require special care and would confound concordance interpretation

## Inputs

- Sorted cell-type bisulfite sequencing data (e.g. sorted GCB and monocyte) in methylation count form for DSS
- Heterogeneous mixture bisulfite sequencing data in DXM bed-like format: chromosome, position1, position2, regionName, fractionalMethylation (0-1), integer coverage, collapsed across strands
- DXM outputs from dxm_solveMethylation: *_allVitProb.txt and *_reconstructed_2_subpops.txt
- DSS DMR list called from sorted GCB vs monocyte methylation counts

## Outputs

- DXM i-DMR list (*_DXM_dmrs.txt) with chromosome, start, end, region name
- Per-region DSS/i-DMR overlap table (e.g. dss_idmr_overlap.tsv)
- Summary concordance fractions: fraction of DSS DMRs with an overlapping i-DMR and fraction of DSS 'missed' DMRs recovered under refined region definitions

## How to apply

Convert sorted-cell-type bisulfite data (e.g. GCB vs monocyte) into the tab-delimited bed-like DXM format (chromosome, position1, position2, regionName, fractionalMethylation 0-1, integer coverage), collapsing across strands, then run DSS on the sorted counts with equal dispersion, delta = 0.3, and DMR significance at p-value < 0.05. In parallel, run the DXM pipeline on the heterogeneous mixture: dxm_solveMethylation with -c set to the maximum coverage in the input, then dxm_callIDMR using the *_allVitProb.txt and *_reconstructed_2_subpops.txt outputs to produce the *_DXM_dmrs.txt i-DMR file. Intersect DSS DMR coordinates with DXM i-DMR coordinates using bedtools intersect, computing the fraction of i-DMRs overlapping a DSS DMR and the fraction of DSS 'missed' DMRs recovered under refined region definitions. Write the per-region overlap table and summary concordance fractions (e.g. dss_idmr_overlap.tsv). Interpret the concordance in light of the finding that differences between DSS-called DMRs and i-DMRs are similar to those observed between different DMR-callers, so partial (~64%) overlap is expected rather than a failure mode.

## Related tools

- **DSS** (Independent DMR caller run on sorted GCB vs monocyte methylation counts (equal dispersion, delta 0.3, p < 0.05) to provide the reference DMR set)
- **DXM** (Deconvolves the heterogeneous mixture (dxm_solveMethylation) and calls i-DMRs (dxm_callIDMR) to produce the subpopulation DMR set being benchmarked) — https://github.com/edwardslab-wustl/dxm
- **bedtools** (Intersects DSS DMR coordinates with DXM i-DMR coordinates and supports adding region names via intersect when building the input bed)
- **python3** (Runs the DXM pipeline (requires python3 3.5+, numpy, cython) and computes overlap fractions for the summary table) — https://github.com/edwardslab-wustl/dxm

## Examples

```
dxm_solveMethylation -i mixture.bed -c 400 -o testSample; dxm_callIDMR -v testSample_allVitProb.txt -m testSample_reconstructed_2_subpops.txt -o putative; bedtools intersect -wo -a putative_DXM_dmrs.txt -b dss_dmrs.bed
```

## Evaluation signals

- Computed i-DMR/DMR concordance reproduces the reported ~64% agreement between DSS-called DMRs and DXM i-DMRs
- Recovery of ~86% of DSS-'missed' DMRs under refined region definitions is confirmed before accepting the result
- DSS DMR call uses equal dispersion, delta = 0.3, and p-value < 0.05; deviations in these settings should be flagged
- All methylation values in the DXM input are fractional (0-1, i.e. bisulfite outputs scaled from 0-100 are divided by 100) and coverage values are integers below the -c maximum
- The DXM solveMethylation output contains both *_reconstructed_2_subpops.txt (two-profile regions) and *_allVitProb.txt files required by dxm_callIDMR

## Limitations

- Differences between DSS-called DMRs and DXM i-DMRs are similar in magnitude to differences between different DMR-callers, so concordance is inherently partial and caller-dependent
- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpretation in samples with high 5hmC levels
- Care must be taken interpreting DXM results in regions with allele-specific methylation or copy number variants
- As regions get larger and include more CpGs, more CpGs must show a methylation change for DXM to solve the region as a distinct methylation profile, affecting region-definition-dependent recovery

## Evidence

- [results] DXM on average identified an intrasample DMR (i-DMR) between underlying subpopulations for about 64% of all DMRs called independently between the sorted cell types: [evidence span withheld — anchor status: not_found]
- [results] DSS caller parameters: [evidence span withheld — anchor status: not_found]
- [discussion] The differences between DSS-called DMRs and i-DMRs in artificial mixtures are similar to those observed between different DMR-callers: "The differences between DSS-called DMRs and i-DMRs in artificial mixtures are similar to those observed between different DMR-callers (62)."
- [readme] README input format: "The methylation data input file should be in a tab-delimited bed-like format:

<chr> <position1> <position2> <regionName> <fractionalMethylation> <coverage>"
- [readme] README dxm_callIDMR output: "The output of dxm_callIDMR is <prefix>_DXM_dmrs.txt (e.g. putative_DXMdmrs.txt). Its format is tab-delimited:
	1. chromosome
	2. start coordinate
	3. end coordinate
	4. region name"
