---
name: triplex-prediction-parameter-sweep
description: 'Use when calibrating or benchmarking triplex (TPX) prediction parameters:
  multiple RNA:DNA interaction peak sets and RNA sequences are available and you need
  to determine which minimal TPX length, masking cut-off or scoring option maximises
  AUC against shuffled genomic negatives.'
when_to_use_negative:
- You need whole-transcript triplex potential classification without parameter tuning
  — TriplexFPP gives triplex lncRNA/DNA-site predictions directly from FASTA, though
  it failed to identify known TPX-forming lncRNAs (CDKN2B-AS1, lncSmad7, MEG3, NEAT1,
  TERC) in this paper's comparison
- Input is a single fixed run where 3plex defaults (minimal length 10, error rate
  20%, guanine rate 40%) are already accepted and no AUC comparison across settings
  is required
- The binding data lack replicates or multiple peak sets, so the peak-filtering factor
  of the sweep cannot be varied and only the default method is testable
edam_operation: http://edamontology.org/operation_0430
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0099
tools:
- name: 3plex
  role: Core TPX prediction tool whose parameters are swept; integrates PATO scanning,
    RNAplfold single-strandedness masking and LongTarget-derived stability scoring
  repo: https://github.com/molinerisLab/3plex
- name: RNAplfold
  role: ViennaRNA predictor supplying per-nucleotide unpaired probabilities used to
    generate ss0/ss10/ss20/ss50 masked RNA variants within the sweep
  repo: https://github.com/molinerisLab/3plex
- name: bedtools
  role: shuffle is used to draw negative genomic regions matched to the positive peak
    length distribution while excluding genome gaps and ENCODE blacklist regions
- name: pROC
  role: R package (v1.17.0.1) computing ROC curves and AUC values for each parameter
    combination, with one-sided Mann-Whitney significance tests
- name: Triplexator
  role: Provides the potential (match-count) score used alongside 3plex stability-based
    scoring in the parameter comparison
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:9cc4651c21197d91cc45dd8a0e4128145903811231d57fd315ca3cee9dd0dd28
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/molinerisLab/3plex
  repo_url: https://github.com/molinerisLab/3plex
  tools:
  - 3plex
  - RNAplfold
  - Triplexator
  - ViennaRNA
  - bedtools
  - pROC
  edam_operation: http://edamontology.org/operation_0430
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# triplex-prediction-parameter-sweep

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A systematic evaluation workflow that sweeps 3plex parameter combinations (minimal TPX length, error rate, guanine rate, consecutive errors, ssRNA masking, repeat filter) against experimental lncRNA binding-site positives versus matched genomic negatives, scoring each configuration by ROC/AUC and testing parameter effects with linear models, ANOVA and permutation tests.

## When to use

Use when calibrating or benchmarking triplex (TPX) prediction parameters: multiple RNA:DNA interaction peak sets and RNA sequences are available and you need to determine which minimal TPX length, masking cut-off or scoring option maximises AUC against shuffled genomic negatives.

## When NOT to use

- You need whole-transcript triplex potential classification without parameter tuning — TriplexFPP gives triplex lncRNA/DNA-site predictions directly from FASTA, though it failed to identify known TPX-forming lncRNAs (CDKN2B-AS1, lncSmad7, MEG3, NEAT1, TERC) in this paper's comparison
- Input is a single fixed run where 3plex defaults (minimal length 10, error rate 20%, guanine rate 40%) are already accepted and no AUC comparison across settings is required
- The binding data lack replicates or multiple peak sets, so the peak-filtering factor of the sweep cannot be varied and only the default method is testable

## Inputs

- ssRNA transcript sequences in FASTA (one sequence per file for the 3plex Docker binary)
- Positive binding-site peak sets (BED) from replicate handling: IDR conservative, IDR optimal, Overlap conservative at SPP q<0.05, Top1000, author-provided
- Reference genome for bedtools shuffle of matched negative regions, excluding genome gaps and ENCODE blacklist regions
- RNAplfold per-nucleotide unpaired probabilities (ViennaRNA, window length 8) for ss0/ss10/ss20/ss50 masking variants
- Parameter grid definition: minimal length (16/12/10/8), error rate (10%/20%), guanine rate (10%/40%/70%), consecutive errors (1/3), repeat filter (on/off)

## Outputs

- Per-combination TPX match tables (*_tpx.stability.gz) with TFO:TTS coordinates, potential score, error rate, motif, guanine rate and stability
- ROC curves and AUC values per parameter combination from pROC, with Mann-Whitney p-values and BH-adjusted FDR
- ANOVA Panoval and permutation-based relevance probability (10,000 permutations) for the swept parameter
- Parameter-effect summaries (e.g. ss_masking_auc_comparison.csv, lncRNA_tpx_classification_table.tsv) with AUC per lncRNA x parameter level

