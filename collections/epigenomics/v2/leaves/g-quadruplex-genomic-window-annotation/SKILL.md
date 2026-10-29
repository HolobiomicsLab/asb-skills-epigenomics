---
name: g-quadruplex-genomic-window-annotation
description: Use when nanopore (FAST5/Albacore/Tombo) data for NA12878 must be annotated
  with G4 motif windows and per-window translocation signals, or when called G4 novelties
  need Quadron weak/strong stability labels.
when_to_use_negative:
- Input is simulated translocation data rather than experimental NA12878 nanopore
  reads — the window annotation pipeline over non-B DNA DB motifs and Tombo reads
  applies to the experimental workflow.
- You only need novelty scores for B-DNA vs non-B windows and already have the translocation
  signal windows computed — re-running window extraction is unnecessary.
- Sequences are not G-quadruplex or other non-B motifs from the non-B DB, so no motif
  windows can be anchored.
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: Tombo
  role: Re-squiggles Albacore FAST5 output against hg38 to give base-level current
    measurements needed for per-window translocation times
- name: Albacore
  role: Basecalls raw ONT current into FAST5/FASTQ with an event table describing
    the DNA context in the nanopore
- name: Quadron
  role: Scores each called G4 locus (with G4-seq mismatch scores) to produce stability
    scores binarized as weak (<=19) or strong (>19)
- name: pandas
  role: Assembles the Table 1-style weak/strong stability count table and writes table1_stability_counts.csv
- name: numpy
  role: Numerical handling of window signals and count arrays alongside pandas
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:6599503632278f1d18625f585e9cc2dccc194d67173ad4baa2382b92acae10ba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Albacore
  - Python
  - Quadron
  - Tombo
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# G-quadruplex genomic window annotation

## Summary

Annotates genomic windows around G-quadruplex (G4) motifs from the non-B DNA DB and computes per-window ONT translocation signals, optionally scoring called G4 loci with Quadron stability scores (weak <= 19 vs strong > 19) for stability comparisons across novelty detection methods.

## When to use

Use when nanopore (FAST5/Albacore/Tombo) data for NA12878 must be annotated with G4 motif windows and per-window translocation signals, or when called G4 novelties need Quadron weak/strong stability labels.

## When NOT to use

- Input is simulated translocation data rather than experimental NA12878 nanopore reads — the window annotation pipeline over non-B DNA DB motifs and Tombo reads applies to the experimental workflow.
- You only need novelty scores for B-DNA vs non-B windows and already have the translocation signal windows computed — re-running window extraction is unnecessary.
- Sequences are not G-quadruplex or other non-B motifs from the non-B DB, so no motif windows can be anchored.

## Inputs

- Tombo re-squiggled NA12878 nanopore reads (FAST5-derived event tables aligned to hg38)
- G4 motif positions extracted from the non-B DNA DB (https://nonb-abcc.ncifcrf.gov/apps/site/default)
- hg38 reference genome (hg38.fa)
- G4 novelty calls per detection method from task_005 output over experimental NA12878 windows
- G4-seq mismatch scores for Quadron

## Outputs

- Annotated genomic windows around G4 motifs with per-window translocation signal (seconds)
- Per-locus Quadron stability scores for each called G4
- table1_stability_counts.csv: weak/strong G4 stability counts per novelty detection method

## How to apply

Extract G4 motif positions from the non-B DNA DB, fix windows of a chosen length (the paper evaluates window sizes 50 and 100) around each motif, and extend positions on the opposite strand. Find high-quality Tombo-mapped reads overlapping the windows and matched motif-free regions, then compute the translocation signal on non-overlapping windows as the time duration in seconds (number of current measurements divided by the ONT sampling rate). For stability annotation of called G4 novelties (from GoFAE-DND, isolation forest, local outlier factor, one-class SVM), run Quadron on each called G4 locus supplying G4-seq mismatch scores, binarize each scored G4 as weak (stability score <= 19) or strong (score > 19), and aggregate weak/strong counts per method with pandas into a Table 1-style CSV. The rationale is that motif presence alone is not sufficient evidence of a non-B structure, so windows are annotated jointly with translocation-time signal and, for G4s, an experimentally calibrated stability score.

## Related tools

- **Tombo** (Re-squiggles Albacore FAST5 output against hg38 to give base-level current measurements needed for per-window translocation times)
- **Albacore** (Basecalls raw ONT current into FAST5/FASTQ with an event table describing the DNA context in the nanopore)
- **Quadron** (Scores each called G4 locus (with G4-seq mismatch scores) to produce stability scores binarized as weak (<=19) or strong (>19))
- **pandas** (Assembles the Table 1-style weak/strong stability count table and writes table1_stability_counts.csv)
- **numpy** (Numerical handling of window signals and count arrays alongside pandas)

## Examples

```
tombo resquiggle $path/workspace/pass/ hg38.fa --dna --overwrite --basecall-group Basecall_1D_001 --include-event-stdev --processes 48
```

## Evaluation signals

- The reproduced Table 1 must show isolation forest yielding the fewest G4 novelties but the largest share of strong (Quadron score > 19) G4s.
- Local outlier factor must yield zero called G4s, consistent with the paper's Table 1 (LOF did not produce any G4 novelties).
- Every scored G4 must be binarized into exactly one of weak (score <= 19) or strong (score > 19); no missing or overlapping labels.
- Translocation times must equal number of current measurements divided by the ONT sampling rate, computed on non-overlapping windows of the chosen size (50 or 100).
- Window counts must match the six-step preparation: motifs fixed, opposite-strand extension applied, only high-quality reads overlapping windows retained.

## Limitations

- The FAST5 files total over 4TB, so downloading the experimental data (SRR15058166) may be slow.
- Motif presence from the non-B DNA DB is necessary but not sufficient evidence of an actual non-B structure, hence the need for translocation-time and Quadron stability corroboration.
- Local outlier factor produced no G4 novelties at all, so stability annotation for that method is trivially empty.

## Evidence

- [readme] Step 1: Extract motifs postions from [non-B DNA DB]: "Step 1: Extract motifs postions from [non-B DNA DB](https://nonb-abcc.ncifcrf.gov/apps/site/default)."
- [readme] Fix windows of length x around motifs: "Step 2: 
Fix windows of length x around motifs."
- [methods] Translocation time computation: "we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate"
- [readme] Motif sufficiency caveat: "computational methods rely on non-B DNA base motifs, which are necessary but not sufficient indicators of non-B structures."
- [other] Quadron weak/strong binarization and expected Table 1 outcome: [evidence span withheld — anchor status: not_found]
- [other] Table 1 validation: "IF identified the smallest set of novelties, but with the most weakly and strongly stable G4s (Table 1)."
