---
name: nucleosome-positioning-aupr-evaluation
description: Use when multiple nucleosome-positioning methods have been run on the
  same ATAC-seq and/or MNase-seq data and their predicted nucleosome calls must be
  compared on an identical validation set using AUPR.
when_to_use_negative:
- Only a single method's nucleosome predictions are available with no baseline calls
  (e.g. NucleoATAC or DANPOS2) to compare against
- The validation annotations differ between methods being compared — AUPR is only
  interpretable when all methods are scored on the identical validation set
- The goal is comparing DNA-binding-factor (TF) binding site predictions rather than
  nucleosome positioning; that requires ChIP-chip/ChIP-exo/ORGANIC annotations instead
edam_operation: http://edamontology.org/operation_1815
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: RoboCOP
  role: Multivariate state space model producing nucleosome occupancy predictions
    from ATAC-seq alone and from MNase-seq combined with ATAC-seq; its outputs are
    evaluated by AUPR against chemical-cleavage annotations
  repo: https://github.com/HarteminkLab/RoboCOP
- name: NucleoATAC
  role: Baseline nucleosome-positioning method that uses ATAC-seq data; its calls
    are compared head-to-head with RoboCOP_ATAC (Supplementary Figure S12A)
- name: DANPOS2
  role: Baseline MNase-seq-based nucleosome-positioning method whose calls are compared
    with RoboCOP_MNase+ATAC (Supplementary Figure S12)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:28f0e9937149671c73e2c4e0ddd98f6b26877e433013bc78bce25d6d9e62bf3a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - DANPOS2
  - GCC
  - NucleoATAC
  - RoboCOP
  edam_operation: http://edamontology.org/operation_1815
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# nucleosome-positioning-aupr-evaluation

## Summary

Benchmarking protocol for comparing predicted nucleosome positions from multiple chromatin-occupancy methods (e.g. RoboCOP with ATAC-seq or MNase+ATAC, DANPOS2, NucleoATAC) by computing nucleosome-position AUPR against a common high-precision validation set, such as chemical-cleavage annotations in S. cerevisiae.

## When to use

Use when multiple nucleosome-positioning methods have been run on the same ATAC-seq and/or MNase-seq data and their predicted nucleosome calls must be compared on an identical validation set using AUPR.

## When NOT to use

- Only a single method's nucleosome predictions are available with no baseline calls (e.g. NucleoATAC or DANPOS2) to compare against
- The validation annotations differ between methods being compared — AUPR is only interpretable when all methods are scored on the identical validation set
- The goal is comparing DNA-binding-factor (TF) binding site predictions rather than nucleosome positioning; that requires ChIP-chip/ChIP-exo/ORGANIC annotations instead

## Inputs

- Nucleosome position predictions from RoboCOP run on ATAC-seq data alone (GEO GSE66386)
- Nucleosome position predictions from RoboCOP run on MNase-seq (GEO GSE153609) combined with ATAC-seq (GSE66386)
- Nucleosome position calls from NucleoATAC run on the same ATAC-seq data
- Nucleosome position calls from DANPOS2 run on the same MNase-seq data
- High-precision chemical-cleavage nucleosome validation annotations

## Outputs

- nucleosome_aupr_scores.csv containing nucleosome-position AUPR values for RoboCOP (ATAC-only and MNase+ATAC), DANPOS2, and NucleoATAC
- supplementary_figure_S12_aupr_comparison.png reproducing the Supplementary Figure S12 layout comparing the four methods

## How to apply

Run each competing method on the same input data first: RoboCOP with ATAC-seq alone (GEO GSE66386), RoboCOP with MNase-seq (GSE153609) combined with ATAC-seq, NucleoATAC on the same ATAC-seq data, and DANPOS2 on the same MNase-seq data. Before evaluation, clone the RoboCOP repository, compile the shared library with 'bash gccCompile' to generate librobocop.so, and set the 'cshared' entry in config.ini to its absolute path so the RoboCOP engine can be invoked. Then compute nucleosome-position AUPR for all four method outputs against the chemical-cleavage validation annotations, ensuring every method is scored on the identical validation set. Write the per-method scores to nucleosome_aupr_scores.csv and render the Supplementary Figure S12 comparison (RoboCOP_ATAC vs NucleoATAC; RoboCOP_MNase+ATAC vs DANPOS2) as supplementary_figure_S12_aupr_comparison.png. The evaluation succeeds only when all AUPR values are computed on the same validation annotations and the figure layout reproduces Supplementary Figure S12.

## Related tools

- **RoboCOP** (Multivariate state space model producing nucleosome occupancy predictions from ATAC-seq alone and from MNase-seq combined with ATAC-seq; its outputs are evaluated by AUPR against chemical-cleavage annotations) — https://github.com/HarteminkLab/RoboCOP
- **NucleoATAC** (Baseline nucleosome-positioning method that uses ATAC-seq data; its calls are compared head-to-head with RoboCOP_ATAC (Supplementary Figure S12A))
- **DANPOS2** (Baseline MNase-seq-based nucleosome-positioning method whose calls are compared with RoboCOP_MNase+ATAC (Supplementary Figure S12))

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git; cd RoboCOP/robocop; bash gccCompile; ls "`pwd`/librobocop.so"
```

## Evaluation signals

- All four methods' AUPR values are computed against the identical chemical-cleavage validation annotation set
- nucleosome_aupr_scores.csv contains AUPR entries for RoboCOP (ATAC-only), RoboCOP (MNase+ATAC), DANPOS2, and NucleoATAC
- Rendered figure reproduces the Supplementary Figure S12 layout with RoboCOP_ATAC compared against NucleoATAC and RoboCOP_MNase+ATAC compared against DANPOS2
- RoboCOP run with ATAC-seq alone finds more nucleosomes than NucleoATAC, consistent with Supplementary Figure S12A
- RoboCOP positions many more nucleosomes than NucleoATAC using the same ATAC-seq data, as stated in the article discussion

## Limitations

- NucleoATAC reports only a small fraction of genomic nucleosomes (~20%), so comparisons with RoboCOP_ATAC are affected by this reduced call set
- MNase-seq gives markedly better RoboCOP performance than ATAC-seq, because genome-wide fragment distribution and MNase exonuclease activity reflect DBF sizes; ATAC-only comparisons are therefore expected to show lower accuracy
- No version numbers are stated for any of the tools (RoboCOP, NucleoATAC, DANPOS2), so exact reproducibility of AUPR values may depend on tool versions

## Evidence

- [supplementary] RoboCOP run with ATAC-seq alone finds more nucleosomes than NucleoATAC, another method that incorporates ATAC-seq to find nucleosome positions: [evidence span withheld — anchor status: not_found]
