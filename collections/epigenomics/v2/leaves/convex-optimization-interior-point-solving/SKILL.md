---
name: convex-optimization-interior-point-solving
description: Use when fitting a probabilistic mixture/sub-compartment model to Hi-C
  interaction data where the joint likelihood is non-convex but separates into convex
  blocks (per-compartment scores Ci on each chromosome, plus alpha/beta Rabl-effect
  parameters) amenable to block-coordinate.
when_to_use_negative:
- The data are bulk compartment calls (A/B) rather than probabilistic multi-sub-compartment
  inference — PCA-based tools like HOMER runHiCpca.pl are the appropriate comparison
- 'You only need to delete problematic interactions from the input: ExcludedInteractions
  filtering must also remove their expectations, so simply editing the HiCsummary
  file is not equivalent and this skill''s exclusion logic should be used instead'
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3930
tools:
- name: CscoreTool-M
  role: Implements the alternating per-chromosome Ci interior-point solves and alpha/beta
    Newton updates over multiple random-start sessions; invoked as a compiled C++
    executable (built from CscoreTool-M.cpp, twister.h, twister.cpp with g++ -fopenmp
    -O3).
  repo: https://github.com/scoutzxb/CscoreTool-M
- name: g++
  role: Compiles the single-source optimizer (with OpenMP) when the prebuilt CentOS
    binary does not run.
- name: HOMER
  role: Defines the HiCsummary input format compatibility for the interaction file
    fed to the optimizer.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:4df0264a3dd9dcbb3e1874d8430dc2d17ca4fb349344af66392cbf7b098b5cda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  - HOMER
  - g++
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3930
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# convex-optimization-interior-point-solving

## Summary

A method-rich skill for solving the non-convex maximum-likelihood estimation of 3D genome sub-compartment parameters (Ci, alpha, beta) in CscoreTool-M by decomposing it into per-chromosome convex sub-problems solved with a primal-dual interior-point method and Newton inner iterations, alternating parameter blocks until the likelihood converges to a local maximum.

## When to use

Use when fitting a probabilistic mixture/sub-compartment model to Hi-C interaction data where the joint likelihood is non-convex but separates into convex blocks (per-compartment scores Ci on each chromosome, plus alpha/beta Rabl-effect parameters) amenable to block-coordinate interior-point/Newton optimization.

## When NOT to use

- The data are bulk compartment calls (A/B) rather than probabilistic multi-sub-compartment inference — PCA-based tools like HOMER runHiCpca.pl are the appropriate comparison
- You only need to delete problematic interactions from the input: ExcludedInteractions filtering must also remove their expectations, so simply editing the HiCsummary file is not equivalent and this skill's exclusion logic should be used instead

## Inputs

- Equal-length genomic window bed file with Rabl position (0=centromere, 1=telomere) in column 4 (e.g. hg38_500k_Rabl.bed)
- Hi-C summary file in HOMER runHiCpca.pl HiCsummary format (e.g. Test.summary from 0.5% of GM12878 chr1 reads, Rao 2014)
- Integer parameters: N_subcompartment (e.g. 5), N_Rablwindow (e.g. 20), N_session (e.g. 12)
- Optional Blacklist.bed of ENCODE blacklist regions
- Optional ExcludedInteractions.txt listing translocation region sets (chrName_chrStart_chrEnd groups per line)

## Outputs

- cscore.txt: Cscore per genomic window, interpretable as the probability of the region belonging to each sub-compartment
- cscore.bedgraph: bedgraph for visualization with c-scores normalized to sum to 1 per genomic region (regions with no reads omitted)
- rabl.txt: estimated Rabl effect as a matrix (N_Rablwindow x N_Rablwindow)

## How to apply

Initialize the sub-compartment model parameters from multiple random starts, running N_session (default 12) parallel sessions and keeping the best likelihood. Alternate two optimization blocks: (1) solve the per-chromosome convex sub-problems for the compartment probabilities Ci using a primal-dual interior-point method with Newton search as the fixed inner solver, and (2) update the global alpha and beta (Rabl effect) parameters via Newton updates. Because each block optimization only increases the likelihood, iterate until convergence to a local maximum, restarting from new random initializations if needed. Model five sub-compartments (N_subcompartment=5) over a 20-bin Rabl matrix (N_Rablwindow=20) computed from equal-length genomic windows whose fourth bed column gives the Rabl position from centromere (0) to telomere (1); interactions touching ENCODE blacklist regions and, if supplied, translocation region pairs in ExcludedInteractions.txt must be removed along with their expectations before fitting.

## Related tools

- **CscoreTool-M** (Implements the alternating per-chromosome Ci interior-point solves and alpha/beta Newton updates over multiple random-start sessions; invoked as a compiled C++ executable (built from CscoreTool-M.cpp, twister.h, twister.cpp with g++ -fopenmp -O3).) — https://github.com/scoutzxb/CscoreTool-M
- **g++** (Compiles the single-source optimizer (with OpenMP) when the prebuilt CentOS binary does not run.)
- **HOMER** (Defines the HiCsummary input format compatibility for the interaction file fed to the optimizer.)

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_3 5 20 12 hg38_blacklisted.bed Test_excluded.txt
```

## Evaluation signals

- All three output files (cscore.txt, cscore.bedgraph, rabl.txt) exist under the output prefix
- cscore.bedgraph c-scores are interpretable as probabilities and normalize to sum to 1 per genomic region, with regions having no reads omitted
- rabl.txt is emitted as a matrix of the estimated Rabl effect sized by N_Rablwindow (e.g. 20)
- The objective (log-likelihood) is monotonically non-decreasing across block-optimization iterations before convergence to a local maximum
- Interactions involving ENCODE blacklist regions and ExcludedInteractions translocation pairs are absent from the fit, including their expectations

## Limitations

- The alternating scheme converges only to a local maximum, hence the need for N_session random-restart sessions; the number of sessions should be chosen according to available resources
- Blacklisted regions (possibly due to copy number variations) and translocation regions must be excluded explicitly; failure to remove their expectations biases the likelihood
- The prebuilt executable was compiled on CentOS and may not run on other systems, requiring recompilation with g++ -fopenmp -O3

## Evidence

- [other] The algorithm works by sequentially optimizing Ci on each chromosome, and then optimizing alpha and beta; by iterating this procedure, the likelihood function only increases each time, converging to a local maximum.: [evidence span withheld — anchor status: not_found]
- [readme] Usage: CscoreTool-M <windows_Rabl.bed> <input.summary> <OutputPrefix> <N_subcompartments> <N_Rablwindow> <N_session> [Blacklist.bed] [ExcludedInteractions.txt]: "Usage: CscoreTool-M <windows_Rabl.bed> <input.summary> <OutputPrefix> <N_subcompartments> <N_Rablwindow> <N_session> [Blacklist.bed] [ExcludedInteractions.txt]"
- [readme] Note that the expectation are also excluded in the calculation, so it's different from simply delete the interactions from the input file.: "Note that the expectation are also excluded in the calculation, so it's different from simply delete the interactions from the input file."
- [intro] g++ CscoreTool-M.cpp twister.cpp -fopenmp -O3 -o CscoreTool-M: "g++ CscoreTool-M.cpp twister.cpp -fopenmp -O3 -o CscoreTool-M"
- [readme] It should be an equal-length bed file with the fourth column being the Rabl position, i.e. the relative position between centromere (0) and telomere (1).: "It should be an equal-length bed file with the fourth column being the Rabl position, i.e. the relative position between centromere (0) and telomere (1)."
