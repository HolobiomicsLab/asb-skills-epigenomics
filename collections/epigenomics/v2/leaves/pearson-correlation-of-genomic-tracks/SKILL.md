---
name: pearson-correlation-of-genomic-tracks
description: Use when you have CscoreTool-M cscore.txt or cscore.bedgraph sub-compartment
  probabilities and an independent track (NAD-seq, Lamin-B1 DamID) that can be aggregated
  over the same 100 kb windows, and you want to quantify agreement.
when_to_use_negative:
- The tracks are not aggregated over the same equal-length Rabl windows used by CscoreTool-M,
  so per-window correlation would be misaligned.
- The input is a binary sub-compartment annotation rather than continuous probability
  scores — the CscoreTool-M scores are probabilities (bedgraph c-scores sum to 1 per
  region), which is what makes Pearson correlation interpretable.
- 'Regions with no Hi-C reads: the bedgraph omits regions with no reads, so treating
  missing windows as zeros instead of dropping them will distort the correlation.'
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CscoreTool-M
  role: Produces the per-window sub-compartment probability cscores (cscore.txt /
    cscore.bedgraph) that are correlated against NAD-seq and Lamin-B1 DamID tracks
  repo: https://github.com/scoutzxb/CscoreTool-M
- name: pandas
  role: Aggregates NAD-seq and Lamin-B1 DamID signal over the same 100 kb windows
    and computes the Pearson correlations in Python
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:ee3cb4f94b177c3303c6e8ed2201c3bdd72347dfd88d26ffbf3de7857a6d0c0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  - HOMER
  - g++
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# pearson-correlation-of-genomic-tracks

## Summary

Computes Pearson correlations between CscoreTool-M per-window sub-compartment probability scores and independent genomic tracks (NAD-seq, Lamin-B1 DamID) aggregated over the same 100 kb Rabl windows, to validate that a sub-compartment (e.g., C6_6) corresponds to a nuclear-body domain such as NADs.

## When to use

Use when you have CscoreTool-M cscore.txt or cscore.bedgraph sub-compartment probabilities and an independent track (NAD-seq, Lamin-B1 DamID) that can be aggregated over the same 100 kb windows, and you want to quantify agreement.

## When NOT to use

- The tracks are not aggregated over the same equal-length Rabl windows used by CscoreTool-M, so per-window correlation would be misaligned.
- The input is a binary sub-compartment annotation rather than continuous probability scores — the CscoreTool-M scores are probabilities (bedgraph c-scores sum to 1 per region), which is what makes Pearson correlation interpretable.
- Regions with no Hi-C reads: the bedgraph omits regions with no reads, so treating missing windows as zeros instead of dropping them will distort the correlation.

## Inputs

- CscoreTool-M per-window sub-compartment probability file (XXX_cscore.txt) from a 6-sub-compartment mESC run
- Normalized cscore bedgraph (XXX_cscore.bedgraph) whose c-scores sum to 1 per genomic region
- NAD-seq enrichment signal (Bizhanova et al. 2020)
- Lamin-B1 DamID signal (Peric-Hupkes et al. 2010)
- Rabl-format equal-length 100 kb window BED defining the analysis bins

## Outputs

- Per-window merged table of sub-compartment probabilities and track enrichments (100 kb bins)
- Pearson correlation values per sub-compartment/track pair (e.g., C6_NADseq_LaminB1_correlations.tsv)

## How to apply

First ensure both signals live on identical genomic bins: CscoreTool-M was run with an equal-length Rabl-format BED (fourth column = Rabl position between centromere and telomere) and 100 kb windows for the mESC analysis. Extract the per-window probability column of interest from the cscore output (e.g., C6_6 for nucleolus-associated domains) and aggregate NAD-seq (Bizhanova et al. 2020) and Lamin-B1 DamID (Peric-Hupkes et al. 2010) enrichment over the same 100 kb bins with pandas/numpy. Compute the Pearson correlation of the probability column against each track's per-window enrichment. Interpret by comparing across sub-compartments: C6_6 correlates strongly with NAD-seq (reported 0.88) but more weakly with Lamin-B1 DamID (reported 0.567), which is the evidence that C6_6 is a distinct NAD-corresponding sub-compartment rather than a generic lamina-associated compartment. Write the result table (e.g., C6_NADseq_LaminB1_correlations.tsv) and check the reported values are reproduced before trusting downstream conclusions.

## Related tools

- **CscoreTool-M** (Produces the per-window sub-compartment probability cscores (cscore.txt / cscore.bedgraph) that are correlated against NAD-seq and Lamin-B1 DamID tracks) — https://github.com/scoutzxb/CscoreTool-M
- **pandas** (Aggregates NAD-seq and Lamin-B1 DamID signal over the same 100 kb windows and computes the Pearson correlations in Python)

## Examples

```
python -c "import pandas as pd; d=pd.read_csv('C6_cscore.txt',sep='\t'); print(d[['C6_6','NADseq']].corr(method='pearson').iloc[0,1])"
```

## Evaluation signals

- Pearson correlation of C6_6 against NAD-seq enrichment over 100 kb windows reproduces the reported 0.88.
- C6_6 against Lamin-B1 DamID yields a much lower correlation (reported 0.567), confirming C6_6 is a distinct NAD sub-compartment rather than a LAD.
- Normalized cscore values sum to 1 for each genomic region, as the bedgraph guarantees.
- Window coordinates of the track aggregation exactly match the equal-length Rabl BED bins used for the CscoreTool-M run.
- Windows with no Hi-C reads are excluded (absent from the bedgraph) rather than imputed as zero enrichment.

## Limitations

- Correlation strength is window-size dependent: the reported values (0.88 with NAD-seq) were obtained on 100 kb windows and may change at other bin sizes.
- A sub-compartment's probability score correlates with lamina/nucleolus association only to the extent Hi-C structure reflects it — C6_6 correlates far better with NAD-seq (0.88) than with Lamin-B1 DamID, so a high correlation with one track does not generalize to the other.
- Blacklist regions and regions with no reads are excluded from the CscoreTool-M analysis, so correlations are computed only on the retained windows.

## Evidence

- [abstract] The compartment scores inferred using CscoreTool-M represents the probability of a genomic region locating in a specific sub-compartment.: "The compartment scores inferred using CscoreTool-M represents the probability of a genomic region locating in a specific sub-compartment."
- [methods] XXX_cscore.bedgraph This is the bedgraph file made for visualization. Ths c-scores are normalized to add up to 1 for each genomic region.: "XXX_cscore.bedgraph This is the bedgraph file made for visualization. Ths c-scores are normalized to add up to 1 for each genomic region."
- [readme] windows.bed This file is to specify the genomic windows to analyze. It should be an equal-length bed file with the fourth column being the Rabl position: "windows.bed This file is to specify the genomic windows to analyze. It should be an equal-length bed file with the fourth column being the Rabl position"
