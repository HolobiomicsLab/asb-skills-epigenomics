---
name: roc-auc-binding-site-evaluation
description: Use when you need to quantify whether a triplex-prediction tool (e.g.
  3plex, Triplexator) enriches real RNA:DNA binding peaks over matched negative regions,
  and to tier lncRNAs by AUC evidence.
when_to_use_negative:
- You lack a defined positive peak set or cannot generate length-matched negatives
  — AUC comparison is meaningless without both classes
- You only need raw triplex coordinates/annotations rather than a quantitative enrichment
  assessment of predictions against binding data
- Comparing against TriplexFPP CNN-based classifications, where the source study found
  no good agreement with 3plex predictions and missed known TPX-forming lncRNAs (CDKN2B-AS1,
  lncSmad7, MEG3, NEAT1, TERC)
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: pROC
  role: R package used to compute ROC curves and AUC values for positive versus negative
    regions
- name: bedtools
  role: bedtools shuffle generates matched negative genomic regions preserving positive
    peak length distribution while excluding positives, genome gaps and ENCODE blacklist
    regions
- name: 3plex
  role: Produces the per-region TPX scores (stability-based Normalised/Best stability
    or Triplexator potential score) that are ranked for ROC/AUC evaluation
  repo: https://github.com/molinerisLab/3plex
- name: Triplexator
  role: Baseline TPX prediction tool whose potential score is evaluated alongside
    3plex stability-based scoring in the ROC/AUC framework
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:8430f1c6803e37e8b63185a0f4e17ad19cddbf9b42552f3d3d206eccdd664fc5
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
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# roc-auc-binding-site-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluation strategy that scores triplex (TPX) prediction enrichment of experimentally determined lncRNA binding sites by computing ROC curves and AUC values for positive peaks versus matched shuffled genomic negatives, using the pROC R package with one-sided Mann-Whitney tests and Benjamini-Hochberg correction.

## When to use

Use when you need to quantify whether a triplex-prediction tool (e.g. 3plex, Triplexator) enriches real RNA:DNA binding peaks over matched negative regions, and to tier lncRNAs by AUC evidence.

## When NOT to use

- You lack a defined positive peak set or cannot generate length-matched negatives — AUC comparison is meaningless without both classes
- You only need raw triplex coordinates/annotations rather than a quantitative enrichment assessment of predictions against binding data
- Comparing against TriplexFPP CNN-based classifications, where the source study found no good agreement with 3plex predictions and missed known TPX-forming lncRNAs (CDKN2B-AS1, lncSmad7, MEG3, NEAT1, TERC)

## Inputs

- Positive binding peak sets in BED format (IDR conservative, IDR optimal, Overlap conservative q<0.05, Top1000, or author-provided peaks)
- lncRNA sequences in FASTA format (e.g. GENCODE v32 annotated transcripts)
- Genome assembly with gaps and ENCODE blacklist regions for negative generation
- Per-region TPX prediction scores from 3plex (Normalised stability / Best stability) or the Triplexator potential score

## Outputs

- ROC curves and AUC values per lncRNA per parameter combination (pROC)
- Significance statistics: one-sided Mann-Whitney p-values with Benjamini-Hochberg FDR
- lncRNA_tpx_classification_table.tsv: per-lncRNA AUC evidence tier table

## How to apply

Treat the lncRNA binding peak sets (ENCODE ChIP-seq pipeline re-analysed: IDR conservative, IDR optimal, Overlap conservative q<0.05, Top1000, or author-provided peaks) as positives and generate matched negatives with bedtools shuffle, preserving the positive peaks' length distribution while excluding positive regions, genome gaps and ENCODE blacklist regions. Run the TPX predictor (3plex with stability-based scoring such as Normalised/Best stability, or the Triplexator potential score) over both positive and negative regions, then feed the per-region scores to pROC to compute ROC curves and AUC values. Assess significance with one-sided Mann-Whitney tests and apply Benjamini-Hochberg correction across the parameter scan (minimal TPX length 8/10, error rate 10%/20%, guanine rate 10%/40%/70%, consecutive errors 1/3, repeat filter on/off, masking cut-offs ss0/ss10/ss20/ss50). Aggregate per-lncRNA AUCs into an evidence tier table (e.g. AUC>0.8 strong, AUC>0.7 good) saved as lncRNA_tpx_classification_table.tsv, reporting FDR values alongside AUCs. Prefer IDR conservative peaks, which gave the best average AUC (+0.09 versus published peak sets) in the source study.

## Related tools

- **pROC** (R package used to compute ROC curves and AUC values for positive versus negative regions)
- **bedtools** (bedtools shuffle generates matched negative genomic regions preserving positive peak length distribution while excluding positives, genome gaps and ENCODE blacklist regions)
- **3plex** (Produces the per-region TPX scores (stability-based Normalised/Best stability or Triplexator potential score) that are ranked for ROC/AUC evaluation) — https://github.com/molinerisLab/3plex
- **Triplexator** (Baseline TPX prediction tool whose potential score is evaluated alongside 3plex stability-based scoring in the ROC/AUC framework)

## Examples

```
docker run -u `id -u`:`id -g` -it --rm -v $PWD:$PWD imolineris/3plex:v1.0 $PWD/ssRNA.fa $PWD/dsDNA.fa $PWD/results_3plex -l 10 -e 20 -g 40 -c 1; Rscript -e "roc_obj <- proc::roc(response ~ score, data=regions, direction='<'); auc(roc_obj)"
```

## Evaluation signals

- The reproduced lncRNA_tpx_classification_table.tsv must contain the reported evidence tiers, with each lncRNA assigned a tier supported by its computed AUC and statistical significance
- Known TPX-forming lncRNAs should show AUC above 0.8 (e.g. AK156552.1, HAND2-AS1, MALAT1) or above 0.7 (e.g. AL109615.3, 7SK, 1200007C13Rik, CPEB2-DT) with FDR<e-10
- Negative regions must match positive peak length distribution and exclude positives, genome gaps and ENCODE blacklist regions — verify with bedtools intersection checks
- AUCs should be reproducible under the stated parameter space (min TPX length 8/10, error rate 10%/20%, guanine rate 10%/40%/70%, consecutive errors 1/3, repeat filter on/off, ss0/ss10/ss20/ss50 masking)
- IDR conservative peak sets should yield higher average AUC (+0.09) than published peak sets, as reported by the authors

## Limitations

- Only 2 of 7 lncRNAs had IDR conservative peaks; the other 5 showed insufficient reproducibility among replicates to apply this peak filtering method
- A minimal TPX length of 6 gave no further improvement in a subsample of the parameter space, and allowing 1 vs up to 3 consecutive errors showed no significant AUC differences
- TriplexFPP predictions did not agree well with 3plex and failed to identify experimentally known TPX-forming lncRNAs (CDKN2B-AS1, lncSmad7, MEG3, NEAT1, TERC)

## Evidence

- [methods] pROC-based AUC evaluation: [evidence span withheld — anchor status: not_found]
- [methods] matched negative generation: [evidence span withheld — anchor status: not_found]
- [results] AUC evidence tiers: [evidence span withheld — anchor status: not_found]
- [readme] 3plex scoring options: [evidence span withheld — anchor status: external_doc]
