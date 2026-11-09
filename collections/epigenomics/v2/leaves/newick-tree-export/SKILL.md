---
name: newick-tree-export
description: Use when multi-region tumour epiallele presence/absence profiles (post
  purity correction) need to be converted into an evolutionary tree relating tumour
  regions and matched normal tissue.
when_to_use_negative:
- Epiallele profiles have not been decontaminated — normal tissue contamination (parameter
  rho) must first be estimated and profiles corrected and clipped to [0,1]
- Reads cannot be phased between disjoint loci, so this tree inference is unsuitable
  when the goal requires phasing epialleles across distant loci
- Only one tumour sample region is available — the method requires multiple tissue
  samples to build pairwise distance matrices
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0622
tools:
- name: R
  role: Language used to compute per-locus Euclidean distance matrices from decontaminated
    epiallele profiles and average them
  repo: github.com/james-e-barrett
- name: ape
  role: Provides the fastme.bal (balanced minimum evolution) function that fits the
    phylogenetic tree to the averaged distance matrix
- name: bed
  role: R package that performs the upstream Bayesian epiallele detection producing
    the decontaminated profiles used for tree construction
  repo: github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:98f0c3601373722f374ad5ceb557fb900846d8d5da0b575e64adb775575da816
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/github.com/james-e-barrett
  tools:
  - R
  - ape
  - bed
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# newick-tree-export

## Summary

Infers a tumour phylogenetic tree from decontaminated epiallele profiles by averaging per-locus Euclidean distance matrices across samples and fitting a balanced minimum evolution tree with fastme.bal (ape R package), exported in Newick format.

## When to use

Use when multi-region tumour epiallele presence/absence profiles (post purity correction) need to be converted into an evolutionary tree relating tumour regions and matched normal tissue.

## When NOT to use

- Epiallele profiles have not been decontaminated — normal tissue contamination (parameter rho) must first be estimated and profiles corrected and clipped to [0,1]
- Reads cannot be phased between disjoint loci, so this tree inference is unsuitable when the goal requires phasing epialleles across distant loci
- Only one tumour sample region is available — the method requires multiple tissue samples to build pairwise distance matrices

## Inputs

- Decontaminated tumour epiallele profiles t_hat_s per sample (phi_q vectors after purity correction rho and clipping to [0,1])
- Locus-level epiallele proportions for multiple tumour regions and matched normal tissue (e.g. seven CRUK0062 tumour regions plus one normal)

## Outputs

- Phylogenetic tree in Newick format (e.g. tumour_phylogeny.nwk) relating the tumour regions and matched normal sample
- Averaged inter-sample distance matrix over all analysed loci

## How to apply

Load the decontaminated tumour epiallele profiles t_hat_s (vectors phi_q per sample after purity correction rho and clipping to [0,1]) into R. For each analysed locus, compute Euclidean distances between t_hat_s and t_hat_s' for every pair of samples to build a per-locus distance matrix, then average these matrices across all loci into a single overall distance matrix. Fit the tree to this averaged matrix with the fastme.bal (balanced minimum evolution) function from the ape R package. Write the resulting tree to a Newick file (e.g. tumour_phylogeny.nwk) and compare its topology against an independently derived genetic tree for the same tumour (cf. Fig. 4d of the CRUK0062 case study).

## Related tools

- **R** (Language used to compute per-locus Euclidean distance matrices from decontaminated epiallele profiles and average them) — github.com/james-e-barrett
- **ape** (Provides the fastme.bal (balanced minimum evolution) function that fits the phylogenetic tree to the averaged distance matrix)
- **bed** (R package that performs the upstream Bayesian epiallele detection producing the decontaminated profiles used for tree construction) — github.com/james-e-barrett/bed

## Examples

```
library(ape); tree <- fastme.bal(avg_dist_matrix); write.tree(tree, file="tumour_phylogeny.nwk")
```

## Evaluation signals

- The exported .nwk file parses as valid Newick and contains one leaf per analysed tissue sample (e.g. seven CRUK0062 tumour regions plus matched normal)
- Tree topology is broadly consistent with an independently derived genetic tree of the same tumour (cf. Fig. 4d)
- Each per-locus distance matrix is symmetric with zero diagonal before averaging across loci
- Decontaminated epiallele proportions t_hat used as input are clipped to the range [0,1]
- The number of loci contributing to the averaged matrix matches the filtered loci set (median read depth >= 100, >= 6 CpGs, <= 25% missing data, autosomes only)

## Limitations

- Reads cannot be phased between disjoint loci, although epiallele number and entropy may correlate between close loci
- Case study limited to a single patient (one tumour with seven regions plus one matched normal); larger multi-patient studies deferred to future work
- Discarded epialleles (<5% of observed reads) and filtered loci (chromosomes X/Y, <6 CpGs, <100 median read depth, >25% missing) reduce the loci available for the distance matrix

## Evidence

- [methods] We used the ‘fastme.bal’ function as part of the ‘ape’ R package [19].: "We used the ‘fastme.bal’ function as part of the ‘ape’ R package [19]."
- [intro] By tracing the presence and absence of different epialleles across different regions of the tumour and matched normal tissue it is possible to reconstruct the evolutionary history of the tumour: "By tracing the presence and absence of different epialleles across different regions of the tumour and matched normal tissue it is possible to reconstruct the evolutionary history of the tumour"
- [methods] once estimates of ρ have been obtained we can calculate the ‘decontaminated’ tumour epiallele profiles at each locus: "once estimates of ρ have been obtained we can calculate the ‘decontaminated’ tumour epiallele profiles at each locus"
- [readme] To install this R package use the devtools install_github command: "To install this R package use the devtools install_github command"
