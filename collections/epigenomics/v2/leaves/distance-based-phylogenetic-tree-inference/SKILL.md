---
name: distance-based-phylogenetic-tree-inference
description: Use when you have decontaminated epiallele (or analogous binary/continuous
  feature) profiles per tumour region and need a phylogenetic tree linking sample
  evolutionary relationships.
when_to_use_negative:
- Input is raw read-level methylation data rather than decontaminated epiallele profiles
  — the Bayesian detection and purity/decontamination steps must run first
- Reads cannot be phased between disjoint loci, so the approach is unsuitable when
  phased multi-locus haplotypes are required
- Only a single bulk sample is available — the method relies on tracing epialleles
  across multiple tumour regions (the study used seven regions of CRUK0062 plus one
  matched normal)
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3295
tools:
- name: ape
  role: Provides the fastme.bal function used to fit the balanced minimum evolution
    tree to the averaged epiallele distance matrix
- name: bed
  role: R package (james-e-barrett/bed) implementing the Bayesian epiallele detection
    model that produces the epiallele profiles decontaminated prior to tree construction
  repo: github.com/james-e-barrett/bed
- name: R
  role: Language/environment in which profile loading, distance computation, and fastme.bal
    tree fitting are performed
  repo: github.com/james-e-barrett
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:dbc8aee9baaa380da070429e011b8e4b45b7c98ad773ee58d346f62a3cf5a2dc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ape
  - bed
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# distance-based-phylogenetic-tree-inference

## Summary

Reconstructs the evolutionary history of multi-region tumour samples by computing pairwise Euclidean distances between decontaminated epiallele profiles, averaging per-locus distance matrices, and fitting a tree with the balanced minimum evolution method (fastme.bal from the ape R package), output in Newick format.

## When to use

Use when you have decontaminated epiallele (or analogous binary/continuous feature) profiles per tumour region and need a phylogenetic tree linking sample evolutionary relationships.

## When NOT to use

- Input is raw read-level methylation data rather than decontaminated epiallele profiles — the Bayesian detection and purity/decontamination steps must run first
- Reads cannot be phased between disjoint loci, so the approach is unsuitable when phased multi-locus haplotypes are required
- Only a single bulk sample is available — the method relies on tracing epialleles across multiple tumour regions (the study used seven regions of CRUK0062 plus one matched normal)

## Inputs

- Decontaminated tumour epiallele profiles t_hat_s (vectors phi_q per sample, purity-corrected and clipped to [0,1])
- Per-sample epiallele proportions from the Bayesian epiallele detection (bed) model fitted to multi-region RRBS data
- Locus set passing filters: median read depth >= 100, >= 6 CpGs per locus, <= 25% missing data, autosomes only, loci present in >= 2 samples

## Outputs

- A phylogenetic tree in Newick format (e.g. tumour_phylogeny.nwk)
- The averaged inter-sample distance matrix across loci

## How to apply

Load the decontaminated tumour epiallele profiles t_hat_s (vectors phi_q per sample after purity correction rho and clipping to [0,1]) for each tumour region in R. For each analysed locus, compute the Euclidean distance between t_hat_s and t_hat_s' for every pair of samples to form a per-locus distance matrix, then average the per-locus matrices across all loci to obtain a single overall distance matrix. Fit the tree to this averaged matrix using fastme.bal (balanced minimum evolution) from the ape R package. Write the resulting tree to Newick format (e.g. tumour_phylogeny.nwk) and check that its topology is broadly consistent with an independently derived genetic tree of the same tumour. Loci feeding into this step should have been pre-filtered to median read depth >= 100, six or more CpGs, <= 25% missing data, and exclusion of chromosomes X and Y.

## Related tools

- **ape** (Provides the fastme.bal function used to fit the balanced minimum evolution tree to the averaged epiallele distance matrix)
- **bed** (R package (james-e-barrett/bed) implementing the Bayesian epiallele detection model that produces the epiallele profiles decontaminated prior to tree construction) — github.com/james-e-barrett/bed
- **R** (Language/environment in which profile loading, distance computation, and fastme.bal tree fitting are performed) — github.com/james-e-barrett

## Examples

```
library(ape); tree <- fastme.bal(D); write.tree(tree, file = "tumour_phylogeny.nwk")
```

## Evaluation signals

- The inferred tree is written successfully in Newick format and parses as a valid rooted/unrooted topology over the seven tumour regions (plus normal if included)
- Tree topology is broadly consistent with an independently derived genetic tree of the same tumour (cf. Fig. 4d)
- Per-locus distance matrices are symmetric, zero-diagonal, and averaged over exactly the loci passing the depth/CpG/missingness filters
- Decontaminated profiles t_hat_s used for distances are clipped to [0,1] after purity correction rho
- Epialleles accounting for < 5% of observed reads were discarded before computing phi_s, so distances exclude low-support epialleles

## Limitations

- Reads cannot be phased between disjoint loci, so epiallele number and entropy may be correlated between close loci
- Study limited to a single patient (one tumour with seven regions plus one matched normal); larger multi-patient studies deferred to future work
- Whether epiallele-level disorder quantification provides a more refined measure of tumour aggressiveness remains unverified

## Evidence

- [methods] We used the ‘fastme.bal’ function as part of the ‘ape’ R package [19].: "We used the ‘fastme.bal’ function as part of the ‘ape’ R package [19]."
- [methods] Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered.: "Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered."
- [methods] Any cases where t^μ<0 are set to 0 and any cases where t^μ>1 are set to 1.: [evidence span withheld — anchor status: not_found]
- [readme] To install this R package use the devtools install_github command: "To install this R package use the devtools install_github command"