## How to apply

Build positive sets from peak filtering methods (IDR conservative, IDR optimal, Overlap conservative filtered at SPP q<0.05, Top1000, author-provided peaks) and generate matched negatives with bedtools shuffle, preserving the positive length distribution while excluding genome gaps and ENCODE blacklist regions. Run 3plex over the parameter grid — minimal TPX length 16/12/10/8 (default 10, allowed >=5), error rate 10%/20%, guanine rate 10%/40%/70%, consecutive errors 1/3, ssRNA masking ss0/ss10/ss20/ss50 with RNAplfold window length 8, and repeat filter on/off — using the Triplexator potential score or stability-based scoring (Normalised/Best stability). Compute ROC curves and AUC per parameter combination with the pROC R package (v1.17.0.1), assessing significance with one-sided Mann-Whitney tests and Benjamini-Hochberg correction. To test a parameter's relevance, fit a full linear model (e.g. AUC ~ peak filtering method + ssRNA + n peaks + minimal length) versus a nested model without the term of interest via lm and ANOVA in R, then run a randomisation test with 10,000 permutations of the AUC-versus-parameter association, counting how many permuted ANOVA p-values are <= the observed one.

## Related tools

- **3plex** (Core TPX prediction tool whose parameters are swept; integrates PATO scanning, RNAplfold single-strandedness masking and LongTarget-derived stability scoring) — https://github.com/molinerisLab/3plex
- **RNAplfold** (ViennaRNA predictor supplying per-nucleotide unpaired probabilities used to generate ss0/ss10/ss20/ss50 masked RNA variants within the sweep) — https://github.com/molinerisLab/3plex
- **bedtools** (shuffle is used to draw negative genomic regions matched to the positive peak length distribution while excluding genome gaps and ENCODE blacklist regions)
- **pROC** (R package (v1.17.0.1) computing ROC curves and AUC values for each parameter combination, with one-sided Mann-Whitney significance tests)
- **Triplexator** (Provides the potential (match-count) score used alongside 3plex stability-based scoring in the parameter comparison)

## Examples

```
docker run -u $(id -u):$(id -g) -it --rm -v $PWD:$PWD imolineris/3plex:v1.0 $PWD/ssRNA.fa $PWD/dsDNA.fa $PWD/results_3plex -l 8 -e 20 -g 40 -c 1 -s 10
```

## Evaluation signals

- ANOVA comparison of full vs nested linear models yields Panoval < 2.2e-16 for the swept parameter (e.g. minimal TPX length)
- Permutation test with 10,000 permutations estimates the parameter relevance probability below 1e-04
- AUC values fall within [0, 1] for every parameter combination and each lncRNA x masking-level row of the comparison table
- Reproduced effect direction matches the paper: minimal lengths 8 and 10 perform equally best, the default 16 is worst (average AUC reduced by ~0.1), and 12 is intermediate; masking improves AUC only for a subset of lncRNAs (3 of 7: MEG3, Meg3, lncSmad7)
- Negative regions from bedtools shuffle match the positive length distribution and exclude genome gaps and ENCODE blacklist regions

## Limitations

- Minimal TPX length of 6 gave no further improvement when tested in a subsample of the parameter space
- Allowing 1 vs up to 3 consecutive errors showed no significant differences in average AUC, so that parameter axis adds little signal
- Only 2 of 7 TPX-validated lncRNAs had IDR conservative peaks; the other 5 had insufficient replicate reproducibility to apply that filtering method
- A 3plex Docker run exits with status 139 on Linux kernels >= 4.8 unless vsyscall is enabled at startup

## Evidence

- [results] Minimal TPX length significantly affects prediction accuracy (p-value<1e-04, permutation test): [evidence span withheld — anchor status: not_found]
- [results] the lengths 8 and 10 performing equally best; the default cut-off of 16 was the worst: [evidence span withheld — anchor status: not_found]
- [methods] Compute ROC curves and AUC values for each parameter combination with the pROC R package (v1.17.0.1): [evidence span withheld — anchor status: not_found]
- [readme] -l N, --min_length N    Minimum triplex length required. Allowed values: N>=5. [ Default: 10 ]: [evidence span withheld — anchor status: external_doc]
- [methods] Run a randomisation test with 10000 permutations of the AUC-versus-parameter association: [evidence span withheld — anchor status: not_found]
