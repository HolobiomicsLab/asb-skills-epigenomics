---
name: positional-correlation-profiling
description: Use when analyzing scored, anchored DNA sequences (e.g. FASTA headers
  with sequence scores) and you need position-resolved motif enrichment rather than
  a single aggregate enrichment score.
when_to_use_negative:
- You only need a single aggregate motif enrichment score without positional resolution
  — most MEA methods determine enrichment independent of motif position, which may
  suffice
- You need de novo motif discovery/assembly from the sequences — MEPP currently only
  functions with a fixed motif library (e.g. JASPAR or the bundled HOMER/Ohler motif
  files)
- Input sequences lack scores or anchors; the method requires sequence scores in FASTA
  headers and a center/anchor offset (--center) to profile positions against
edam_operation: http://edamontology.org/operation_0445
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3168
tools:
- name: MEPP
  role: 'Primary tool: computes motif heatmaps, GC-controlled partial correlations
    per position, permutation-based CIs/P-values, and plots the positional enrichment
    profile'
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installation channel for MEPP from the GitHub repository
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:3239cdd1449457d7e3605a7d55e02a0dfd2a5491cc7dd7c9e85a175f9e492f47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  - pip
  edam_operation: http://edamontology.org/operation_0445
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# positional-correlation-profiling

## Summary

Reconstructs MEPP's positional enrichment profile: at each heatmap position column relative to an anchor point (e.g. TSS or a motif), it computes the partial Pearson correlation of the motif score matrix against the sequence score vector, controlling for per-sequence GC content, and derives confidence intervals and FDR-corrected P-values via permutation testing.

## When to use

Use when analyzing scored, anchored DNA sequences (e.g. FASTA headers with sequence scores) and you need position-resolved motif enrichment rather than a single aggregate enrichment score.

## When NOT to use

- You only need a single aggregate motif enrichment score without positional resolution — most MEA methods determine enrichment independent of motif position, which may suffice
- You need de novo motif discovery/assembly from the sequences — MEPP currently only functions with a fixed motif library (e.g. JASPAR or the bundled HOMER/Ohler motif files)
- Input sequences lack scores or anchors; the method requires sequence scores in FASTA headers and a center/anchor offset (--center) to profile positions against

## Inputs

- Scored FASTA file with headers of the form '>sequence_name sequence_score' (--fa)
- Motif matrices file in JASPAR format (--motifs)
- Per-sequence GC content computed from the analyzed sequences

## Outputs

- positional_profile.tsv (position, partial correlation, CI bounds, corrected P-value per heatmap column)
- MEPP motif heatmap plots (png/svg, default 300 DPI) colored by the central colormap (default gray_r)
- Clustering table with adjusted positional-correlation P-values

## How to apply

Load the motif occurrence heatmap (one column per position relative to the anchor point) and the sequence score vector, then compute per-sequence GC content from the analyzed sequences. For each position column, calculate the partial Pearson correlation (pXY·Z) of the motif score matrix against the sequence score vector, regressing out per-sequence GC content as the control covariate to handle lower-order nucleotide bias. Permute the sequence scores (--perms, default 1000) to build null distributions of the partial correlation at every position, deriving a confidence interval (--ci, default 95.0) and a raw P-value per position. Apply Benjamini-Yekutieli FDR correction (statsmodels multipletests with method 'fdr_by', default alpha 0.01) across all position-wise P-values to account for dependence between neighboring positions. Assemble the positional profile vector with confidence bands and corrected P-values into positional_profile.tsv with columns for position, partial correlation, CI bounds, and corrected P-value.

## Related tools

- **MEPP** (Primary tool: computes motif heatmaps, GC-controlled partial correlations per position, permutation-based CIs/P-values, and plots the positional enrichment profile) — https://github.com/npdeloss/mepp
- **pip** (Installation channel for MEPP from the GitHub repository) — https://github.com/npdeloss/mepp

## Examples

```
mepp --fa scored.fa --motifs jaspar_motifs.txt --out mepp_out --perms 1000 --ci 95.0 --mtmethod fdr_by
```

## Evaluation signals

- positional_profile.tsv exists and contains position, partial correlation, CI bounds, and corrected P-value fields for every heatmap column
- All corrected P-values in the output table fall within [0, 1]
- Permutation null distributions reproduce stable confidence intervals at the configured --perms (default 1000) and --ci (default 95.0) settings
- Known positional relationships are recapitulated, e.g. core promoter element positioning surrounding Drosophila melanogaster TSS when using the Ohler motif set
- Motif match thresholds set via MOODS with default --pval 0.0001 and --pcount 0.0001 yield non-empty heatmaps without excessive degenerate-sequence rejection (--dgt)

## Limitations

- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work
- Permutation testing can lead to significant GPU memory usage; --gjobs must be set carefully to avoid jobs crowding each other out of GPU memory, causing profile generation to fail
- No software version specified for MEPP and no changelog found for the repository
- Sequences exceeding the degenerate-base percentage (--dgt) are rejected from the analysis, filtering out repeats

## Evidence

- [abstract] outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs) within the analyzed sequences while accounting: "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs)"
- [intro] The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias.: "The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias."
- [readme] pip install git+https://github.com/npdeloss/mepp@main: "To install MEPP, use pip::
    
    pip install git+https://github.com/npdeloss/mepp@main"
- [readme] --perms INTEGER Number of permutations for permutation testing and confidence intervals. Can lead to significant GPU memory usage. Default: 1000: "--perms INTEGER                 Number of permutations for permutation
                                      testing and confidence intervals. Can lead
                                      to"
- [supplementary] The code for MEPP is available from its Github repository at https://github.com/npdeloss/mepp, and can be installed through pip: "The code for MEPP is available from its Github repository at https://github.com/npdeloss/mepp, and can be installed through pip"
