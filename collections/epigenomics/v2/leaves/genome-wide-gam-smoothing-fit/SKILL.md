---
name: genome-wide-gam-smoothing-fit
description: Use when you need base-level smoothing and significance/peak calls for
  ChIP-Seq protein-DNA occupancy across whole chromosomes or gigabase-scale genomes
  (e.g., TF binding, genotype contrasts) without ad-hoc binning or windowing.
when_to_use_negative:
- Input is a gene- or region-level count table (e.g., an RNA-Seq expression matrix)
  rather than per-position ChIP-Seq counts — GenoGAM models read count frequencies
  as smooth functions along chromosomes.
- A quick exploratory fit on data lacking the factorial genotype/treatment structure
  assumed by the design ~ s(x) + s(x, by = genotype) — the fitting path and the s(x):genotype
  term interpretability depend on it.
- You only need alignment or read counting — this skill starts from aligned BAM files
  and an experiment design.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3169
tools:
- name: GenoGAM 2.0
  role: 'Core R/Bioconductor package: builds the HDF5-backed GenoGAMDataSet, computes
    size factors, fits the tiled GAM via genogam(), and provides computeSignificance()/callPeaks()'
  repo: https://github.com/gstricker/fastGenoGAM
- name: GenoGAM (Bioconductor release)
  role: Current maintained version of the package replacing the deprecated fastGenoGAM
    implementation
  repo: https://github.com/gstricker/GenoGAM
- name: mgcv
  role: General-purpose R library for fitting GAMs on which GenoGAM's model framework
    is built
- name: SuperLU
  role: Direct sparse solver exploited for GAM parameter fitting, making whole-genome
    fits 2-3 orders of magnitude faster
- name: HDF5
  role: On-disk storage backend (hdf5 = TRUE) that keeps the memory footprint and
    I/O low for out-of-core genome-wide fits
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:1d7f41c91566f93ed5fc273cd898e5d4dffcafa093cb907afa3862f9b5584e25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - Bioconductor
  - GenoGAM (Bioconductor release)
  - GenoGAM 2.0
  - HDF5
  - R
  - SuperLU
  - mgcv
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# genome-wide-gam-smoothing-fit

## Summary

Fit genome-wide generalized additive models (GAMs) to ChIP-Seq read counts with GenoGAM 2.0: build an HDF5-backed GenoGAMDataSet, compute size factors, and fit smooth count functions per genomic tile (SuperLU sparse solver, mgcv framework) to obtain base-level fits, position-wise log p-values, and called peaks at gigabase scale.

## When to use

Use when you need base-level smoothing and significance/peak calls for ChIP-Seq protein-DNA occupancy across whole chromosomes or gigabase-scale genomes (e.g., TF binding, genotype contrasts) without ad-hoc binning or windowing.

## When NOT to use

- Input is a gene- or region-level count table (e.g., an RNA-Seq expression matrix) rather than per-position ChIP-Seq counts — GenoGAM models read count frequencies as smooth functions along chromosomes.
- A quick exploratory fit on data lacking the factorial genotype/treatment structure assumed by the design ~ s(x) + s(x, by = genotype) — the fitting path and the s(x):genotype term interpretability depend on it.
- You only need alignment or read counting — this skill starts from aligned BAM files and an experiment design.

## Inputs

- ChIP-Seq BAM files plus an experiment design table (e.g., experimentDesign.txt from extdata/Set1) covering IP/control or genotype conditions
- GAM design formula, e.g. ~ s(x) + s(x, by = genotype) with x = genomic position
- Chunk/overhang sizes (chunkSize = 50000, overhangSize = 1000) and an HDF5 storage folder via GenoGAMSettings
- Optional fixed hyperparameters (lambda, theta) to skip cross-validated smoothing-parameter estimation

## Outputs

- GenoGAM object with per-chromosome smooth fits and standard errors (fits(result), se(result))
- Position-wise log p-values accessible via pval(result) after computeSignificance()
- Significant regions / called peaks from callPeaks()
- HDF5-backed on-disk stores of input data and results (out-of-core processing)

## How to apply

