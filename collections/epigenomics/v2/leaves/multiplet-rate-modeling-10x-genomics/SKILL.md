---
name: multiplet-rate-modeling-10x-genomics
description: Use when planning or costing a 10X droplet scRNA-seq experiment with
  scPower and you must account for usable cells per lane given multiplet loss as a
  function of cells loaded per lane.
when_to_use_negative:
- Data were generated with plate-based (non-droplet) or non-10X single cell protocols,
  for which the 10X user-guide-V3-derived multiplet model does not apply
- You already have an empirical doublet rate measured for your own experiment (e.g.
  from hashtag multiplexing or Scrublet), which should take precedence over the generic
  linear model
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPower
  role: Implements the multiplet rate model (linear in cells loaded per lane, fitted
    from 10X Genomics user guide V3) inside its droplet-based cost and power analysis
    framework
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:197252d2f343694e1fb67c5221e273ed0c28525e4ac3efaf979c6c88085abe1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - scPower
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Multiplet rate modeling for 10X Genomics droplet-based scRNA-seq

## Summary

Models the 10X Genomics multiplet (doublet) rate as a linear function of the number of cells loaded per lane, calibrated on the 10X Genomics user guide V3, so that experimental design and budget calculations account for cells lost to multiplets.

## When to use

Use when planning or costing a 10X droplet scRNA-seq experiment with scPower and you must account for usable cells per lane given multiplet loss as a function of cells loaded per lane.

## When NOT to use

- Data were generated with plate-based (non-droplet) or non-10X single cell protocols, for which the 10X user-guide-V3-derived multiplet model does not apply
- You already have an empirical doublet rate measured for your own experiment (e.g. from hashtag multiplexing or Scrublet), which should take precedence over the generic linear model

## Inputs

- Number of cells loaded per lane (droplet-based 10X chemistry)
- 10X droplet cost model parameters: costKit (EUR), costFlowCell (EUR), readsPerFlowcell, samplesPerLane, mappingEfficiency
- 10X Genomics user guide V3 multiplet rate data (bundled reference)

## Outputs

- Multiplet rate as a linear function of cells loaded per lane
- Multiplet-adjusted count of usable cells per sample/lane
- Total experimental cost (library preparation + sequencing) and power estimates per design

## How to apply

scPower treats the multiplet rate as linearly increasing with the number of cells loaded per lane, with the relationship fitted from the 10X Genomics user guide V3. When defining the 10X droplet cost model, supply the lane/channel structure (samplesPerLane=4), library kit and flow-cell costs (e.g. costKit=5600 EUR, costFlowCell=14032 EUR, readsPerFlowcell=4.1e6), and mappingEfficiency=0.8; the multiplet model then determines how many of the targeted cells per sample are usable, which feeds into both the power calculation and the total experimental cost (library preparation plus sequencing). Use this when evaluating parameter grids of cells per individual and read depth under a fixed budget, since deeper loading raises multiplet loss and reduces effective sample size. Validate by checking that the selected design's computed cost stays within the budget and that the expected number of usable cells reflects the multiplet-adjusted rate.

## Related tools

- **scPower** (Implements the multiplet rate model (linear in cells loaded per lane, fitted from 10X Genomics user guide V3) inside its droplet-based cost and power analysis framework) — https://github.com/heiniglab/scPower

## Examples

```
# 10X droplet cost model used with scPower's budget optimizer
optimize.constant.budget(totalBudget=10000, nCellsRange=seq(1000,10000,by=1000), readDepthRange=seq(10000,50000,by=5000), samplesPerLane=4, mappingEfficiency=0.8)
```

## Evaluation signals

- The multiplet rate used in the design evaluation increases monotonically and linearly with cells loaded per lane, consistent with the 10X Genomics user guide V3 fit
- The expected number of usable cells per sample is strictly less than the loaded cell count once multiplet loss is applied
- For the budget-constrained runs, the selected design's computed cost (library + sequencing with costKit=5600 EUR, costFlowCell=14032 EUR, readsPerFlowcell=4.1e6, samplesPerLane=4, mappingEfficiency=0.8) does not exceed the total budget
- Power surfaces (powerDetect vs. totalCells and readDepth) shift predictably when multiplet loss is included, since fewer usable cells lowers detection power

## Limitations

- Actual experiments might result in higher doublet rates and a lower number of usable cells than the default linear model assumption
- The multiplet rate relationship is fitted from the 10X Genomics user guide V3 only; extrapolation to other chemistries or very high cell loading is not validated
- Data-driven priors (and hence multiplet-adjusted power) are only reliable for sample sizes smaller than or roughly equal to the pilot/reference data sets

## Evidence

- [methods] We modelled the multiplet rate linear increasing with the number of cells loaded per lane, using data from the 10X Genomics user guide V3: "We modelled the multiplet rate linear increasing with the number of cells loaded per lane, using data from the 10X Genomics user guide V3"
- [methods] The experimental costs are the sum of the library preparation cost and the sequencing costs.: "The experimental costs are the sum of the library preparation cost and the sequencing costs."
- [discussion] actual experiments might result in higher doublet rates and lower number of usable cells: "actual experiments might result in higher doublet rates and lower number of usable cells."
- [readme] The tool was evaluated with data from different tissues and single cell technologies, based on UMI counts and read counts.: "The tool was evaluated with data from different tissues and single cell technologies, based on UMI counts and read counts."
