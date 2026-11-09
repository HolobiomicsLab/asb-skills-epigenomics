---
name: benjamini-yekutieli-fdr-correction
description: Use when a family of P-values is large and potentially dependent (e.g.
  MEPP positional correlations at adjacent sequence positions, or all motifs at all
  positions) and a dependency-robust FDR adjustment is required.
when_to_use_negative:
- When the P-value family is known to be independent — Benjamini-Hochberg FDR is less
  conservative than Benjamini-Yekutieli, which pays a dependence penalty via the harmonic-series
  factor
- When the goal is de novo motif discovery rather than testing enrichment of known
  motifs — MEPP currently only functions with a fixed motif library (e.g. the JASPAR-format
  matrices supplied via --motifs)
- When only a handful of P-values are being corrected, where the BY correction's conservatism
  can drive all adjusted P-values far above any usable alpha (e.g. the 0.01 default)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MEPP
  role: Produces the dependent family of position-wise motif-enrichment P-values and
    applies Benjamini-Yekutieli correction through its --mtmethod (default fdr_by)
    and --mtalpha (default 0.01) CLI options, grounded in statsmodels.stats.multitest.multipletests
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:c6f9ac4c1650d08c2a3e551904e6a3bc222bc592ab6b9830e271d722992ff9f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# benjamini-yekutieli-fdr-correction

## Summary

Applies Benjamini-Yekutieli (fdr_by) false-discovery-rate correction to large, dependent families of P-values — as MEPP does for positional-correlation P-values computed at every heatmap position across motifs, controlling for dependence between neighboring positions.

## When to use

Use when a family of P-values is large and potentially dependent (e.g. MEPP positional correlations at adjacent sequence positions, or all motifs at all positions) and a dependency-robust FDR adjustment is required.

## When NOT to use

- When the P-value family is known to be independent — Benjamini-Hochberg FDR is less conservative than Benjamini-Yekutieli, which pays a dependence penalty via the harmonic-series factor
- When the goal is de novo motif discovery rather than testing enrichment of known motifs — MEPP currently only functions with a fixed motif library (e.g. the JASPAR-format matrices supplied via --motifs)
- When only a handful of P-values are being corrected, where the BY correction's conservatism can drive all adjusted P-values far above any usable alpha (e.g. the 0.01 default)

## Inputs

- Raw (uncorrected) P-values for positional partial-correlation statistics, one per heatmap position column per motif, derived from permutation null distributions (default --perms 1000 permutations)
- MEPP clustering table entries for which p-values are adjusted (CLI options --mtmethod, --mtalpha)
- Choice of multiple-testing scope: thorough (all motifs at all positions simultaneously) vs. non-thorough (only extreme p-values adjusted)

## Outputs

- FDR-adjusted (Benjamini-Yekutieli) P-values appended to the MEPP clustering table
- positional_profile.tsv with columns for position, partial correlation, CI bounds, and corrected P-value

## How to apply

In MEPP, permutation testing generates a raw P-value for the partial Pearson correlation (pXY·Z, controlling for GC content) at every position column of the motif heatmap. Because neighboring positions are correlated, MEPP adjusts these position-wise P-values with Benjamini-Yekutieli via the statsmodels multipletests 'method' parameter, which the CLI exposes as --mtmethod with default 'fdr_by'. By default, thorough multiple testing is enabled so that all P-values for all motifs at all positions are adjusted simultaneously; --mtalpha sets the alpha (FWER/family-wise error rate) parameter, default 0.01. The adjusted P-values are emitted in the clustering table alongside the positional profile (positional_profile.tsv with position, partial correlation, CI bounds, and corrected P-value).

## Related tools

- **MEPP** (Produces the dependent family of position-wise motif-enrichment P-values and applies Benjamini-Yekutieli correction through its --mtmethod (default fdr_by) and --mtalpha (default 0.01) CLI options, grounded in statsmodels.stats.multitest.multipletests) — https://github.com/npdeloss/mepp

## Examples

```
mepp --fa scored.fa --motifs jaspar_motifs.txt --out mepp_out --mtmethod fdr_by --mtalpha 0.01 --thoroughmt
```

## Evaluation signals

- All corrected P-values fall within [0, 1] and are monotonically >= their raw (uncorrected) counterparts
- The clustering table reports fdr_by as the multiple-testing method with alpha 0.01, matching MEPP's documented defaults
- With --thoroughmt enabled, every P-value for every motif at every position is adjusted simultaneously (not just extreme p-values)
- positional_profile.tsv exists and contains position/correlation/CI/P-value fields for every heatmap column
- Positional correlations whose corrected P-values pass alpha correspond to visible enrichment bands in the MEPP heatmap

## Limitations

- Benjamini-Yekutieli is deliberately conservative to handle arbitrary dependence between neighboring positions, so many weak but real positional signals may not survive the alpha threshold (default 0.01)
- The underlying raw P-values depend on the permutation count (--perms, default 1000); MEPP warns this 'Can lead to significant GPU memory usage', and insufficient permutations destabilize the P-values being corrected
- In non-thorough mode (--non-thoroughmt), only extreme p-values are adjusted, so the corrected family may be incomplete relative to thorough multiple testing

## Evidence

- [methods] Benjamini-Yekutieli FDR correction across all position-wise P-values to account for dependence between neighboring positions: [evidence span withheld — anchor status: not_found]
- [readme] README mtmethod option: [evidence span withheld — anchor status: external_doc]
- [readme] README thorough multiple testing default: [evidence span withheld — anchor status: external_doc]
- [readme] README mtalpha option: [evidence span withheld — anchor status: external_doc]
