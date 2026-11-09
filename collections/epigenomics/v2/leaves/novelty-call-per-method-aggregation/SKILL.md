---
name: novelty-call-per-method-aggregation
description: Use when you have per-method G4 novelty calls from GoFAE-DND, isolation
  forest, LOF, or one-class SVM on NA12878 nanopore windows and need to compare how
  weakly vs strongly stable the called G4s are via Quadron scores.
when_to_use_negative:
- The loci of interest are not G-quadruplex motifs — Quadron scores G4 stability specifically,
  not other non-B classes such as short tandem repeats
- Novelty calls per detection method have not yet been generated on the NA12878 windows;
  this skill consumes an existing per-method call set rather than producing one
tools:
- name: Quadron
  role: Scores each called G4 locus using G4-seq mismatch scores to produce a per-locus
    stability score
- name: Python
  role: Binarizes Quadron stability scores into weak/strong classes at the 19-score
    threshold
- name: pandas
  role: Aggregates weak/strong counts per detection method and writes table1_stability_counts.csv
- name: numpy
  role: Numeric handling of stability score arrays during binarization
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:e8b43342a0228473ed498dbf4722d309651722e0174ada0bcbaedf86d09b8b95
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Quadron
  - numpy
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# novelty-call-per-method-aggregation

## Summary

Scores each G4 novelty locus called by the paper's novelty detection methods (GoFAE-DND, isolation forest, local outlier factor, one-class SVM) with Quadron to obtain stability scores, then binarizes loci as weakly stable (score <= 19) or strongly stable (score > 19) and assembles a per-method weak/strong count table reproducing the paper's Table 1.

## When to use

Use when you have per-method G4 novelty calls from GoFAE-DND, isolation forest, LOF, or one-class SVM on NA12878 nanopore windows and need to compare how weakly vs strongly stable the called G4s are via Quadron scores.

## When NOT to use

- The loci of interest are not G-quadruplex motifs — Quadron scores G4 stability specifically, not other non-B classes such as short tandem repeats
- Novelty calls per detection method have not yet been generated on the NA12878 windows; this skill consumes an existing per-method call set rather than producing one

## Inputs

- Per-method G4 novelty calls (GoFAE-DND, isolation forest, local outlier factor, one-class SVM) over experimental NA12878 windows from the prior novelty detection task
- G4 locus coordinates for each called novelty
- G4-seq mismatch scores for Quadron scoring

## Outputs

- Per-locus Quadron stability scores for each called G4
- table1_stability_counts.csv — a Table 1-style per-method count table of weak (<= 19) and strong (> 19) G4 stability classes

## How to apply

Load the G4 novelty calls per detection method from the preceding novelty-detection output over the experimental NA12878 windows. Run Quadron on each called G4 locus, supplying G4-seq mismatch scores, to obtain a per-locus stability score. Binarize each scored G4 as weak (stability score <= 19) or strong (stability score > 19). Aggregate weak/strong counts per method with pandas and write a Table 1-style stability count table to table1_stability_counts.csv. The rationale is that novelty methods differ not only in how many G4 novelties they call but in the physical stability of the loci they flag, so Quadron's sequence-based stability scoring (driven by G4-seq mismatch scores) provides an orthogonal quality check on each detector's calls.

## Related tools

- **Quadron** (Scores each called G4 locus using G4-seq mismatch scores to produce a per-locus stability score)
- **Python** (Binarizes Quadron stability scores into weak/strong classes at the 19-score threshold)
- **pandas** (Aggregates weak/strong counts per detection method and writes table1_stability_counts.csv)
- **numpy** (Numeric handling of stability score arrays during binarization)

## Examples

```
python -c "import pandas as pd; t=pd.read_csv('quadron_scores.csv'); t['stability']=t['score'].map(lambda s:'strong' if s>19 else 'weak'); t.groupby('method')['stability'].value_counts().to_csv('table1_stability_counts.csv')"
```

## Evaluation signals

- Local outlier factor yields zero called G4s, so its row shows no weak or strong counts, consistent with the paper's Table 1
- Isolation forest yields the fewest G4 novelties but the largest share of strong (score > 19) G4s
- Every scored G4 falls into exactly one of the two classes, with the split applied at Quadron stability score 19 (weak <= 19, strong > 19)
- The output CSV contains one weak/strong count pair per detection method that produced novelties (GoFAE-DND, isolation forest, one-class SVM) plus a zero row for LOF

## Limitations

- LOF produced no G4 novelties in the paper's experimental data, so no stability assessment is possible for that method
- The weak/strong cutoff of 19 is specific to Quadron's stability scoring scale and should not be transferred to other stability scorers
- Quadron requires G4-seq mismatch scores as input; without them, per-locus stability cannot be computed

## Evidence

- [full_text] Besides LOF which did not produce any G4 novelties, IF identified the smallest set of novelties, but with the most weakly and strongly stable G4s (Table 1).: "Besides LOF which did not produce any G4 novelties, IF identified the smallest set of novelties, but with the most weakly and strongly stable G4s (Table 1)."
- [readme] Run novelty detection methods including isolation forest, local outlier factor, and one-class SVM: "Afterwards, we can run isolation forests, local outlier factors, and one class SVMs."
- [results] The G4 novelty calls are made over experimental NA12878 windows: "datasets = ['experimental', 'simulated']
window_size = [50, 100]"
