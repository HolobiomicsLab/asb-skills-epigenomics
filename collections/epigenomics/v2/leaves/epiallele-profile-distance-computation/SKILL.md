---
name: epiallele-profile-distance-computation
description: Use when multi-region tumour RRBS data have been processed into decontaminated
  epiallele profiles t_hat_s per sample and you need a sample-by-sample distance matrix
  or a tumour phylogenetic tree.
when_to_use_negative:
- Input is raw RRBS reads rather than fitted, decontaminated epiallele profiles phi_q
  — the Bayesian bed model (bed_fit/bed_marginal) must be run first
- Samples lack an estimated purity rho, so profiles are contaminated with normal tissue
  and distances between tumour samples would be biased
- The goal is single-sample epigenetic disorder quantification (Shannon entropy of
  phi_s), not between-sample phylogenetic comparison
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: R
  role: Environment in which the distance matrices are computed and the tree is fitted
  repo: github.com/james-e-barrett
- name: ape
  role: Provides the fastme.bal (balanced minimum evolution) function that fits the
    phylogenetic tree to the averaged distance matrix and outputs Newick format
- name: bed
  role: R package implementing the Bayesian epiallele detection that produces the
    decontaminated profiles t_hat_s consumed by this skill
  repo: github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:9fcdc4b381576526e8ecb7232ce0816afa3b57fbb6c34257d05897a9603fd766
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/github.com/james-e-barrett
  tools:
  - R
  - ape
  - bed
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# epiallele-profile-distance-computation

## Summary

Computes a pairwise sample distance matrix from decontaminated tumour epiallele profiles (t_hat_s, i.e. purity-corrected vectors phi_q clipped to [0,1]) and fits a phylogenetic tree with fastme.bal, yielding a Newick-format tumour phylogeny.

## When to use

Use when multi-region tumour RRBS data have been processed into decontaminated epiallele profiles t_hat_s per sample and you need a sample-by-sample distance matrix or a tumour phylogenetic tree.

## When NOT to use

- Input is raw RRBS reads rather than fitted, decontaminated epiallele profiles phi_q — the Bayesian bed model (bed_fit/bed_marginal) must be run first
- Samples lack an estimated purity rho, so profiles are contaminated with normal tissue and distances between tumour samples would be biased
- The goal is single-sample epigenetic disorder quantification (Shannon entropy of phi_s), not between-sample phylogenetic comparison

## Inputs

- Decontaminated epiallele profiles t_hat_s (vectors phi_q per tumour sample, after purity correction rho and clipping to [0,1])
- Set of analysed loci shared by samples (e.g. the seven CRUK0062 tumour regions plus matched normal)
- Fitted purity estimate rho from the Bayesian epiallele detection model

## Outputs

- Per-locus pairwise Euclidean distance matrices between samples
- Single averaged overall sample distance matrix
- Phylogenetic tree in Newick format (e.g. tumour_phylogeny.nwk)

## How to apply

Load the decontaminated epiallele profiles t_hat_s (vectors phi_q per sample after purity correction rho and clipping to [0,1]) in R. For each analysed locus, compute the Euclidean distance between t_hat_s and t_hat_s' for every pair of samples to form a per-locus distance matrix, then average the per-locus matrices across all loci into a single overall distance matrix. Fit a phylogenetic tree to the averaged matrix with the fastme.bal (balanced minimum evolution) function from the ape R package. Write the resulting tree to a Newick file (e.g. tumour_phylogeny.nwk) and compare its topology to an independently derived genetic tree for validation.

## Related tools

- **R** (Environment in which the distance matrices are computed and the tree is fitted) — github.com/james-e-barrett
- **ape** (Provides the fastme.bal (balanced minimum evolution) function that fits the phylogenetic tree to the averaged distance matrix and outputs Newick format)
- **bed** (R package implementing the Bayesian epiallele detection that produces the decontaminated profiles t_hat_s consumed by this skill) — github.com/james-e-barrett/bed

## Examples

```
library(ape); tree <- fastme.bal(D_avg); write.tree(tree, file="tumour_phylogeny.nwk")
```

## Evaluation signals

- Per-locus distance matrices are Euclidean and symmetric with zero diagonal
- Decontaminated proportions lie within [0,1]: any t^mu < 0 set to 0 and any t^mu > 1 set to 1 before distance computation
- Overall matrix equals the mean of the per-locus distance matrices across all analysed loci
- fastme.bal returns a rooted/unrooted tree object that can be written as valid Newick (tumour_phylogeny.nwk)
- Inferred topology is broadly consistent with an independently derived genetic tree (cf. Fig. 4d for CRUK0062)

## Limitations

- Reads cannot be phased between disjoint loci, so distances are per-locus averaged rather than haplotype-aware
- Study validated on a single patient (one tumour, seven regions plus one matched normal); generalisation to multi-patient data was deferred to future work

## Evidence

- [methods] fastme.bal tree fitting with ape: "We used the ‘fastme.bal’ function as part of the ‘ape’ R package [19]."
- [methods] decontamination precedes profile use: "once estimates of ρ have been obtained we can calculate the ‘decontaminated’ tumour epiallele profiles at each locus"
- [methods] clipping to [0,1]: [evidence span withheld — anchor status: not_found]
- [readme] bed package installation context: "To install this R package use the devtools install_github command:

    library(devtools)
    install_github("james-e-barrett/bed")"
