---
name: compartment-score-parameter-correlation
description: Use when a Hi-C dataset (real or simulated from known models) has been
  deconvolved by CscoreTool-M and each inferred sub-compartment must be assigned to,
  or validated against, a reference cell-type or model parameter set.
when_to_use_negative:
- Input is a single homogeneous cell-type Hi-C dataset with no reference model parameters
  to correlate against — the correlation step has nothing to validate against
- You need de novo sub-compartment inference itself rather than validation of already-inferred
  sub-compartments (run the CscoreTool-M executable first)
- The Hi-C interactions have not been processed by CscoreTool-M (no _cscore.txt output
  exists)
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3308
tools:
- name: CscoreTool-M
  role: Probabilistic-modeling tool that infers multiple 3D genome sub-compartments
    from Hi-C summary data; its per-window Cscore output is the profile correlated
    against reference model parameters.
  repo: https://github.com/scoutzxb/CscoreTool-M
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:63928832d09283cae45501a9732e89963bfc4331d427961a9de0192ed4b18dfc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# compartment-score-parameter-correlation

## Summary

Validates CscoreTool-M sub-compartment inference by correlating each inferred sub-compartment's Cscore profile (from XXX_cscore.txt) against the reference model parameters used to generate or characterize the data, e.g. separating GM12878-like from IMR90-like sub-compartments in a mixed-population simulation.

## When to use

Use when a Hi-C dataset (real or simulated from known models) has been deconvolved by CscoreTool-M and each inferred sub-compartment must be assigned to, or validated against, a reference cell-type or model parameter set.

## When NOT to use

- Input is a single homogeneous cell-type Hi-C dataset with no reference model parameters to correlate against — the correlation step has nothing to validate against
- You need de novo sub-compartment inference itself rather than validation of already-inferred sub-compartments (run the CscoreTool-M executable first)
- The Hi-C interactions have not been processed by CscoreTool-M (no _cscore.txt output exists)

## Inputs

- CscoreTool-M output file <prefix>_cscore.txt (Cscore per genomic window)
- Optional <prefix>_cscore.bedgraph (normalized c-scores summing to 1 per region)
- Reference model parameters used to simulate or characterize each cell type (e.g. GM12878- and IMR90-derived parameters from depth-matched Hi-C summary datasets)

## Outputs

- Correlation of each inferred sub-compartment profile against each reference model parameter set
- Assignment of sub-compartments to reference cell types (e.g. Sim1–Sim5 → GM12878-like; Sim6–Sim10 → IMR90-like)

## How to apply

First run CscoreTool-M to infer sub-compartments (e.g. N_subcompartment=10, genomic window size 5, N_Rablwindow=20, N_session=12 parallel sessions) and parse the XXX_cscore.txt output, which holds the Cscore estimated for each genomic window; optionally inspect XXX_cscore.bedgraph where c-scores are normalized to add up to 1 per genomic region. Next, correlate each sub-compartment's score profile against the model parameters used in the simulation (e.g. GM12878- and IMR90-derived parameters after depth-matching the two Hi-C summaries before merging). Finally, validate by confirming the inferred sub-compartments partition cleanly according to the reference models — in the mixed-population simulation, exactly five of the 10 sub-compartments (Sim1–Sim5) resembled GM12878 and five (Sim6–Sim10) resembled IMR90. This works because CscoreTool-M compartment scores represent the probability of a genomic region locating in a specific sub-compartment, so profiles track the parameters that generated the data.

## Related tools

- **CscoreTool-M** (Probabilistic-modeling tool that infers multiple 3D genome sub-compartments from Hi-C summary data; its per-window Cscore output is the profile correlated against reference model parameters.) — https://github.com/scoutzxb/CscoreTool-M

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_1 5 20 12
```

## Evaluation signals

- The N inferred sub-compartments separate into the expected number of groups per reference model (e.g. exactly 5 of 10 sub-compartments GM12878-like and 5 IMR90-like by correlation sign/magnitude)
- Cscore profiles in _cscore.bedgraph are normalized to add up to 1 for each genomic region
- Each genomic window in _cscore.txt carries a Cscore interpretable as the probability of the region locating in a specific sub-compartment
- Inferred sub-compartment profiles reproduce the relative position patterns (Rabl effect from _rabl.txt) consistent with the model parameters used in simulation
- Sub-compartments assigned to each cell type show expected functional enrichment (housekeeping vs cell-type-specific genes) per the paper's findings

## Limitations

- Validation depends on having a simulated or known reference: the correlation-to-parameters check only works when model parameters (e.g. depth-matched GM12878/IMR90 summaries) are available
- Proliferating cells have higher genome organization heterogeneity than terminally differentiated non-proliferating cells, likely due to cell-cycle stage, which can confound sub-compartment interpretation
- Regions with no reads are not shown in the normalized bedgraph, so score profiles are incomplete for sparse windows

## Evidence

- [results] five of the 10 inferred sub-compartments (Sim1 to Sim5) resembled the GM12878 model, while the other five (Sim6 to Sim10) had a similar profile to the IMR90 model: [evidence span withheld — anchor status: not_found]
- [abstract] The compartment scores inferred using CscoreTool-M represents the probability of a genomic region locating in a specific sub-compartment: "The compartment scores inferred using CscoreTool-M represents the probability of a genomic region locating in a specific sub-compartment."
- [results] Simulate two independent Hi-C summary datasets, one parameterized from GM12878 and one from IMR90, and depth-match them before merging: [evidence span withheld — anchor status: not_found]
- [readme] XXX_cscore.txt This is the Cscore estimated for each genomic window.: "XXX_cscore.txt This is the Cscore estimated for each genomic window."
- [readme] Ths c-scores are normalized to add up to 1 for each genomic region. Regions with no reads are not shown.: "Ths c-scores are normalized to add up to 1 for each genomic region. Regions with no reads are not shown."
