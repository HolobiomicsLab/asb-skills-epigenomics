---
name: partial-correlation-covariate-control
description: Use when a scored sequence set (e.g. FASTA with sequence scores in headers)
  plus a JASPAR motif library must be tested for motif enrichment at each position
  relative to an anchor (TSS, other motifs) while controlling for GC bias.
when_to_use_negative:
- The goal is de novo motif recognition/assembly — MEPP currently only functions with
  a fixed motif library, and de novo motif assembly is left to future work.
- Position-agnostic motif enrichment is sufficient (e.g. ZOOPs-style presence/absence
  tests as in HOMER or MEIRLOP) — the positional partial-correlation machinery adds
  no value there.
- No continuous biological score is available per sequence — the partial Pearson correlation
  requires a scored sequence vector, not binary labels alone.
edam_operation: http://edamontology.org/operation_3659
tools:
- name: MEPP
  role: Implements the GC-controlled partial Pearson correlation, permutation testing,
    and FDR correction that produce the positional enrichment profile
  repo: https://github.com/npdeloss/mepp
- name: MOODS
  role: Sets the motif match threshold used to build the motif occurrence heatmap
    that the partial correlations are computed against (via --pval 0.0001, --pcount
    0.0001, --bg)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:9da2d394f8bfb39e5a514b6d5c7c9d33172a27ef2e6e20204d900b58b7d24025
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  - MOODS
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# partial-correlation-covariate-control

## Summary

Computes MEPP's positional enrichment profile by calculating, at each heatmap position column, the partial Pearson correlation (pXY·Z) of the motif score matrix against the sequence score vector while regressing out per-sequence GC content, then deriving permutation-based confidence intervals and Benjamini-Yekutieli-corrected P-values.

## When to use

Use when a scored sequence set (e.g. FASTA with sequence scores in headers) plus a JASPAR motif library must be tested for motif enrichment at each position relative to an anchor (TSS, other motifs) while controlling for GC bias.

## When NOT to use

- The goal is de novo motif recognition/assembly — MEPP currently only functions with a fixed motif library, and de novo motif assembly is left to future work.
- Position-agnostic motif enrichment is sufficient (e.g. ZOOPs-style presence/absence tests as in HOMER or MEIRLOP) — the positional partial-correlation machinery adds no value there.
- No continuous biological score is available per sequence — the partial Pearson correlation requires a scored sequence vector, not binary labels alone.

## Inputs

- Scored FASTA file where sequence headers take the form '>sequence_name sequence_score'
- Motif matrices file in JASPAR format (e.g. from JASPAR downloads or data/homer.motifs.txt)
- Per-sequence GC content computed from the analyzed sequences (control covariate)
- Output directory path for MEPP results

## Outputs

- positional_profile.tsv: tab-separated table with position, partial correlation, CI bounds, and corrected P-value for every heatmap column
- MEPP positional enrichment plots (png/svg, default 300 DPI) with motif occurrence heatmaps
- Clustering table of MEPP profiles with adjusted positional-correlation P-values

## How to apply

Load the motif occurrence heatmap (one column per position relative to the anchor point) and the sequence score vector, and compute per-sequence GC content from the analyzed sequences. For each position column, compute the partial Pearson correlation with the sequence score vector, using per-sequence GC content as the control covariate Z — this is the mechanism by which MEPP avoids arbitrary threshold selection while controlling for sequence bias. Permute the sequence scores (--perms, default 1000 in the MEPP CLI) to generate null distributions of the partial correlation at every position, deriving a confidence interval (--ci, default 95.0) and a raw P-value per position. Apply multiple testing correction across all position-wise P-values via statsmodels multipletests (default method fdr_by, i.e. Benjamini-Yekutieli, alpha 0.01), optionally with --thoroughmt so all p-values for all motifs at all positions are adjusted simultaneously. Assemble the result into positional_profile.tsv with columns for position, partial correlation, CI bounds, and corrected P-value.

## Related tools

- **MEPP** (Implements the GC-controlled partial Pearson correlation, permutation testing, and FDR correction that produce the positional enrichment profile) — https://github.com/npdeloss/mepp
- **MOODS** (Sets the motif match threshold used to build the motif occurrence heatmap that the partial correlations are computed against (via --pval 0.0001, --pcount 0.0001, --bg))

## Examples

```
mepp --fa scored.fa --motifs data/homer.motifs.txt --out mepp_out --perms 1000 --ci 95.0 --mtmethod fdr_by
```

## Evaluation signals

- positional_profile.tsv exists and contains position/correlation/CI/P-value fields for every heatmap column
- All corrected P-values in positional_profile.tsv fall within [0, 1]
- Correction uses Benjamini-Yekutieli (fdr_by) rather than Benjamini-Hochberg, reflecting dependence between neighboring positions
- Permutation count and CI match CLI settings (default 1000 permutations, 95.0% CI), and p-values are derived from the permutation null at every position
- Profile recapitulates known positional biology, e.g. core promoter element positioning around Drosophila melanogaster TSS or SCL-to-GATA1 motif spacing in K562 cells

## Limitations

- The --perms parameter can lead to significant GPU memory usage, and MEPP relies on Tensorflow datasets/GPU processing unless --nogpu is set
- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work
- No software version or changelog is specified for the repository

## Evidence

- [abstract] outputs a positional enrichment profile of a given TF's binding motif: "We developed motif enrichment positional profiling (MEPP), a novel MEA method that outputs a positional enrichment profile of a given TF's binding motif"
- [intro] The score-based principle avoids arbitrary thresholds while controlling sequence bias: "The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias."
- [readme] Scored FASTA input format: [evidence span withheld — anchor status: external_doc]
- [readme] Permutations for testing and confidence intervals: [evidence span withheld — anchor status: external_doc]
- [readme] Multiple testing method for positional correlation p-values: [evidence span withheld — anchor status: external_doc]
