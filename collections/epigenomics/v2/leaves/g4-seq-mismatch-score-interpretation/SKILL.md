---
name: g4-seq-mismatch-score-interpretation
description: Use when you have G4 novelty loci called by one or more novelty detection
  methods over NA12878 nanopore windows and need to characterize whether each called
  G4 is weakly or strongly stable via Quadron G4 stability scores.
when_to_use_negative:
- When no Quadron/G4-seq mismatch scores are available for the called loci — the stability
  score cannot be computed without them
- When the goal is to evaluate novelty detection performance against known labels
  rather than to characterize the stability distribution of the called G4s
- When the loci of interest are non-B structures other than G-quadruplexes (e.g. short
  tandem repeats) — Quadron stability scoring is specific to G4s
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Quadron
  role: Computes per-locus G4 stability scores from the called G4 sequences using
    supplied G4-seq mismatch scores
- name: Python
  role: Runs the scoring/binarization workflow over the novelty calls
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: pandas
  role: Aggregates weak/strong counts per method and writes table1_stability_counts.csv
- name: numpy
  role: Numeric handling of stability score arrays and thresholding (score <= 19 vs
    > 19)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:026cf729211807a57eb64c5488c253dccf4628b0680112b0184170811f4039cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
  tools:
  - Python
  - Quadron
  - numpy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# g4-seq-mismatch-score-interpretation

## Summary

Scores and binarizes G-quadruplex loci by thermodynamic stability using Quadron with G4-seq mismatch scores, classifying each called G4 novelty as weak (stability score <= 19) or strong (stability score > 19) so novelty-detection methods (GoFAE-DND, isolation forest, local outlier factor, one-class SVM) can be compared on the stability profile of their calls.

## When to use

Use when you have G4 novelty loci called by one or more novelty detection methods over NA12878 nanopore windows and need to characterize whether each called G4 is weakly or strongly stable via Quadron G4 stability scores.

## When NOT to use

- When no Quadron/G4-seq mismatch scores are available for the called loci — the stability score cannot be computed without them
- When the goal is to evaluate novelty detection performance against known labels rather than to characterize the stability distribution of the called G4s
- When the loci of interest are non-B structures other than G-quadruplexes (e.g. short tandem repeats) — Quadron stability scoring is specific to G4s

## Inputs

- G4 novelty calls per detection method (GoFAE-DND, isolation forest, local outlier factor, one-class SVM) from novelty detection over experimental NA12878 nanopore translocation-time windows
- G4-seq mismatch scores for the genomic loci corresponding to each called G4
- G4 locus coordinates (from non-B DNA DB motif positions fixed in windows around motifs)

## Outputs

- Per-locus Quadron stability scores for each called G4 novelty
- Table 1-style weak/strong stability count table per novelty detection method, written to table1_stability_counts.csv

## How to apply

First load the per-method G4 novelty calls (GoFAE-DND, isolation forest, local outlier factor, one-class SVM) produced over the experimental NA12878 windows. Run Quadron on each called G4 locus, supplying G4-seq mismatch scores, to obtain a per-locus stability score. Binarize each scored G4 as weak when the stability score is <= 19 and strong when it is > 19, then aggregate weak/strong counts per novelty detection method with pandas and write a Table 1-style stability count table to table1_stability_counts.csv. The rationale is that Quadron's G4-seq mismatch scores capture sequence-dependent G4 stability, allowing the novelty detectors' calls to be judged not just by count but by predicted structural stability, exactly as in the paper's Table 1 comparison.

## Related tools

- **Quadron** (Computes per-locus G4 stability scores from the called G4 sequences using supplied G4-seq mismatch scores)
- **Python** (Runs the scoring/binarization workflow over the novelty calls) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **pandas** (Aggregates weak/strong counts per method and writes table1_stability_counts.csv)
- **numpy** (Numeric handling of stability score arrays and thresholding (score <= 19 vs > 19))

## Examples

```
import pandas as pd; df['class'] = df['quadron_score'].apply(lambda s: 'weak' if s <= 19 else 'strong'); df.groupby(['method','class']).size().unstack(fill_value=0).to_csv('table1_stability_counts.csv')
```

## Evaluation signals

- Local outlier factor must yield zero called G4s, so its weak/strong counts are both zero, consistent with the paper's Table 1
- Isolation forest must yield the smallest set of G4 novelties among methods that call G4s, yet the largest share of strong (stability score > 19) G4s
- Every scored G4 must be binarized into exactly one of weak (score <= 19) or strong (score > 19); no locus should be left unclassified
- Reproduced table1_stability_counts.csv must match the paper's Table 1 structure: one row per novelty detection method with weak and strong G4 counts
- Each called G4 locus should have exactly one Quadron stability score derived from its G4-seq mismatch scores

## Limitations

- Local outlier factor did not produce any G4 novelties in this analysis, so its stability distribution cannot be assessed
- Stability is predicted from sequence (G4-seq mismatch scores via Quadron), not measured directly; calls judged weak/strong are predictions rather than biophysical measurements
- The weak/strong boundary is a fixed threshold (stability score 19) established by the paper; other G4 stability conventions may not transfer

## Evidence

- [other] task_006 workflow: [evidence span withheld — anchor status: not_found]
- [other] task_006 finding: "Besides LOF which did not produce any G4 novelties, IF identified the smallest set of novelties, but with the most weakly and strongly stable G4s (Table 1)."
- [abstract] novelty detection framing: "We formalize non-B detection as a novelty detection problem and develop the GoFAE-DND, an autoencoder that uses goodness-of-fit (GoF) tests as a regularizer."
- [readme] experimental window preparation: "Step 1: Extract motifs postions from [non-B DNA DB]."
- [methods] translocation time computation: "we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate"
