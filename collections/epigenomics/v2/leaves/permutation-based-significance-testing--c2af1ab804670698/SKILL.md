---
name: permutation-based-significance-testing
description: Use when a positional enrichment profile (one correlation per heatmap
  column relative to an anchor point such as a TSS or motif) must carry statistical
  significance and confidence bands robust to GC bias and spatial autocorrelation.
when_to_use_negative:
- The question is genome-wide motif presence/absence enrichment without positional
  information (a ZOOPs-style MEA such as HOMER or MEIRLOP suffices)
- No continuous sequence score is available to permute — MEPP's design requires scored
  sequences across a continuum reflecting biological extremes
- 'De novo motif discovery is required: MEPP currently only functions with a fixed
  motif library'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_3295
tools:
- name: MEPP
  role: Runs the positional correlation analysis with permutation testing, confidence
    intervals, and multiple-testing correction via the mepp CLI
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:5113e9dcece9a091f4ce9d3bdd37397456beae23dc301996536ae641fbc3d17d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# permutation-based-significance-testing

## Summary

Generates null distributions by permuting the sequence score vector to assign per-position confidence intervals and P-values to MEPP's GC-controlled partial Pearson correlations, then corrects position-wise P-values with Benjamini-Yekutieli FDR to handle dependence between neighboring positions.

## When to use

Use when a positional enrichment profile (one correlation per heatmap column relative to an anchor point such as a TSS or motif) must carry statistical significance and confidence bands robust to GC bias and spatial autocorrelation.

## When NOT to use

- The question is genome-wide motif presence/absence enrichment without positional information (a ZOOPs-style MEA such as HOMER or MEIRLOP suffices)
- No continuous sequence score is available to permute — MEPP's design requires scored sequences across a continuum reflecting biological extremes
- De novo motif discovery is required: MEPP currently only functions with a fixed motif library

## Inputs

- Motif occurrence heatmap with one column per position relative to the anchor point (from MEPP motif scanning via MOODS with --pval 0.0001, --pcount 0.0001)
- Sequence score vector (continuous biological activity scores parsed from scored FASTA headers '>sequence_name sequence_score')
- Per-sequence GC content computed from the analyzed sequences (control covariate Z)

## Outputs

- positional_profile.tsv: tab-separated table with columns for position, partial correlation, CI bounds, and Benjamini-Yekutieli-corrected P-value for every heatmap column
- MEPP positional profile plot with confidence bands (png/svg at 300 dpi)

## How to apply

At each position column of the motif occurrence heatmap, compute the partial Pearson correlation pXY·Z between the motif score matrix and the sequence score vector, regressing out per-sequence GC content as the control covariate. Then permute the sequence scores to build a null distribution of the partial correlation at every position, from which a confidence interval (default 95%, --ci) and a raw P-value are derived; the number of permutations is set via --perms (default 1000) and drives Tensorflow GPU memory usage. Apply Benjamini-Yekutieli FDR across position-wise P-values (statsmodels multipletests --mtmethod fdr_by, --mtalpha 0.01) because neighboring positions are dependent; --thoroughmt adjusts all p-values for all motifs at all positions simultaneously, while --non-thoroughmt adjusts only extreme p-values. Rationale: the score-based permutation approach avoids arbitrary threshold selection while controlling for lower-order nucleotide bias.

## Related tools

- **MEPP** (Runs the positional correlation analysis with permutation testing, confidence intervals, and multiple-testing correction via the mepp CLI) — https://github.com/npdeloss/mepp

## Examples

```
mepp --fa scored.fa --motifs motifs.jaspar --out mepp_out --perms 1000 --ci 95.0 --mtmethod fdr_by --mtalpha 0.01
```

## Evaluation signals

- positional_profile.tsv exists and contains position, partial correlation, CI bound, and corrected P-value fields for every heatmap column
- All corrected P-values fall within [0, 1] and CI lower bounds are ≤ point estimates ≤ CI upper bounds
- P-values derive from a permutation null of the GC-controlled partial correlation (permuted sequence scores), not a parametric test
- Multiple-testing method resolves to Benjamini-Yekutieli (fdr_by) with the reported alpha, and thorough testing covers all motifs × all positions by default
- Permutation count matches --perms (default 1000); increasing it should narrow CI bands without shifting point estimates materially

## Limitations

- Higher --perms values 'can lead to significant GPU memory usage' (Tensorflow datasets), constraining permutation count on small GPUs
- Thorough multiple testing is on by default (all motifs × all positions adjusted simultaneously); --non-thoroughmt only adjusts extreme p-values, changing corrected results
- MEPP currently only functions with a fixed motif library; no de novo motif recognition/assembly

## Evidence

- [intro] The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias.: "The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias."
- [abstract] we demonstrate MEPP's utility in determining the sequence positions where motif presence correlates with measures of biological activity: "we demonstrate MEPP's utility in determining the sequence positions where motif presence correlates with measures of biological activity"
- [readme] --perms INTEGER Number of permutations for permutation testing and confidence intervals. Can lead to significant GPU memory usage. Default: 1000: "--perms INTEGER                 Number of permutations for permutation
                                      testing and confidence intervals. Can lead
                                      to"
- [readme] --mtmethod METHOD Multiple testing method for adjusting p-values of positional correlations listed in the clustering table: "--mtmethod METHOD                Multiple testing method for adjusting
                                      p-values of positional correlations listed
                                      in the"
