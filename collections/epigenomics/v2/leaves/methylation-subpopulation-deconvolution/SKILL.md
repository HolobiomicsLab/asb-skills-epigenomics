---
name: methylation-subpopulation-deconvolution
description: Use when bisulfite sequencing data (eRRBS, Methyl-Seq) come from a heterogeneous
  sample and you need to infer the number of major subpopulations, their methylation
  profiles, or intrasample DMRs.
when_to_use_negative:
- Samples with high 5-hydroxymethylation (e.g. neuronal cell types) — DXM does not
  consider 5hmC, so results may be misinterpreted
- Regions with allele-specific methylation or copy number variants, where DXM's allelic
  subpopulation model does not apply cleanly
- Samples with extreme methylation changes (e.g. DNMT3A mutation or demethylating
  agent treatment) without retuning DXM parameters such as transition probabilities
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: DXM
  role: 'Primary tool: dxm_solveMethylation deconvolves subpopulation methylation
    profiles; dxm_callIDMR calls intrasample DMRs; dxm_estimateFracs optionally estimates
    subpopulation prevalences'
  repo: https://github.com/edwardslab-wustl/dxm
- name: bedtools
  role: Overlaps methylation BED file with CpG island annotations via 'bedtools intersect
    -wo' to assign unique region names to DXM input
- name: python3
  role: Runtime requirement (3.5+) for DXM; also used with numpy to simulate idealized
    methylation mixtures for benchmarking
  repo: https://github.com/edwardslab-wustl/dxm
- name: numpy
  role: Required Python package for DXM; used in benchmark workflows to generate simulated
    heterogeneous mixtures at fixed prevalences and coverage depths
  repo: https://github.com/edwardslab-wustl/dxm
- name: Docker
  role: Alternative distribution of DXM as a container image at hub.docker.com/r/edwardslab/dxm
  repo: https://hub.docker.com/r/edwardslab/dxm
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:9084a61b51ac3fde036e935517c8152036e7c884924e38a2944a4d0354085c13
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# methylation-subpopulation-deconvolution

## Summary

Deconvolve major allelic subpopulation methylation profiles from heterogeneous bisulfite sequencing data using DXM, which models subclonal methylation over user-defined regions without prior knowledge of subpopulation number or cell types.

## When to use

Use when bisulfite sequencing data (eRRBS, Methyl-Seq) come from a heterogeneous sample and you need to infer the number of major subpopulations, their methylation profiles, or intrasample DMRs.

## When NOT to use

- Samples with high 5-hydroxymethylation (e.g. neuronal cell types) — DXM does not consider 5hmC, so results may be misinterpreted
- Regions with allele-specific methylation or copy number variants, where DXM's allelic subpopulation model does not apply cleanly
- Samples with extreme methylation changes (e.g. DNMT3A mutation or demethylating agent treatment) without retuning DXM parameters such as transition probabilities

## Inputs

- Bisulfite sequencing methylation calls (e.g. bismark_methylation_extractor .cov files) from a heterogeneous sample
- Tab-delimited BED-like methylation file: chromosome, position1, position2, regionName, fractionalMethylation (0-1), integer coverage
- CpG island (cgi.bed) or refGene promoter (refGene_hg19_10kb_fixed.txt.gz) annotation bed for region naming

## Outputs

- <prefix>_reconstructed_1_subpops.txt (regions with one major methylation profile)
- <prefix>_reconstructed_2_subpops.txt (methylation states of minor and major subpopulations per region)
- <prefix>_allVitProb.txt (relative posterior probabilities)
- <prefix>_DXM_dmrs.txt (i-DMR coordinates and region names from dxm_callIDMR)
- <prefix>_solvedPrevalences.txt (optional fractional subpopulation prevalences from dxm_estimateFracs)

## How to apply

