---
name: simulation-based-method-benchmarking
description: Use when a paper claims a deconvolution or inference method outperforms
  a competitor and you must reproduce the comparison on simulated mixtures with known
  ground-truth subpopulation profiles, such as 40x GCB-monocyte bisulfite data.
when_to_use_negative:
- The data are real patient samples with unknown ground-truth subpopulations, so reconstruction
  accuracy cannot be computed against a gold standard
- The competitor method cannot be run on the input (e.g. methylPurify cannot handle
  reads longer than 100 bp without the documented code edits, so the comparison would
  be invalid)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: DXM
  role: Deconvolves subpopulation methylation profiles from simulated mixture inputs
    via dxm_solveMethylation for benchmarking against methylPurify
  repo: https://github.com/edwardslab-wustl/dxm
- name: methylPurify
  role: Competing subclonal methylation analysis method run on the same simulated
    mixtures with code edits for reads longer than 100 bp
- name: python3
  role: Runtime environment (3.5+) for DXM and accuracy-computation scripts
- name: numpy
  role: Required Python package for DXM and downstream accuracy aggregation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:ab41c658ab31f46e420ea51d93cc438c3249dcfdc8d35c036e7f4ffde5e2a012
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# simulation-based-method-benchmarking

## Summary

Benchmark a deconvolution method (DXM) against a competing method (methylPurify) on simulated bisulfite sequencing mixtures with known ground-truth subpopulation methylation states, quantifying minor-subpopulation reconstruction accuracy to support claims of improved performance.

## When to use

Use when a paper claims a deconvolution or inference method outperforms a competitor and you must reproduce the comparison on simulated mixtures with known ground-truth subpopulation profiles, such as 40x GCB-monocyte bisulfite data.

## When NOT to use

- The data are real patient samples with unknown ground-truth subpopulations, so reconstruction accuracy cannot be computed against a gold standard
- The competitor method cannot be run on the input (e.g. methylPurify cannot handle reads longer than 100 bp without the documented code edits, so the comparison would be invalid)

## Inputs

- Simulated bisulfite sequencing mixture methylation data (e.g. 20x HMEC:HCC1954 35:65 and 40x GCB:monocyte) as tab-delimited BED-like files: chr, position1, position2, regionName, fractionalMethylation, coverage
- Known ground-truth subpopulation methylation states for each simulated mixture
- methylPurify code with documented edits to handle reads longer than 100 bp

## Outputs

- DXM reconstructed two-subpopulation profiles (<prefix>_reconstructed_2_subpops.txt)
- Minor-subpopulation reconstruction accuracy values for DXM and methylPurify per mixture
- Aggregated comparison table of accuracies across both mixtures

## How to apply

Prepare simulated mixture datasets (e.g. 20x HMEC:HCC1954 at 35:65 and 40x GCB:monocyte) as tab-delimited BED-like methylation files with columns chromosome, position1, position2, regionName, fractionalMethylation, coverage. Run dxm_solveMethylation on each mixture with -i set to the mixture BED file and -c set to the maximum input coverage (20 or 40) to avoid needless memory usage, then run methylPurify on the same inputs with the documented code edits required to support reads longer than 100 bp. Compare each method's two-subpopulation reconstructions (<prefix>_reconstructed_2_subpops.txt, which report minor then major subpopulation methylation state) against the ground-truth subpopulation states to compute minor-subpopulation accuracy, then aggregate both methods' accuracies into a comparison table.

## Related tools

- **DXM** (Deconvolves subpopulation methylation profiles from simulated mixture inputs via dxm_solveMethylation for benchmarking against methylPurify) — https://github.com/edwardslab-wustl/dxm
- **methylPurify** (Competing subclonal methylation analysis method run on the same simulated mixtures with code edits for reads longer than 100 bp)
- **python3** (Runtime environment (3.5+) for DXM and accuracy-computation scripts)
- **numpy** (Required Python package for DXM and downstream accuracy aggregation)

## Examples

```
dxm_solveMethylation -i gcb_monocyte_40x.bed -c 40 -o gcb_monocyte_40x
```

## Evaluation signals

- DXM achieves average accuracy of ~98% for major subpopulations and ~84% for minor subpopulations across CGI and promoter windows on the simulated mixtures
- methylPurify's minor-subpopulation performance falls below 50% in most mixtures, matching the paper's reported underperformance
- Output files <prefix>_reconstructed_2_subpops.txt exist with the expected tab-delimited schema (chr, pos1, pos2, region name, minor subpopulation state, major subpopulation state)
- -c flag of dxm_solveMethylation matches the maximum coverage of the input file (20 or 40) so memory is not needlessly inflated
- Reproduction is accepted only if DXM accuracies reach the reported ~98%/~84% while methylPurify stays below 50%

## Limitations

- methylPurify requires documented code edits to support reads longer than 100 bp, so the comparison is conditional on those modifications
- Simulated-mixture accuracy may not transfer to real heterogeneous samples; DXM's DLBCL patient findings (18 patients) needed validation in a larger cohort
- DXM computational time scales with number of subpopulations, number of CpGs, and maximum coverage, and setting -c above the maximum input coverage needlessly increases memory usage

## Evidence

- [other] DXM achieved average accuracy of ~98% for major subpopulations and ~84% for minor subpopulations across CGI and promoter windows, whereas methylPurify's performance on minor subpopulations fell below 50% in most mixtures.: [evidence span withheld — anchor status: not_found]
- [readme] README benchmarking-relevant output format: "The format for \<prefix\>_reconstructed_2_subpops.txt is: 1. chromosome 2. position 3. position2 4. region name 5. methylation state of minor subpopulation 6. methylation state of major subpopulation"
- [readme] README coverage parameter guidance: "Set the -c flag to the maximum coverage in the input file.  Increasing -c above the maximum coverage will needlessly increase memory usage."
- [intro] DXM benchmarking claim against methylPurify: "We benchmarked DXM on a wide set of simulated mixtures using bisulfite sequencing reads from sorted hematopoietic cell types and found that DXM outperformed methylPurify"
- [methods] README input format requirement: "Convert your methylation data to a tab-delimited BED-like file with these columns: chromosome position1 position2 methylation_level coverage."
