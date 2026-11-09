---
name: precision-recall-curve-evaluation
description: Use when you have computationally called nucleosome dyad positions plus
  independent, high-precision ground-truth dyad annotations (e.g., Brogaard et al.
  chemical cleavage) and need to quantify positioning accuracy via precision-recall.
when_to_use_negative:
- Validating transcription factor binding site predictions — the paper does that against
  ChIP-chip, ChIP-exo, and ORGANIC annotations, not dyad-based PR curves
- The organism or conditions lack independent high-precision dyad annotations (e.g.,
  chemical-cleavage data) to serve as ground truth
- Evaluating occupancy/coverage accuracy rather than dyad-position accuracy — this
  skill only scores dyad positions within a 50 bp window
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: RoboCOP
  role: Multivariate HMM run in MNase-seq-only mode to infer genome-wide probabilistic
    nucleosome occupancy and called dyad positions that are then PR-evaluated
  repo: https://github.com/HarteminkLab/RoboCOP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:3c9e6f7ff4682cec1e024249744432081bf03ca99529a6580074fb022e839025
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - RoboCOP
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# precision-recall-curve-evaluation

## Summary

Validates genome-wide nucleosome dyad predictions (e.g., from RoboCOP_MNase) by scoring called dyads against high-precision chemical-cleavage (Brogaard et al., ref 24) annotations and plotting a precision-recall curve, as in Figure 4C and Supplementary Figure S12A of the RoboCOP paper.

## When to use

Use when you have computationally called nucleosome dyad positions plus independent, high-precision ground-truth dyad annotations (e.g., Brogaard et al. chemical cleavage) and need to quantify positioning accuracy via precision-recall.

## When NOT to use

- Validating transcription factor binding site predictions — the paper does that against ChIP-chip, ChIP-exo, and ORGANIC annotations, not dyad-based PR curves
- The organism or conditions lack independent high-precision dyad annotations (e.g., chemical-cleavage data) to serve as ground truth
- Evaluating occupancy/coverage accuracy rather than dyad-position accuracy — this skill only scores dyad positions within a 50 bp window

## Inputs

- Paired-end MNase-seq data for S. cerevisiae (GEO accession GSE153609)
- RoboCOP-inferred genome-wide probabilistic nucleosome occupancy profile and called dyad positions
- High-precision nucleosome dyad annotations from chemical cleavage method (Brogaard et al., reference 24)

## Outputs

- Precision-recall curve for RoboCOP_MNase dyad predictions (saved as robocop_mnase_pr_curve.png, reproducing Figure 4C and Supplementary Figure S12A)
- Precision and recall values computed over all called dyads at a 50 bp true-positive tolerance

## How to apply

Run RoboCOP in MNase-seq-only mode (multivariate HMM) following the example_robocop.ipynb vignette after compiling librobocop.so via 'bash gccCompile' and setting its absolute path as cshared in config.ini. Extract the called nucleosome dyad positions from the inferred genome-wide probabilistic occupancy profile for S. cerevisiae (paired-end MNase-seq from GEO accession GSE153609). Score each called dyad as a true positive if it lies within 50 bp of a dyad from the high-precision chemical cleavage (Brogaard et al., 24) annotations, then sweep the dyad probability score threshold to compute precision and recall over all called dyads. Plot the resulting precision-recall curve (reproducing Figure 4C and Supplementary Figure S12A) and save it as robocop_mnase_pr_curve.png. Correct application is confirmed when the dyad probability score peaks precisely at the annotated dyads, showing the predicted positions align closely with the ground-truth chemical-cleavage dyads.

## Related tools

- **RoboCOP** (Multivariate HMM run in MNase-seq-only mode to infer genome-wide probabilistic nucleosome occupancy and called dyad positions that are then PR-evaluated) — https://github.com/HarteminkLab/RoboCOP

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git && cd RoboCOP/robocop && bash gccCompile
```

## Evaluation signals

- Each called dyad is scored as a true positive only if it lies within 50 bp of a Brogaard et al. (24) chemical-cleavage dyad; verify this tolerance is used consistently
- The dyad probability score should peak precisely at the annotated dyads, indicating alignment with ground-truth positions
- The plotted precision-recall curve should reproduce Figure 4C and Supplementary Figure S12A of the RoboCOP paper
- librobocop.so exists at the absolute path set in config.ini (cshared), confirming the compilation and configuration step succeeded
- Precision and recall are computed over all called dyads extracted from the inferred occupancy profile, not a subsample

## Limitations

- RoboCOP performance is markedly better with MNase-seq than ATAC-seq (due to fragment distribution and MNase exonuclease activity reflecting DBF sizes), so PR results may be weaker on ATAC-seq input
- Accuracy of chromatin occupancy profiles improves with deeper sequencing; shallow sequencing depth will degrade the PR curve
- No version numbers are stated for RoboCOP or other tools in the paper, so exact reproducibility of the PR curve may vary across releases

## Evidence

- [other] Score each called dyad as a true positive if it lies within 50 bp of a dyad from the high-precision chemical cleavage (24) annotations: [evidence span withheld — anchor status: not_found]
- [other] RoboCOP_MNase accurately recapitulates annotated nucleosome positions: its dyad probability score peaks precisely at the annotated dyads from reference (24): [evidence span withheld — anchor status: not_found]
