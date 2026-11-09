---
name: hi-c-trans-interaction-simulation-from-compartment-model
description: Use when validating that a sub-compartment inference method recovers
  known compartment scores from simulated Hi-C trans-interaction data, or when testing
  how low sequencing depth can go before inference degrades.
when_to_use_negative:
- The goal is real biological discovery in a primary Hi-C dataset rather than validating
  or benchmarking sub-compartment inference against a known reference model.
- The analysis targets intra-chromosomal contact maps or TAD/calling-loop tasks —
  CscoreTool-M here is applied to trans-interaction summary data with a Rabl-position
  window model.
- Only a single depth is of interest and no downsampling/re-inference comparison across
  depths is planned.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: CscoreTool-M
  role: Re-infers 5-compartment sub-compartment scores from each simulated trans-interaction
    summary file; run as a single-file Linux executable with arguments <windows_Rabl.bed>
    <input.summary> <OutputPrefix> <N_subcompartments> <N_Rablwindow> <N_session>
  repo: https://github.com/scoutzxb/CscoreTool-M
- name: g++
  role: Fallback compiler for building CscoreTool-M from CscoreTool-M.cpp, twister.h
    and twister.cpp when the prebuilt CentOS binary does not run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:5c85d02ab79399fcdfaa5929a8daf0186eb5ce2a04c7178e7ce045f7efa45898
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  - g++
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# hi-c-trans-interaction-simulation-from-compartment-model

## Summary

Simulates Hi-C trans-interaction summary files by downsampling reads from a GM12878-derived 5-compartment reference model (Rao et al. 2014) at multiple sequencing depths (1x, 0.1x, 0.01x), then re-infers sub-compartment scores with CscoreTool-M to benchmark depth robustness.

## When to use

Use when validating that a sub-compartment inference method recovers known compartment scores from simulated Hi-C trans-interaction data, or when testing how low sequencing depth can go before inference degrades.

## When NOT to use

- The goal is real biological discovery in a primary Hi-C dataset rather than validating or benchmarking sub-compartment inference against a known reference model.
- The analysis targets intra-chromosomal contact maps or TAD/calling-loop tasks — CscoreTool-M here is applied to trans-interaction summary data with a Rabl-position window model.
- Only a single depth is of interest and no downsampling/re-inference comparison across depths is planned.

## Inputs

- High-resolution GM12878 cell Hi-C dataset (Rao, 2014, GEO GSE63525) used to build the 5-compartment reference model
- hg38 500kb equal-length Rabl bed file with fourth column giving relative centromere(0)-telomere(1) position
- Simulated trans-interaction summary file in HOMER HiCsummary format (as used by runHiCpca.pl) at 1x, 0.1x, and 0.01x depth
- Original 5-compartment model compartment scores for correlation comparison

## Outputs

- Simulated downsampling summary files at each depth (1x, 0.1x, 0.01x)
- XXX_cscore.txt per-depth files with re-inferred Cscore for each genomic window
- Genome-wide Pearson correlation values between re-inferred scores and original model scores at each depth

## How to apply

Build the reference model from the high-resolution GM12878 Hi-C dataset (Rao et al. 2014, GSE63525) and simulate trans-interaction summary files at 1x, 0.1x, and 0.01x depth by downsampling randomly selected reads (analogous to the package's 0.5%-selected chr1 test.summary). Prepare the CscoreTool-M executable with chmod +x, falling back to compilation with g++ CscoreTool-M.cpp twister.cpp -fopenmp -O3 if the prebuilt binary fails. For each depth, run ./CscoreTool-M with the hg38 500kb Rabl bed file and the simulated summary file, setting N_subcompartment=5, N_Rablwindow=20, and N_session=12 (adjust N_session to available compute resources). Parse each *_cscore.txt output to extract the re-inferred compartment score per genomic window, then compute genome-wide Pearson correlations against the original 5-compartment model scores. Validation: every genome-wide correlation must exceed 0.9, including on the 0.01x depth dataset.

## Related tools

- **CscoreTool-M** (Re-infers 5-compartment sub-compartment scores from each simulated trans-interaction summary file; run as a single-file Linux executable with arguments <windows_Rabl.bed> <input.summary> <OutputPrefix> <N_subcompartments> <N_Rablwindow> <N_session>) — https://github.com/scoutzxb/CscoreTool-M
- **g++** (Fallback compiler for building CscoreTool-M from CscoreTool-M.cpp, twister.h and twister.cpp when the prebuilt CentOS binary does not run)

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed sim.summary sim_1x 5 20 12
```

## Evaluation signals

- Genome-wide Pearson correlation between re-inferred scores and original 5-compartment model scores exceeds 0.9 at 1x, 0.1x, and 0.01x depth
- Correlation weakens monotonically (or near-monotonically) with lower sequencing depth but stays above the 0.9 threshold
- Each run produces the three expected outputs: XXX_cscore.txt, XXX_cscore.bedgraph, and XXX_rabl.txt
- Cscores in XXX_cscore.bedgraph are normalized to add up to 1 for each genomic region, with regions lacking reads omitted
- Command succeeded with exactly 6 required arguments (bed, summary, prefix, 5, 20, 12); optional 7th/8th arguments (Blacklist.bed, ExcludedInteractions.txt) only if blacklist/translocation filtering was intended

## Limitations

- Correlations between inferred and model scores weaken at lower sequencing depth, though they remained above 0.9 even at 0.01x in the simulation test
- The prebuilt executable was compiled on a CentOS linux machine and may not run on other systems, requiring recompilation from source
- N_session choice depends on the compute resources available; inappropriate settings affect runtime/resource use
- Without Blacklist.bed/ExcludedInteractions.txt arguments, ENCODE blacklist regions (e.g. copy-number-variant loci) and translocation-involved regions are not excluded

## Evidence

- [other] In the simulation test, the inferred compartment scores correlate genome-wide with the model parameters, and although correlations weaken at lower depth, they remain higher than 0.9: [evidence span withheld — anchor status: not_found]
- [readme] The executable is compiled on a CentOS linux machine. If it doesn't work, download the CscoreTool-M.cpp, twister.h and twister.cpp files and put them in the same folder. Then run: "The executable is compiled on a CentOS linux machine. If it doesn't work, download the CscoreTool-M.cpp, twister.h and twister.cpp files and put them in the same folder. Then run"
- [readme] Usage: CscoreTool-M <windows_Rabl.bed> <input.summary> <OutputPrefix> <N_subcompartments> <N_Rablwindow> <N_session> [Blacklist.bed] [ExcludedInteractions.txt]: "Usage: CscoreTool-M <windows_Rabl.bed> <input.summary> <OutputPrefix> <N_subcompartments> <N_Rablwindow> <N_session> [Blacklist.bed] [ExcludedInteractions.txt]"
- [methods] ./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_1 5 20 12: "./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_1 5 20 12"
- [readme] XXX_cscore.bedgraph This is the bedgraph file made for visualization. Ths c-scores are normalized to add up to 1 for each genomic region.: "XXX_cscore.bedgraph This is the bedgraph file made for visualization. Ths c-scores are normalized to add up to 1 for each genomic region."
