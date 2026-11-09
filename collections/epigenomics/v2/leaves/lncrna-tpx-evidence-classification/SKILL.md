---
name: lncrna-tpx-evidence-classification
description: Use when lncRNAs have experimentally determined genomic binding sites
  (e.g. ChIRP/CHART/RADICL-seq peaks) but no prior triplex-mechanism evidence, and
  the goal is to quantify statistical support for DNA binding via TPX formation.
when_to_use_negative:
- When the lncRNA already has experimentally validated triplex evidence — the skill
  targets lncRNAs lacking prior TPX evidence
- When binding data are raw FASTQ only — peaks must first be called and filtered (e.g.
  IDR conservative) before ROC/AUC evaluation
- When no matched negative regions can be generated (e.g. targets cover most of the
  mappable genome), since AUC enrichment testing requires positives vs shuffled controls
edam_operation: http://edamontology.org/operation_0475
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3511
- http://edamontology.org/topic_0099
tools:
- name: 3plex
  role: Predicts ssRNA:dsDNA triplexes on binding peaks using PATO scanning, stability-based
    scoring, and RNAplfold single-strandedness masking
  repo: https://github.com/molinerisLab/3plex
- name: RNAplfold
  role: ViennaRNA tool predicting RNA secondary structure; -u option masks low-accessibility
    nucleotides at ss10/ss20/ss50 cut-offs before 3plex scanning
- name: bedtools
  role: shuffle generates matched negative genomic regions preserving positive-peak
    length distribution while excluding positives, genome gaps and ENCODE blacklist
    regions
- name: pROC
  role: R package used to compute ROC curves and AUC values for positive peaks versus
    negative regions
- name: Triplexator
  role: Its potential score is computed alongside 3plex stability-based scores (Normalised
    stability / Best stability) for comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:6cdcbc959f67040c50dd1f2f71def1f89933a1ce813a21d1138163282da7ee26
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
  edam_operation: http://edamontology.org/operation_0475
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3511
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# lncRNA-tpx-evidence-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies lncRNAs as triplex (TPX) DNA-binders by running 3plex over their experimental binding peaks versus matched negative regions, then aggregating per-lncRNA ROC/AUC evidence into tiered classification (e.g. AUC>0.8 strong, AUC>0.7 good, FDR<e-10).

## When to use

Use when lncRNAs have experimentally determined genomic binding sites (e.g. ChIRP/CHART/RADICL-seq peaks) but no prior triplex-mechanism evidence, and the goal is to quantify statistical support for DNA binding via TPX formation.

## When NOT to use

- When the lncRNA already has experimentally validated triplex evidence — the skill targets lncRNAs lacking prior TPX evidence
- When binding data are raw FASTQ only — peaks must first be called and filtered (e.g. IDR conservative) before ROC/AUC evaluation
- When no matched negative regions can be generated (e.g. targets cover most of the mappable genome), since AUC enrichment testing requires positives vs shuffled controls

## Inputs

- lncRNA sequences in FASTA format (ssRNA.fa, one sequence per file for 3plex)
- Binding peak sets reprocessed with the ENCODE ChIP-seq pipeline (IDR conservative, IDR optimal, Overlap conservative q<0.05, Top1000, author-provided peaks) in BED format
- Reference genome with gaps and ENCODE blacklist regions for negative-region generation

## Outputs

- lncRNA_tpx_classification_table.tsv with per-lncRNA AUC and evidence tiers
- Per-parameter-set ROC curves and AUC values (pROC)
- 3plex *_tpx.stability.gz tab-delimited TFO:TTS match files with Score, Error_rate, Guanine_rate, Stability columns
- Benjamini-Hochberg-corrected one-sided Mann-Whitney p-values per lncRNA

## How to apply

Retrieve lncRNA sequences and their uniformly re-analysed binding peak sets (ENCODE ChIP-seq pipeline; IDR conservative, IDR optimal, Overlap conservative q<0.05, Top1000, author-provided peaks). Predict RNA secondary structure with RNAplfold from ViennaRNA (-u 8) and mask low-accessibility nucleotides at ss10/ss20/ss50 cut-offs, keeping ss0 as unmasked reference. Run 3plex over the parameter space (minimal TPX length 8/10, error rate 10%/20%, guanine rate 10%/40%/70%, consecutive errors 1/3, repeat filter on/off) using Normalised stability / Best stability scores alongside the Triplexator potential score. Generate matched negatives with bedtools shuffle, preserving positive-peak length distribution while excluding positive regions, genome gaps and ENCODE blacklist regions. Compute ROC curves and AUCs with the pROC R package, test significance with one-sided Mann-Whitney tests and Benjamini-Hochberg correction, then aggregate per-lncRNA AUC evidence into a classification table (lncRNA_tpx_classification_table.tsv).

