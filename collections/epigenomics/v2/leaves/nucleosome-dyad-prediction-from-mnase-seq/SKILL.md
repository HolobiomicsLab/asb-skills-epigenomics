---
name: nucleosome-dyad-prediction-from-mnase-seq
description: Use when paired-end MNase-seq data (e.g. S. cerevisiae GSE153609) must
  be converted into nucleosome dyad positions and occupancy probabilities, and validated
  against ground-truth dyad annotations.
when_to_use_negative:
- Input is ATAC-seq rather than MNase-seq data when nucleosome-positioning fidelity
  is the priority — the article notes markedly better performance with MNase-seq,
  due to genome-wide fragment distribution and MNase exonuclease activity reflecting
  DBF sizes
- The goal is TF binding site identification rather than nucleosome dyad positioning
  — use the full multivariate RoboCOP occupancy landscape instead
- Alternative peak/footprint approaches (MACS2 followed by FIMO labeling) are intended
  — the article shows RoboCOP performs notably better, so that path is not equivalent
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: RoboCOP
  role: Multivariate HMM that integrates MNase-seq accessibility data with nucleotide
    sequence to compute genome-wide probabilistic nucleosome occupancy and call dyad
    positions
  repo: https://github.com/HarteminkLab/RoboCOP
- name: NucleoATAC
  role: Alternative nucleosome positioning method referenced for comparison (RoboCOP_ATAC
    positions many more nucleosomes than NucleoATAC on the same data)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:20b9d365262509893ffd673a89bd6fdf000aa98ea11768d09cdc400917f46079
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - NucleoATAC
  - RoboCOP
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# nucleosome-dyad-prediction-from-mnase-seq

## Summary

Infer genome-wide probabilistic nucleosome occupancy and extract dyad positions from paired-end MNase-seq data using RoboCOP, a multivariate HMM that integrates chromatin accessibility with nucleotide sequence, then validate dyads against high-precision chemical-cleavage annotations.

## When to use

Use when paired-end MNase-seq data (e.g. S. cerevisiae GSE153609) must be converted into nucleosome dyad positions and occupancy probabilities, and validated against ground-truth dyad annotations.

## When NOT to use

- Input is ATAC-seq rather than MNase-seq data when nucleosome-positioning fidelity is the priority — the article notes markedly better performance with MNase-seq, due to genome-wide fragment distribution and MNase exonuclease activity reflecting DBF sizes
- The goal is TF binding site identification rather than nucleosome dyad positioning — use the full multivariate RoboCOP occupancy landscape instead
- Alternative peak/footprint approaches (MACS2 followed by FIMO labeling) are intended — the article shows RoboCOP performs notably better, so that path is not equivalent

## Inputs

- Paired-end MNase-seq data for S. cerevisiae (GEO accession GSE153609)
- Nucleotide (reference genome) sequence
- RoboCOP repository from github.com/HarteminkLab/RoboCOP
- High-precision chemical-cleavage nucleosome dyad annotations (Brogaard et al., reference 24)

## Outputs

- Genome-wide probabilistic nucleosome occupancy profile (via multivariate HMM posterior probabilities)
- Called nucleosome dyad positions extracted from the occupancy profile
- Precision-recall curve of dyad predictions versus chemical-cleavage annotations (e.g. robocop_mnase_pr_curve.png)

## How to apply

Download paired-end MNase-seq data (e.g. GEO accession GSE153609 for S. cerevisiae) and clone the RoboCOP repository from GitHub. Compile the shared library with 'bash gccCompile' to generate librobocop.so and set its absolute path as cshared in config.ini (following config_example.ini). Run RoboCOP in MNase-seq-only mode following the example_robocop.ipynb vignette to infer genome-wide probabilistic nucleosome occupancy with the multivariate HMM, then extract called nucleosome dyad positions from the inferred occupancy profile. For validation, score each called dyad as a true positive if it lies within 50 bp of a dyad from the high-precision chemical-cleavage (Brogaard et al.) annotations, compute precision and recall over all called dyads, and plot the precision-recall curve.

## Related tools

- **RoboCOP** (Multivariate HMM that integrates MNase-seq accessibility data with nucleotide sequence to compute genome-wide probabilistic nucleosome occupancy and call dyad positions) — https://github.com/HarteminkLab/RoboCOP
- **NucleoATAC** (Alternative nucleosome positioning method referenced for comparison (RoboCOP_ATAC positions many more nucleosomes than NucleoATAC on the same data))

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git; cd RoboCOP/robocop; bash gccCompile
```

## Evaluation signals

- RoboCOP's dyad probability score peaks precisely at annotated dyads from the high-precision chemical-cleavage (Brogaard et al., reference 24) annotations
- Precision-recall curve of called dyads versus chemical-cleavage annotations reproduces Figure 4C / Supplementary Figure S12A
- librobocop.so is successfully generated by 'bash gccCompile' and its absolute path resolves correctly in config.ini (cshared entry)
- Called dyads counted as true positives lie within 50 bp of a reference chemical-cleavage dyad
- Inferred occupancy profile covers the genome at single-nucleotide resolution with probabilistic scores rather than hard calls

## Limitations

- No version numbers are stated for RoboCOP (or comparison tools MACS2, FIMO, NucleoATAC) in the article
- Accuracy and comprehensiveness of the occupancy profile improve with deeper sequencing; sequencing depth requirements are not quantified
- Prior information about DBF transition probabilities is not incorporated (deferred to future work)
- Performance is markedly better with MNase-seq than ATAC-seq, so ATAC-seq-based runs may under-represent nucleosome positioning fidelity

## Evidence

- [results] RoboCOP_MNase accurately recapitulates annotated nucleosome positions: its dyad probability score peaks precisely at the annotated dyads from reference (24): [evidence span withheld — anchor status: not_found]