Convert each sample's methylation calls (e.g. bismark_methylation_extractor .cov output) into a tab-delimited BED-like file (chromosome, position1, position2, regionName, fractionalMethylation, coverage), collapsing across strands and filtering sites so coverage stays below the maximum expected sequencing coverage. Assign unique region names (e.g. CpG island IDs) via 'bedtools intersect -wo -a methylation.bed -b cgi.bed' with an awk column selection. Run dxm_solveMethylation with -i input.bed, -c set to the maximum coverage in the input (default 400), and an -o output prefix, which produces <prefix>_reconstructed_1_subpops.txt, <prefix>_reconstructed_2_subpops.txt, and <prefix>_allVitProb.txt; optionally run dxm_estimateFracs -k <n> first to estimate subpopulation prevalences. Call intrasample DMRs with dxm_callIDMR -v <prefix>_allVitProb.txt -m <prefix>_reconstructed_2_subpops.txt -o putative, yielding <prefix>_DXM_dmrs.txt. Correct application is judged by solving the expected number of major subpopulations and, in benchmark mixtures, ~96% profile accuracy at >30x coverage.

## Related tools

- **DXM** (Primary tool: dxm_solveMethylation deconvolves subpopulation methylation profiles; dxm_callIDMR calls intrasample DMRs; dxm_estimateFracs optionally estimates subpopulation prevalences) — https://github.com/edwardslab-wustl/dxm
- **bedtools** (Overlaps methylation BED file with CpG island annotations via 'bedtools intersect -wo' to assign unique region names to DXM input)
- **python3** (Runtime requirement (3.5+) for DXM; also used with numpy to simulate idealized methylation mixtures for benchmarking) — https://github.com/edwardslab-wustl/dxm
- **numpy** (Required Python package for DXM; used in benchmark workflows to generate simulated heterogeneous mixtures at fixed prevalences and coverage depths) — https://github.com/edwardslab-wustl/dxm
- **Docker** (Alternative distribution of DXM as a container image at hub.docker.com/r/edwardslab/dxm) — https://hub.docker.com/r/edwardslab/dxm

## Examples

```
dxm_solveMethylation -i sampleInput.bed -c 400 -o testSample && dxm_callIDMR -v testSample_allVitProb.txt -m testSample_reconstructed_2_subpops.txt -o putative
```

## Evaluation signals

- On simulated idealized mixtures at >30x coverage, solved subpopulation profile accuracy should average ~96.2% (HMEC:CD4+T) and ~95.1% (HMEC:HCC1954) against binarized ground-truth references
- Each mixture should resolve the correct number of major subpopulations in <prefix>_reconstructed_1_subpops.txt vs <prefix>_reconstructed_2_subpops.txt
- Input BED file schema check: six tab-delimited columns with fractionalMethylation in [0,1] (divide by 100 if preprocessing tool outputs 0-100) and integer coverage, with unique regionName per region
- The -c flag must equal the maximum coverage in the input file; setting it higher needlessly increases memory usage
- Per-sample i-DMR counts from <prefix>_DXM_dmrs.txt should fall in the reported 3708-18200 range (average 10826) for the 31-DLBCL eRRBS cohort, and i-DMR counts vs age should show a weak correlation (r^2 = 0.19)

## Limitations

- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpreting results in samples with high levels of 5hmC
- Care must be taken in interpreting DXM results in regions with allele-specific methylation or copy number variants
- As regions get larger and include more CpGs, the number of CpGs that must have a methylation change to be solved as a distinct profile increases; solving a global profile across an entire chromosome is impractical given genome gaps and repetitive DNA
- Parameter retuning (e.g. transition probabilities) may be needed for samples with extreme methylation changes such as DNMT3A mutation or demethylating agent treatment
- dxm_estimateFracs output is incompatible with dxm_solveMethylation, which has its own fractional prevalence solution call

## Evidence

- [abstract] we have developed a computational approach, DXM, to deconvolve the methylation profiles of major allelic subpopulations from the bisulfite sequencing data of a heterogeneous sample: "we have developed a computational approach, DXM, to deconvolve the methylation profiles of major allelic subpopulations from the bisulfite sequencing data of a heterogeneous sample"
- [methods] All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation.: "All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation."
- [readme] DXM solved eRRBS and Methyl-Seq samples for 2 subpopulations at average coverage of 60x in ~2 hours.: "DXM solved eRRBS and Methyl-Seq samples for 2 subpopulations at average coverage of 60x in ~2 hours."
- [results] In simulated idealized mixtures, DXM profile accuracy is very high, averaging 96.2% for HMEC-CD4+T cell mixtures and 95.1% for HMEC-HCC1954 cell mixtures when coverage is greater than 30×.: [evidence span withheld — anchor status: not_found]
- [discussion] DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpreting results in samples with high levels of 5hmC: "DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpreting results in samples with high levels of 5hmC"
