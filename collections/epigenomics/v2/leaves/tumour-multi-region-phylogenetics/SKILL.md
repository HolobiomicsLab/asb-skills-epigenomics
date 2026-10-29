---
name: tumour-multi-region-phylogenetics
description: Use when you have decontaminated epiallele profiles t_hat_s for multiple
  tumour regions (plus matched normal) and need to infer the tumour's phylogenetic
  tree from epiallele presence/absence.
when_to_use_negative:
- You only have a single tumour region sample — the model borrows information from
  all tumour regions to gain statistical power, so multi-region data are required
- The epiallele profiles have not been purity-corrected/decontaminated — normal tissue
  contamination must be estimated and corrected before building the tree
- You need to phase reads across disjoint loci — this is a stated limitation of the
  approach and cannot be done
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: R
  role: Implementation environment; the decontaminated profiles are loaded and per-locus
    Euclidean distance matrices computed and averaged in R
  repo: github.com/james-e-barrett
- name: ape
  role: Provides the 'fastme.bal' balanced-minimum-evolution function used to fit
    the phylogenetic tree to the averaged distance matrix and output it in Newick
    format
- name: devtools
  role: Used to install the bed R package (install_github('james-e-barrett/bed'))
    that produces the decontaminated epiallele profiles feeding the tree
  repo: github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:3590b956630cf4dd1d13a9f40a0ce4b519eb6ae76e0a08ad68222a095dac8479
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/github.com/james-e-barrett
  tools:
  - R
  - ape
  - devtools
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# tumour-multi-region-phylogenetics

## Summary

Reconstructs the evolutionary history of a multi-region tumour from decontaminated epiallele presence/absence profiles by averaging per-locus pairwise sample distances and fitting a balanced-minimum-evolution tree, output in Newick format.

## When to use

Use when you have decontaminated epiallele profiles t_hat_s for multiple tumour regions (plus matched normal) and need to infer the tumour's phylogenetic tree from epiallele presence/absence.

## When NOT to use

- You only have a single tumour region sample — the model borrows information from all tumour regions to gain statistical power, so multi-region data are required
- The epiallele profiles have not been purity-corrected/decontaminated — normal tissue contamination must be estimated and corrected before building the tree
- You need to phase reads across disjoint loci — this is a stated limitation of the approach and cannot be done

## Inputs

- Decontaminated tumour epiallele profiles t_hat_s (vectors phi_q per sample) after purity correction rho and clipping to [0,1]
- RRBS-derived methylation data from multiple regions of a single tumour and one matched normal tissue sample (e.g. seven CRUK0062 tumour regions plus matched normal)

## Outputs

- Phylogenetic tree in Newick format (e.g. tumour_phylogeny.nwk) describing relationships among tumour regions

## How to apply

Start from the decontaminated tumour epiallele profiles t_hat_s (vectors phi_q per sample, obtained after purity correction rho and clipping negative values to 0 and values >1 to 1). For each analysed locus, compute the Euclidean distance between t_hat_s and t_hat_s' for every pair of samples to form a per-locus distance matrix, then average these matrices across all loci into a single overall distance matrix, as prescribed by the methods. Fit a phylogenetic tree to the averaged distance matrix with the 'fastme.bal' (balanced minimum evolution) function from the ape R package. Write the resulting tree to a Newick file (e.g. tumour_phylogeny.nwk) and compare its topology against an independently derived genetic tree of the same tumour. Rationale: tracing epiallele presence/absence across regions and matched normal tissue allows reconstruction of tumour evolutionary history and measurement of global epigenetic disorder.

## Related tools

- **R** (Implementation environment; the decontaminated profiles are loaded and per-locus Euclidean distance matrices computed and averaged in R) — github.com/james-e-barrett
- **ape** (Provides the 'fastme.bal' balanced-minimum-evolution function used to fit the phylogenetic tree to the averaged distance matrix and output it in Newick format)
- **devtools** (Used to install the bed R package (install_github('james-e-barrett/bed')) that produces the decontaminated epiallele profiles feeding the tree) — github.com/james-e-barrett/bed

## Examples

```
library(ape); tree <- fastme.bal(avg_dist_matrix); write.tree(tree, file="tumour_phylogeny.nwk")
```

## Evaluation signals

- Every pair of tumour-region samples has a finite Euclidean distance for each analysed locus, and the averaged overall distance matrix is square, symmetric, with zero diagonal
- The decontaminated proportions t_hat_s contain no values <0 or >1 (clipping applied: 'Any cases where t^μ<0 are set to 0 and any cases where t^μ>1 are set to 1')
- The output is a valid Newick-format tree (tumour_phylogeny.nwk) whose tips are the tumour regions plus matched normal
- The inferred tree topology is broadly consistent with an independently derived genetic tree of the same tumour (cf. Fig. 4d of the paper)
- The fastme.bal call from the ape R package completes without error on the averaged distance matrix

## Limitations

- Reads cannot be phased between disjoint loci, which limits resolution at linked loci
- The approach was demonstrated on a single patient (one tumour with seven regions plus one matched normal); larger multi-patient studies were deferred to future work
- Epialleles accounting for less than 5% of observed reads are discarded prior to computing phi_s, so low-frequency epialleles cannot inform the tree

## Evidence

- [methods] tree via fastme.bal: "We used the ‘fastme.bal’ function as part of the ‘ape’ R package [19]."
- [methods] decontamination clipping: [evidence span withheld — anchor status: not_found]
- [abstract] phylogeny rationale: "By tracing the distribution of epialleles throughout the tumour we can infer the phylogenetic history of the tumour, identify epialleles that differ between normal and cancer tissue"
- [readme] package install: "To install this R package use the devtools install_github command:

    library(devtools)
    install_github("james-e-barrett/bed")"
