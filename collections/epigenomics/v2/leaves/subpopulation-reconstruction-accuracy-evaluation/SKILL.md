---
name: subpopulation-reconstruction-accuracy-evaluation
description: Use when evaluating how accurately a subpopulation methylation deconvolution
  method (e.g. DXM, methylPurify) reconstructs known minor/major subpopulation methylation
  states from simulated bisulfite sequencing mixtures.
when_to_use_negative:
- The sample's true subpopulations are unknown — accuracy against ground truth cannot
  be computed without simulated or sorted reference profiles
- The goal is calling iDMRs or estimating subpopulation prevalences rather than scoring
  reconstruction accuracy (use dxm_callIDMR or dxm_estimateFracs instead)
- Input contains high 5-hydroxymethylation (e.g. neuronal samples) — DXM does not
  model 5hmC, so accuracy comparisons would be confounded
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: DXM
  role: Deconvolves subpopulation methylation profiles from bisulfite sequencing mixtures
    via dxm_solveMethylation for the benchmark arm
  repo: https://github.com/edwardslab-wustl/dxm
- name: methylPurify
  role: Comparison method for subclonal methylation analysis, run on the same mixture
    inputs after code edits for reads longer than 100 bp
- name: python3
  role: Runtime and scripting environment for DXM (3.5+) and for computing reconstruction
    accuracies against ground truth
- name: numpy
  role: Required Python package underlying DXM's computations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:037ca00483b4d2ad5e50b4e7aba26f0ecad5e714cd33daa185ae8aee81b02aee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - DXM
  - methylPurify
  - numpy
  - python3
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# subpopulation-reconstruction-accuracy-evaluation

## Summary

Benchmarking skill for comparing deconvolution methods (DXM vs methylPurify) on simulated heterogeneous bisulfite sequencing mixtures, by scoring reconstructed two-subpopulation methylation profiles against known ground truth in CGI and promoter windows.

## When to use

Use when evaluating how accurately a subpopulation methylation deconvolution method (e.g. DXM, methylPurify) reconstructs known minor/major subpopulation methylation states from simulated bisulfite sequencing mixtures.

## When NOT to use

- The sample's true subpopulations are unknown — accuracy against ground truth cannot be computed without simulated or sorted reference profiles
- The goal is calling iDMRs or estimating subpopulation prevalences rather than scoring reconstruction accuracy (use dxm_callIDMR or dxm_estimateFracs instead)
- Input contains high 5-hydroxymethylation (e.g. neuronal samples) — DXM does not model 5hmC, so accuracy comparisons would be confounded

## Inputs

- Simulated heterogeneous mixture methylation data in tab-delimited BED-like format: chromosome, position1, position2, regionName, fractionalMethylation (0-1), coverage (integer)
- Known ground-truth subpopulation methylation states for each region
- Region definitions (e.g. UCSC CGI bed file or refGene_hg19_10kb_fixed.txt.gz TSS +/- 5kb windows)

## Outputs

- DXM <prefix>_reconstructed_2_subpops.txt files with minor and major subpopulation methylation states per region
- Per-method minor- and major-subpopulation reconstruction accuracies for CGI and promoter windows
- Aggregated comparison table of DXM versus methylPurify accuracy across mixtures

## How to apply

Prepare simulated mixtures (e.g. 20x HMEC:HCC1954 at 35:65 and 40x GCB:monocyte) as tab-delimited BED-like files with columns chromosome, position1, position2, regionName, fractionalMethylation, coverage. Run dxm_solveMethylation on each mixture with -i set to the input BED file and -c set to the mixture's maximum coverage (20 or 40) to avoid needless memory usage; run methylPurify on the same inputs after applying the documented code edits needed to support reads longer than 100 bp. Score each method's reconstructed two-subpopulation profiles from the <prefix>_reconstructed_2_subpops.txt outputs (columns: chr, pos1, pos2, region name, minor subpopulation methylation state, major subpopulation methylation state) against ground-truth subpopulation methylation states, computing minor-subpopulation reconstruction accuracy. Aggregate accuracies across CGI and promoter windows; the reported benchmark found DXM reached ~98% accuracy for major and ~84% for minor subpopulations while methylPurify fell below 50% on most minor subpopulations, so accept the comparison only if DXM reproduces this advantage.

## Related tools

- **DXM** (Deconvolves subpopulation methylation profiles from bisulfite sequencing mixtures via dxm_solveMethylation for the benchmark arm) — https://github.com/edwardslab-wustl/dxm
- **methylPurify** (Comparison method for subclonal methylation analysis, run on the same mixture inputs after code edits for reads longer than 100 bp)
- **python3** (Runtime and scripting environment for DXM (3.5+) and for computing reconstruction accuracies against ground truth)
- **numpy** (Required Python package underlying DXM's computations)

## Examples

```
dxm_solveMethylation -i gcb_monocyte_40x.bed -c 40 -o gcbMonocyte
```

## Evaluation signals

- DXM reconstructs major subpopulations at ~98% average accuracy and minor subpopulations at ~84% across CGI and promoter windows on the 40x GCB:monocyte mixture
- methylPurify minor-subpopulation accuracy remains below 50% in most mixtures, consistent with the paper's benchmarking finding
- Reconstructed output files follow the documented schema: chromosome, position, position2, region name, minor subpopulation state, major subpopulation state
- -c flag of dxm_solveMethylation equals the maximum coverage of the input (20 for HMEC:HCC1954, 40 for GCB:monocyte) to avoid needless memory usage
- Coverage values in the input BED file are filtered below the maximum expected sequencing coverage before solving

## Limitations

- methylPurify requires documented code edits to handle reads longer than 100 bp, which must be applied before fair benchmarking
- DXM performance depends on region size: larger regions with more CpGs require more CpGs with methylation change to be solved as a distinct profile
- Interpretation of reconstruction results needs care in regions with allele-specific methylation or copy number variants
- Retuning DXM parameters (e.g. transition probabilities) for genomic compartments or datasets could improve accuracy

## Evidence

- [results] DXM achieved average accuracy of ~98% for major subpopulations and ~84% for minor subpopulations across CGI and promoter windows, whereas methylPurify's performance on minor subpopulations fell below 50% in most mixtures.: [evidence span withheld — anchor status: not_found]
- [readme] Run dxm_solveMethylation with -c set to maximum coverage.: "Set the -c flag to the maximum coverage in the input file.  Increasing -c above the maximum coverage will needlessly increase memory usage."
- [readme] Reconstructed two-subpopulations output format.: "The format for <prefix>_reconstructed_2_subpops.txt is: 1. chromosome 2. position 3. position2 4. region name 5. methylation state of minor subpopulation 6. methylation state of major subpopulation"
- [methods] Coverage filtering requirement before running DXM.: "All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation."
- [intro] Benchmark finding that DXM outperforms methylPurify on simulated mixtures.: "We benchmarked DXM on a wide set of simulated mixtures using bisulfite sequencing reads from sorted hematopoietic cell types and found that DXM outperformed methylPurify"
