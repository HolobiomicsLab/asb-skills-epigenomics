---
name: read-length-compatibility-code-patching
description: Use when benchmarking methylPurify against DXM on simulated bisulfite
  sequencing mixtures whose reads exceed 100 bp, since methylPurify's original code
  fails without documented code edits.
when_to_use_negative:
- Input reads are all 100 bp or shorter, so methylPurify runs unmodified and no compatibility
  patching is needed
- Input is raw bisulfite FASTQ rather than processed per-CpG fractional methylation
  with integer coverage in the BED-like format DXM requires
- The goal is applying DXM deconvolution to a new clinical sample (e.g. DLBCL cohort)
  rather than reproducing the methylPurify benchmark comparison
tools:
- name: DXM
  role: Reference deconvolution method; dxm_solveMethylation reconstructs two-subpopulation
    methylation profiles from the mixture BED files for comparison against patched
    methylPurify
  repo: https://github.com/edwardslab-wustl/dxm
- name: methylPurify
  role: Comparator method whose code must be patched to handle reads longer than 100
    bp before running on the same mixture inputs
- name: python3
  role: Runtime environment required to install and run DXM during the benchmark
- name: numpy
  role: Required Python package dependency for DXM used in the benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:5cb620845989181ff9eef384c48f617aec3e21359ff5fbcfcd910159dc7f8eb8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - DXM
  - methylPurify
  - numpy
  - python3
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# read-length-compatibility-code-patching

## Summary

Patch methylPurify's source code to handle bisulfite sequencing reads longer than 100 bp so it can be fairly benchmarked against DXM on simulated heterogeneous methylation mixtures, then evaluate both methods' subpopulation deconvolution accuracy.

## When to use

Use when benchmarking methylPurify against DXM on simulated bisulfite sequencing mixtures whose reads exceed 100 bp, since methylPurify's original code fails without documented code edits.

## When NOT to use

- Input reads are all 100 bp or shorter, so methylPurify runs unmodified and no compatibility patching is needed
- Input is raw bisulfite FASTQ rather than processed per-CpG fractional methylation with integer coverage in the BED-like format DXM requires
- The goal is applying DXM deconvolution to a new clinical sample (e.g. DLBCL cohort) rather than reproducing the methylPurify benchmark comparison

## Inputs

- Simulated 40x GCB:monocyte mixture as tab-delimited BED-like methylation file (chromosome, position1, position2, regionName, fractionalMethylation, coverage)
- Simulated 20x HMEC:HCC1954 35:65 mixture as tab-delimited BED-like methylation file
- Ground-truth subpopulation methylation states for each mixture
- methylPurify source code (prior to read-length patching)

## Outputs

- <prefix>_reconstructed_2_subpops.txt files from dxm_solveMethylation (chr, pos1, pos2, region name, minor subpopulation methylation state, major subpopulation methylation state)
- methylPurify two-subpopulation reconstruction outputs on the same mixture inputs
- Aggregated DXM versus methylPurify accuracy comparison table (major vs minor subpopulation accuracy per mixture)

## How to apply

Prepare both simulated mixtures (20x HMEC:HCC1954 at 35:65 and 40x GCB:monocyte) as tab-delimited BED-like methylation files with columns chromosome, position1, position2, regionName, fractionalMethylation, coverage. Run dxm_solveMethylation on each mixture, setting -c to the maximum input coverage (20 or 40) to avoid needless memory usage, and apply the documented methylPurify code edits required to support reads longer than 100 bp before running it on the same inputs. Compare each method's two-subpopulation reconstructions (the <prefix>_reconstructed_2_subpops.txt files, whose columns report minor then major subpopulation methylation states) against the known ground-truth subpopulation profiles to compute minor-subpopulation accuracy. Accept the reproduction if DXM reaches ~98%/84% minor-subpopulation accuracy while methylPurify stays below 50% in most mixtures, matching the paper's benchmarking finding.

## Related tools

- **DXM** (Reference deconvolution method; dxm_solveMethylation reconstructs two-subpopulation methylation profiles from the mixture BED files for comparison against patched methylPurify) — https://github.com/edwardslab-wustl/dxm
- **methylPurify** (Comparator method whose code must be patched to handle reads longer than 100 bp before running on the same mixture inputs)
- **python3** (Runtime environment required to install and run DXM during the benchmark)
- **numpy** (Required Python package dependency for DXM used in the benchmark)

## Examples

```
dxm_solveMethylation -i gcb_monocyte_mix.bed -c 40 -o gcb_mono
```

## Evaluation signals

- DXM achieves average accuracy of ~98% for major subpopulations and ~84% for minor subpopulations across CGI and promoter windows
- methylPurify's accuracy on minor subpopulations falls below 50% in most mixtures after the read-length patch
- dxm_solveMethylation is invoked with -c matching the mixture's maximum coverage (20 for HMEC:HCC1954, 40 for GCB:monocyte)
- Reconstructed output files contain the six-column two-subpopulation format: chromosome, position, position2, region name, minor subpopulation state, major subpopulation state
- Accuracy is computed against known ground-truth subpopulation methylation states for both mixtures and aggregated into a single comparison table

## Limitations

- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpretation in samples with high 5hmC such as neuronal cell types
- Care must be taken interpreting DXM results in regions with allele-specific methylation or copy number variants
- As regions get larger and include more CpGs, more CpGs with methylation change are needed for DXM to solve a distinct methylation profile; retuning transition probabilities per compartment or dataset could improve accuracy

## Evidence

- [intro] DXM outperformed methylPurify (22), another method developed for subclonal analysis of bisulfite sequencing data: "DXM outperformed methylPurify (22), another method developed for subclonal analysis of bisulfite sequencing data"
- [other] DXM ~98%/84% vs methylPurify <50% accuracy: [evidence span withheld — anchor status: not_found]
- [readme] Coverage flag rationale: "Set the -c flag to the maximum coverage in the input file.  Increasing -c above the maximum coverage will needlessly increase memory usage."
- [readme] Input format specification: "The methylation data input file should be in a tab-delimited bed-like format:"
