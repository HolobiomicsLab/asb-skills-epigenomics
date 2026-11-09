---
name: imprinted-dmr-allele-specific-methylation-calling
description: Use when bisulfite sequencing data (bismark/bsmap/biscuit .cov output)
  must be tested for allele-specific methylation at imprinted DMRs or X-linked CGIs,
  e.g. detecting state-dependent imprinting loss in naive ES cells.
when_to_use_negative:
- Sample has high 5-hydroxymethylation (e.g. neuronal cell types) since DXM does not
  consider 5hmC
- Regions carry copy number variants or complex multi-allelic (more than two) subpopulations
  where a two-subpopulation allelic interpretation is invalid
- 'Single-CpG or very short regions: larger regions need more CpGs with methylation
  change to be solved as a distinct profile, and sparse regions will not deconvolve
  reliably'
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: DXM
  role: Deconvolves methylation profiles of major allelic subpopulations over the
    restricted imprinted DMR regions (dxm_solveMethylation) and calls iDMRs (dxm_callIDMR)
  repo: https://github.com/edwardslab-wustl/dxm
- name: bedtools
  role: Intersects the methylation BED file with the imprinted DMR/CGI bed and formats
    columns into dxm_in.bed via awk
- name: Python3
  role: Runtime for DXM (3.5+); install via 'pip install .' with numpy and cython
    (>=0.24) required
  repo: https://github.com/edwardslab-wustl/dxm
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:35e6e6cc63ae7d4d1962ab574611d0360d90b0d8db821cc1c258ee55d676c9a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - DXM
  - Python3
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# imprinted-dmr-allele-specific-methylation-calling

## Summary

Applies DXM deconvolution to bisulfite sequencing data restricted to known imprinted DMRs (and X-linked CpG islands) to detect allele-specific methylation as two subpopulation methylation profiles, distinguishing states such as primed vs naive ES cells where imprints are lost.

## When to use

Use when bisulfite sequencing data (bismark/bsmap/biscuit .cov output) must be tested for allele-specific methylation at imprinted DMRs or X-linked CGIs, e.g. detecting state-dependent imprinting loss in naive ES cells.

## When NOT to use

- Sample has high 5-hydroxymethylation (e.g. neuronal cell types) since DXM does not consider 5hmC
- Regions carry copy number variants or complex multi-allelic (more than two) subpopulations where a two-subpopulation allelic interpretation is invalid
- Single-CpG or very short regions: larger regions need more CpGs with methylation change to be solved as a distinct profile, and sparse regions will not deconvolve reliably

## Inputs

- Bisulfite sequencing methylation calls (bismark .cov / bsmap / biscuit output) from GEO accession GSE130556 (REP/BEP somatic samples) and GSE75868 (WIBR3 primed vs naive)
- BED file of known imprinted DMRs (or X-linked CpG islands from the UCSC Genome Browser Table Browser, cgi.bed)
- DXM input format: tab-delimited chr, position1, position2, regionName, fractionalMethylation (0-1), integer coverage

## Outputs

- <prefix>_reconstructed_1_subpops.txt and <prefix>_reconstructed_2_subpops.txt from dxm_solveMethylation
- <prefix>_allVitProb.txt posterior probability file
- <prefix>_DXM_dmrs.txt iDMR calls from dxm_callIDMR
- iDMR_rates_summary.tsv tallying fraction of imprinted DMRs solved as two subpopulations per sample/state

## How to apply

Retrieve bisulfite sequencing data (e.g. REP/BEP somatic samples from GEO GSE130556 and WIBR3 primed vs naive from GSE75868) and convert each sample to the DXM BED-like format (chr, position1, position2, regionName, fractionalMethylation 0-1, integer coverage) using the awk conversion recipes for bismark, bsmap, or biscuit. Restrict to imprinted DMRs (or X-linked CGIs) with 'bedtools intersect -wo -a methylation.bed -b dmrs.bed | awk' writing the dxm_in.bed columns chr, pos1, pos2, intersect-region-name, methylation, coverage. Run dxm_solveMethylation -i dxm_in.bed -c <max_coverage_in_input> -o <prefix>, then dxm_callIDMR -v <prefix>_allVitProb.txt -m <prefix>_reconstructed_2_subpops.txt -o <prefix>putative to obtain <prefix>_DXM_dmrs.txt iDMR calls. Set -c to the maximum coverage in the input file (increasing it above max needlessly increases memory), and collapse data across strands before running. Tally the fraction of imprinted DMRs solved as two subpopulations per state; in the source study DXM called two methylation patterns at 59% of imprinted loci in primed WIBR3 ES cells but under 10% after conversion to the naive state, consistent with expected loss of imprints.

## Related tools

- **DXM** (Deconvolves methylation profiles of major allelic subpopulations over the restricted imprinted DMR regions (dxm_solveMethylation) and calls iDMRs (dxm_callIDMR)) — https://github.com/edwardslab-wustl/dxm
- **bedtools** (Intersects the methylation BED file with the imprinted DMR/CGI bed and formats columns into dxm_in.bed via awk)
- **Python3** (Runtime for DXM (3.5+); install via 'pip install .' with numpy and cython (>=0.24) required) — https://github.com/edwardslab-wustl/dxm

## Examples

```
dxm_solveMethylation -i dxm_in.bed -c <max_coverage> -o <prefix> && dxm_callIDMR -v <prefix>_allVitProb.txt -m <prefix>_reconstructed_2_subpops.txt -o <prefix>putative
```

## Evaluation signals

- Fraction of imprinted DMRs called as two subpopulations in primed WIBR3 ES cells falls near the reported ~59%
- Fraction of imprinted loci called in naive WIBR3 falls below ~10%, consistent with expected loss of imprints
- X-linked CGI differential methylation rate is ~92% in female vs ~14% in male samples
- dxm_solveMethylation -c is set to the maximum coverage present in the input file (verified by checking max coverage)
- dxm_in.bed has exactly 6 tab-delimited columns (chr, pos1, pos2, regionName, fractional methylation 0-1, integer coverage) with unique region names per region

## Limitations

- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpreting results in samples with high levels of 5hmC
- Care must be taken in interpreting DXM results in regions with allele-specific methylation or copy number variants
- Retuning DXM parameters (e.g. transition probabilities) for genomic compartments or for individual datasets could improve accuracy
- As regions get larger and include more CpGs, the number of CpGs that must have a methylation change to be solved as a distinct profile increases
- Data should be collapsed across strands prior to running DXM, and coverage filtered below the maximum expected sequencing coverage

## Evidence

- [full_text] DXM detected multiple methylation patterns at 59% of imprinted loci in primed WIBR3 ES cells but at less than 10% of imprinted loci after conversion to the naive state: [evidence span withheld — anchor status: not_found]
- [readme] bedtools intersect recipe: "bedtools intersect -wo -a methylation.bed -b cgi.bed | awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed"
- [readme] coverage flag rationale: "Set the -c flag to the maximum coverage in the input file.  Increasing -c above the maximum coverage will needlessly increase memory usage."
- [readme] strand collapsing and coverage filter recommendation: "All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation.
We generally recommend collapsing data across strands"
- [discussion] 5hmC caveat: "DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpreting results in samples with high levels of 5hmC"