Read the BAM files and experiment design into an HDF5-backed GenoGAMDataSet (chunkSize = 50000, overhangSize = 1000, design = ~ s(x) + s(x, by = genotype) for factorial IP/genotype experiments), then normalize library sizes with computeSizeFactors(). Fit the model with genogam(), optionally supplying fixed hyperparameters (vignette values lambda = 4601, theta = 4.51) to skip cross-validated smoothing-parameter estimation on small example data. GenoGAM fits the GAM on tiles — overlapping genomic intervals joined at overlap midpoints — exploiting model sparsity via the SuperLU direct solver and building on mgcv, with HDF5 keeping the memory footprint and I/O low; this made whole-genome human fits (ca. 300 million parameters) feasible in under 9 hours on a 60-core server. Extract fits(result) and se(result) per chromosome, compute position-wise log p-values with computeSignificance(), and extract significant regions with callPeaks(). Judge correctness by fit continuity at tile joins (e.g., plotting the center 10 kb where two tiles meet) and, for the peak-summit benchmark, by the proportion of significant peaks within 30 bp of the motif center (95% bootstrap CI) matching GenoGAM 1.0 equivalence values across the seven datasets (yeast TFIIB plus chromosome 22 CEBPB, CTCF, USF1, MAX, PAX5, YY1).

## Related tools

- **GenoGAM 2.0** (Core R/Bioconductor package: builds the HDF5-backed GenoGAMDataSet, computes size factors, fits the tiled GAM via genogam(), and provides computeSignificance()/callPeaks()) — https://github.com/gstricker/fastGenoGAM
- **GenoGAM (Bioconductor release)** (Current maintained version of the package replacing the deprecated fastGenoGAM implementation) — https://github.com/gstricker/GenoGAM
- **mgcv** (General-purpose R library for fitting GAMs on which GenoGAM's model framework is built)
- **SuperLU** (Direct sparse solver exploited for GAM parameter fitting, making whole-genome fits 2-3 orders of magnitude faster)
- **HDF5** (On-disk storage backend (hdf5 = TRUE) that keeps the memory footprint and I/O low for out-of-core genome-wide fits)

## Examples

```
ggd <- GenoGAMDataSet(expDesign, chunkSize = 50000, overhangSize = 1000, design = design, hdf5 = TRUE); ggd <- computeSizeFactors(ggd); result <- genogam(ggd, lambda = 4601, theta = 4.51)
```

## Evaluation signals

- fits(result) and se(result) return per-chromosome smooths; plotting the center 10 kb (idx 45000:55000) where two tiles join should show curves continuous across the overlap midpoint.
- genogam(ggd, lambda = 4601, theta = 4.51) with the vignette's fixed hyperparameters should reproduce the example fit without cross-validated smoothing-parameter estimation.
- computeSignificance() produces position-wise log p-values reachable via pval(result), and callPeaks() regions must derive from those p-values.
- For the peak-summit benchmark, the proportion of significant peaks within 30 bp of the motif center (95% bootstrap CI) must match the GenoGAM 1.0 equivalence values across the seven datasets, written to motif_proximity_results.csv.
- computeSizeFactors() must run before genogam(), and the HDF5 folder must hold the out-of-core data instead of in-memory objects.

## Limitations

- The fastGenoGAM repository version is deprecated; new work should use the Bioconductor GenoGAM release.
- Requires R 3.4.1 or higher, constraining compatible environments; licensed under GPL-2.
- Armadillo may default to 32-bit matrices, causing 'matrix multiplication: incompatible matrix dimensions' errors on large fits — fix by enabling ARMA_64BIT_WORD in RcppArmadilloConfig.h.
- Performance claims are hardware-dependent ('a typical lab server', 'a standard 60-core server') without fully specified machine specs, and no archival (Zenodo) record is provided beyond GitHub hosting.

## Evidence

- [abstract] sparse-solver rationale: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [intro] tiling strategy: "Genome-wide fits were obtained by fitting models on tiles, defined as overlapping genomic intervals of a tractable size, and joining together tile fits at overlap midpoints."
- [methods] significance step: "compute position-wise log p-values with `computeSignificance()`"
- [readme] fixed hyperparameters: "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [readme] HDF5-backed read-in: "Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object. This is done with the HDF5 backend."
