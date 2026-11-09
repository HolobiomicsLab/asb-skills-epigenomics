---
name: genomic-coordinate-distance-matching
description: Use when validating a coordinate-level genome-wide prediction (e.g. called
  nucleosome dyads) against high-precision reference annotations, and you need a distance-thresholded
  precision-recall curve.
when_to_use_negative:
- Input predictions are not coordinate-level calls (e.g. only continuous occupancy
  scores without extracted dyad positions) — the method requires discrete coordinates
  to distance-match against reference dyads.
- No high-precision reference annotations exist for the organism; the ground truth
  here is specifically the chemical cleavage (Brogaard et al.) dyad set for S. cerevisiae.
- Evaluating TF binding site predictions rather than nucleosome positioning — those
  are validated against ChIP-chip, ChIP-exo, and ORGANIC annotations instead.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RoboCOP
  role: Produces the predicted nucleosome dyad coordinates and dyad probability scores
    being validated; run in MNase-seq-only mode via the multivariate HMM.
  repo: https://github.com/HarteminkLab/RoboCOP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:f422cedc9dafb33dfdd22d0658ab5d17eb86a67df303a7e4ec631b34136e0afb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - RoboCOP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# genomic-coordinate-distance-matching

## Summary

A validation procedure that matches predicted genomic coordinates (e.g. RoboCOP_MNase nucleosome dyads) to ground-truth annotations by genomic distance, counting a prediction as a true positive only when it falls within a fixed bp window of a reference coordinate, then computing precision and recall.

## When to use

Use when validating a coordinate-level genome-wide prediction (e.g. called nucleosome dyads) against high-precision reference annotations, and you need a distance-thresholded precision-recall curve.

## When NOT to use

- Input predictions are not coordinate-level calls (e.g. only continuous occupancy scores without extracted dyad positions) — the method requires discrete coordinates to distance-match against reference dyads.
- No high-precision reference annotations exist for the organism; the ground truth here is specifically the chemical cleavage (Brogaard et al.) dyad set for S. cerevisiae.
- Evaluating TF binding site predictions rather than nucleosome positioning — those are validated against ChIP-chip, ChIP-exo, and ORGANIC annotations instead.

## Inputs

- Called nucleosome dyad positions extracted from RoboCOP's inferred genome-wide probabilistic nucleosome occupancy profile (MNase-seq-only mode)
- High-precision chemical cleavage (Brogaard et al., ref 24) nucleosome dyad annotations for S. cerevisiae
- RoboCOP dyad probability score per called dyad for threshold sweeping

## Outputs

- Precision and recall values over all called dyads at the 50 bp matching threshold
- Precision-recall curve for RoboCOP_MNase (reproducing Figure 4C / Supplementary Figure S12A), e.g. saved as robocop_mnase_pr_curve.png

## How to apply

Run RoboCOP in MNase-seq-only mode (paired-end MNase-seq for S. cerevisiae from GEO accession GSE153609) following the example_robocop.ipynb vignette, after compiling librobocop.so via 'bash gccCompile' and setting its absolute path as cshared in config.ini. Extract the called nucleosome dyad positions from the inferred genome-wide probabilistic occupancy profile produced by the multivariate HMM. Score each called dyad as a true positive if it lies within 50 bp of a dyad from the high-precision chemical cleavage (Brogaard et al.) annotations, sweeping the dyad probability score threshold to compute precision and recall over all called dyads. Plot the precision-recall curve (reproducing Figure 4C and Supplementary Figure S12A) and confirm the RoboCOP_MNase dyad probability score peaks precisely at the annotated dyads — alignment of prediction peaks with ground-truth dyads is the signal that matching succeeded.

## Related tools

- **RoboCOP** (Produces the predicted nucleosome dyad coordinates and dyad probability scores being validated; run in MNase-seq-only mode via the multivariate HMM.) — https://github.com/HarteminkLab/RoboCOP

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git; cd RoboCOP/robocop && bash gccCompile && ls "`pwd`/librobocop.so"
```

## Evaluation signals

- Each called dyad is a true positive only when within 50 bp of a chemical-cleavage (Brogaard) reference dyad — verify the 50 bp window was applied consistently.
- The RoboCOP_MNase dyad probability score should peak precisely at the annotated dyads, indicating close alignment with ground-truth positions.
- Precision-recall curve shape should reproduce Figure 4C and Supplementary Figure S12A of the paper.
- librobocop.so compiled successfully (bash gccCompile) and cshared path in config.ini resolves to its absolute path before running predictions.

## Limitations

- Markedly better performance is observed with MNase-seq than ATAC-seq, so validation quality depends on the accessibility assay used; ATAC-seq-based predictions may validate more poorly.
- A fixed 50 bp tolerance window is a modeling choice; dyads outside the window are scored as false positives even if biologically near-correct.
- The article reports no version numbers for RoboCOP, and accuracy of chromatin occupancy profiles improves with deeper sequencing (sequencing depth requirements not quantified).

## Evidence

- [full_text] Score each called dyad as a true positive if it lies within 50 bp of a dyad from the high-precision chemical cleavage (24) annotations, and compute precision and recall over all called dyads.: [evidence span withheld — anchor status: not_found]