## Related tools

- **3plex** (Predicts ssRNA:dsDNA triplexes on binding peaks using PATO scanning, stability-based scoring, and RNAplfold single-strandedness masking) — https://github.com/molinerisLab/3plex
- **RNAplfold** (ViennaRNA tool predicting RNA secondary structure; -u option masks low-accessibility nucleotides at ss10/ss20/ss50 cut-offs before 3plex scanning)
- **bedtools** (shuffle generates matched negative genomic regions preserving positive-peak length distribution while excluding positives, genome gaps and ENCODE blacklist regions)
- **pROC** (R package used to compute ROC curves and AUC values for positive peaks versus negative regions)
- **Triplexator** (Its potential score is computed alongside 3plex stability-based scores (Normalised stability / Best stability) for comparison)

## Examples

```
docker run -u `id -u`:`id -g` -it --rm -v $PWD:$PWD imolineris/3plex:v1.0 $PWD/ssRNA.fa $PWD/dsDNA.fa $PWD/results_3plex -l 10 -e 20 -c 1 -g 40 -s 20
```

## Evaluation signals

- Reproduced lncRNA_tpx_classification_table.tsv must contain the reported evidence tiers: AUC>0.8 for AK156552.1, HAND2-AS1 and MALAT1; AUC>0.7 for AL109615.3, 7SK, 1200007C13Rik and CPEB2-DT
- Significant p-values (FDR<e-10, one-sided Mann-Whitney with Benjamini-Hochberg correction) for lncRNAs classified as strong/good
- Negative regions from bedtools shuffle must match positive-peak length distribution and exclude positive regions, genome gaps and ENCODE blacklist regions
- IDR conservative peaks should be preferred as positives (article reports +0.09 average AUC vs published peak sets); detectable enrichment must be observed for strong/good tiers
- 3plex output *_tpx.stability.gz must contain the documented columns (Sequence_ID, TFO_start, TFO_end, Duplex_ID, TTS_start, TTS_end, Score, Error_rate, Errors, Motif, Strand, Orientation, Guanine_rate, Stability)

## Limitations

- Only 2 of 7 TPX-validated lncRNAs had IDR conservative peaks; the other 5 showed insufficient replicate reproducibility to apply this filtering method
- Minimal TPX length of 6 gave no further improvement, and allowing 1 vs up to 3 consecutive errors made no significant difference in average AUC
- 3plex and TriplexFPP predictions showed no good agreement; TriplexFPP missed experimentally known TPX-forming lncRNAs including CDKN2B-AS1, lncSmad7, MEG3, NEAT1 and TERC
- TPXs tend to be shorter and more degenerated than previously expected, so overly strict length/error constraints may miss real triplexes

## Evidence

- [results] Applying 3plex to the binding sites of 20 lncRNAs lacking experimental TPX evidence, the authors found 3 lncRNAs with AUC above 0.8 (AK156552.1, HAND2-AS1, MALAT1) and 4 lncRNAs with AUC above 0.7: [evidence span withheld — anchor status: not_found]
- [methods] Predict RNA secondary structure with RNAplfold from ViennaRNA (-u option, window length 8) and mask low-accessibility nucleotides at the ss10, ss20 or ss50 cut-offs: [evidence span withheld — anchor status: not_found]
- [methods] Generate matched negative genomic regions with bedtools shuffle, preserving the positive peaks' length distribution and excluding positive regions, genome gaps and ENCODE blacklist regions: [evidence span withheld — anchor status: not_found]
- [methods] Compute ROC curves and AUC values for positive versus negative regions with the pROC R package, assessing significance with one-sided Mann-Whitney tests and Benjamini-Hochberg correction: [evidence span withheld — anchor status: not_found]
- [readme] docker run -u `id -u`:`id -g` -it --rm -v $PWD:$PWD imolineris/3plex:v1.0 $PWD/ssRNA.fa $PWD/dsDNA.fa $PWD/results_3plex: [evidence span withheld — anchor status: external_doc]
