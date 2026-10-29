---
name: two-component-mixture-benchmark-evaluation
description: Use when evaluating or reproducing DXM deconvolution accuracy on simulated
  bisulfite mixtures of two sorted cell types (e.g. GCB–monocyte) at known major/minor
  prevalences over TSS ±5kb promoter regions.
when_to_use_negative:
- Input is a real heterogeneous patient sample with unknown subpopulations and no
  ground truth — this skill is specifically for simulated mixtures with known underlying
  profiles
- 'Regions are very large (e.g. chromosome-scale): DXM requires more CpGs with methylation
  change to solve a distinct profile as regions grow, so benchmarking over whole chromosomes
  is impractical'
- Samples with high 5-hydroxymethylation (e.g. neuronal cell types) or copy-number
  variants, where DXM results need special interpretation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3295
tools:
- name: DXM
  role: dxm_solveMethylation deconvolves each 55x mixture into two subpopulation methylation
    profiles with -c set to the maximum coverage (55)
  repo: https://github.com/edwardslab-wustl/dxm
- name: bedtools
  role: bedtools intersect assigns unique promoter region names by overlapping methylation
    positions with refGene_hg19_10kb_fixed.txt.gz
  repo: https://bedtools.readthedocs.io/en/latest/
- name: Bismark
  role: methylation-extractor .cov output is the starting point converted into the
    DXM BED-like input format
  repo: https://www.bioinformatics.babraham.ac.uk/projects/bismark/
- name: Python3
  role: runtime and environment (3.5+) for DXM, with numpy as a required package
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:806d8db6d38af770057e7c9de6e1fd91a440f78101c70f1b1d1a1b7a162b79bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - Bismark
  - DXM
  - Python3
  - bedtools
  - numpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# two-component-mixture-benchmark-evaluation

## Summary

Benchmarks DXM two-subpopulation methylation deconvolution by building simulated 55x bisulfite read mixtures of sorted GCB and monocyte samples and scoring solved major/minor profiles against the known underlying per-CpG methylation states. Provides ground-truth accuracy tiers (98.5% one-pattern promoters, 96.2% major, 87.1% minor) for validating deconvolution workflows.

## When to use

Use when evaluating or reproducing DXM deconvolution accuracy on simulated bisulfite mixtures of two sorted cell types (e.g. GCB–monocyte) at known major/minor prevalences over TSS ±5kb promoter regions.

## When NOT to use

- Input is a real heterogeneous patient sample with unknown subpopulations and no ground truth — this skill is specifically for simulated mixtures with known underlying profiles
- Regions are very large (e.g. chromosome-scale): DXM requires more CpGs with methylation change to solve a distinct profile as regions grow, so benchmarking over whole chromosomes is impractical
- Samples with high 5-hydroxymethylation (e.g. neuronal cell types) or copy-number variants, where DXM results need special interpretation

## Inputs

- Bisulfite sequencing reads/callsets for sorted GCB and monocyte samples (EGAD00001002442, EGAD00001002523 / GEO GSE130556)
- Bismark methylation-extractor .cov files (chr pos1 pos2 methylation meth_cov unmeth_cov)
- DXM tab-delimited BED-like methylation input: chr, position1, position2, regionName, fractionalMethylation, coverage
- refGene_hg19_10kb_fixed.txt.gz TSS +/-5kb promoter annotation (from dxm example_data, hg19)

## Outputs

- Simulated 55x two-cell-type methylation mixtures at defined major/minor prevalences
- dxm_solveMethylation outputs: <prefix>_reconstructed_1_subpops.txt and <prefix>_reconstructed_2_subpops.txt (minor then major methylation state per CpG)
- benchmark_accuracy.csv with per-CpG accuracy stratified by prevalence tier and one- vs two-subpopulation assignment rates

## How to apply

Retrieve sorted GCB and monocyte bisulfite reads from the Blueprint/BEP accessions EGAD00001002442 and EGAD00001002523 (also GEO GSE130556), convert Bismark methylation-extractor .cov output to the DXM tab-delimited BED-like format (chr, position1, position2, regionName, fractionalMethylation, coverage) with awk, collapsing across strands and filtering coverage to the target maximum. Subsample reads from the two cell types into simulated 55x mixtures at varying major/minor subpopulation prevalences, then restrict CpGs to TSS +/-5kb promoters by overlapping with refGene_hg19_10kb_fixed.txt.gz via bedtools intersect, assigning a unique region name per promoter as DXM requires unique region names for per-region computation. Run dxm_solveMethylation with -c set to the mixture's maximum coverage (55) to solve for two subpopulations, then compare the solved major (reconstructed_2_subpops column 6) and minor (column 5) methylation states against the known per-CpG profiles of the two sorted samples. Score per-CpG accuracy stratified by prevalence tier and the fraction of regions assigned one versus two subpopulation profiles, writing metrics to benchmark_accuracy.csv; correct application reproduces ~98.5% accuracy on one-pattern promoters, ~96.2% on major profiles, and ~87.1% on minor profiles.

## Related tools

- **DXM** (dxm_solveMethylation deconvolves each 55x mixture into two subpopulation methylation profiles with -c set to the maximum coverage (55)) — https://github.com/edwardslab-wustl/dxm
- **bedtools** (bedtools intersect assigns unique promoter region names by overlapping methylation positions with refGene_hg19_10kb_fixed.txt.gz) — https://bedtools.readthedocs.io/en/latest/
- **Bismark** (methylation-extractor .cov output is the starting point converted into the DXM BED-like input format) — https://www.bioinformatics.babraham.ac.uk/projects/bismark/
- **Python3** (runtime and environment (3.5+) for DXM, with numpy as a required package)

## Examples

```
dxm_solveMethylation -i gcb_mono_mix.bed -c 55 -o gcb_mono_mix55
```

## Evaluation signals

- Per-CpG accuracy against known sorted-sample profiles lands near the reported tiers: 98.5% for one-pattern promoters, 96.2% for major profiles, 87.1% for minor profiles
- All input CpG coverages are at or below the -c maximum (55) as required by dxm_solveMethylation
- <prefix>_reconstructed_2_subpops.txt rows have the six-column schema with methylation state of minor subpopulation in column 5 and major in column 6
- Every region name in the DXM input is unique per promoter after bedtools intersect with the TSS +/-5kb annotation
- The fraction of regions assigned two versus one subpopulation profiles matches the known mixture design and is recorded in benchmark_accuracy.csv

## Limitations

- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpretation in samples with high 5hmC such as neuronal cell types
- Care must be taken interpreting DXM results in regions with allele-specific methylation or copy number variants
- As regions get larger and include more CpGs, the number of CpGs that must change methylation to be solved as a distinct profile increases; retuning DXM parameters (e.g. transition probabilities) for genomic compartments or datasets could improve accuracy

## Evidence

- [readme] max coverage flag: "Set the -c flag to the maximum coverage in the input file.  Increasing -c above the maximum coverage will needlessly increase memory usage."
- [readme] output schema: "The format for \<prefix\>_reconstructed_2_subpops.txt is: 1. chromosome 2. position 3. position2 4. region name 5. methylation state of minor subpopulation"
- [methods] strand collapsing: "We generally recommend collapsing data across strands prior to running DXM."
- [supplementary] dataset provenance: "All bisulfite sequencing data has been made available as a GEO accession GSE130556."
- [discussion] region size caveat: "As regions get larger and include more CpGs, the number of CpGs that must have a change in methylation in order to be solved as a distinct methylation profile by DXM will also increase."
