---
name: synthetic-dna-sequence-motif-implantation
description: 'Use when benchmarking DNA motif-discovery tools on ground-truth data:
  you need FASTA pairs with known implanted IUPAC motifs at controlled frequencies
  and an nCC-based recovery score.'
when_to_use_negative:
- The goal is de novo motif discovery on real biological data (e.g. ChIP-seq peaks)
  rather than controlled benchmarking against known implanted motifs
- Motifs of interest are not representable as IUPAC strings of the fixed length used
  in the implantation (here L=6)
- You need position-level occupancy metrics rather than nucleotide-level recovery
  (nCC) of implanted motifs
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0080
tools:
- name: DiNAMO
  role: Motif-discovery tool run on each synthetic positive/control pair; output motifs
    are scored against implanted motifs with nCC
  repo: https://github.com/bonsai-team/DiNAMO
- name: MEME
  role: Baseline EM-based motif finder whose predictions are benchmarked on the synthetic
    implanted-motif cases (also the output format used by DiNAMO)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:e6436276d766ba0ee86b95d9b831ec61fcd144b5795fec9339c087a661e8b691
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  - MEME
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# synthetic-dna-sequence-motif-implantation

## Summary

Generate synthetic positive/control FASTA benchmark pairs with length-6 IUPAC (degenerate or exact) motifs implanted at controlled frequencies (5% down to 0.5% of initial sequences), then score motif-discovery recovery with the nucleotide level correlation coefficient (nCC).

## When to use

Use when benchmarking DNA motif-discovery tools on ground-truth data: you need FASTA pairs with known implanted IUPAC motifs at controlled frequencies and an nCC-based recovery score.

## When NOT to use

- The goal is de novo motif discovery on real biological data (e.g. ChIP-seq peaks) rather than controlled benchmarking against known implanted motifs
- Motifs of interest are not representable as IUPAC strings of the fixed length used in the implantation (here L=6)
- You need position-level occupancy metrics rather than nucleotide-level recovery (nCC) of implanted motifs

## Inputs

- Positive (signal) FASTA files with length-6 IUPAC motifs implanted at 5%, 4%, 3%, 2%, 1%, 0.5% of initial sequences
- Negative (control) FASTA files matched to each positive dataset
- Catalog of implanted motifs (degenerate and exact types, length 6) as ground truth
- Predicted motifs in MEME format from each evaluated tool

## Outputs

- Per-case nCC scores for each tool's predicted motifs against the implanted motifs
- nCC aggregated by motif type (degenerate vs exact) and implantation frequency
- Comparative benchmark table/ranking of motif-discovery tools

## How to apply

Implant IUPAC motifs of length L=6 into positive datasets at six frequencies — 5%, 4%, 3%, 2%, 1% and 0.5% of the number of initial sequences — covering both degenerate and exact motif types, paired with control (negative) FASTA files. Run the motif finder under test on each positive/control pair (e.g. DiNAMO with 'dinamo -pf signal.fa -nf control.fa -l 6', using scanning mode by default and fixed-position mode via -p when the motif occurs at a specific offset). Compare predicted motifs against the implanted motifs after alignment and compute the nucleotide level correlation coefficient (nCC) to quantify recovery quality at the nucleotide level. Aggregate nCC by motif type and implantation frequency; a correct benchmark reproduces the paper's ranking, e.g. DiNAMO achieving the best nCC on degenerate motifs while MEME-CHIP is good only on exact motifs and Discrover the weakest.

## Related tools

- **DiNAMO** (Motif-discovery tool run on each synthetic positive/control pair; output motifs are scored against implanted motifs with nCC) — https://github.com/bonsai-team/DiNAMO
- **MEME** (Baseline EM-based motif finder whose predictions are benchmarked on the synthetic implanted-motif cases (also the output format used by DiNAMO))

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6 -o predicted_motifs.txt
```

## Evaluation signals

- Each tool's predicted motifs, when aligned to the implanted length-6 motifs, yield nCC values that are computable for every replicate
- Aggregated nCC shows the expected ranking: DiNAMO highest on degenerate implanted motifs, MEME-CHIP strong only on exact motifs, Discrover weakest
- Recovery quality degrades monotonically as implantation frequency decreases from 5% toward 0.5%
- DiNAMO outputs are in MEME format and respect the Fisher exact test P-value threshold (default 0.05, Holm–Bonferroni adjusted)
- Replicates cover all 6 implantation frequencies for each motif set so frequency-dependent trends are complete

## Limitations

- The benchmark uses fixed-length (L=6) IUPAC motifs, so tools must be run with comparable motif-length settings; results may not generalize to variable-length motifs
- DiNAMO's default Fisher P-value threshold of 0.05 and maximum degenerate-letter limit (-d) constrain detectable degenerate motifs and may affect cross-tool comparability
- Synthetic implantation does not capture real ChIP-seq noise structure; paper claims of robustness to noise are based on this synthetic design plus selected real datasets

## Evidence

- [results] Implanted IUPAC motifs of length 6 into positive datasets at 6 frequencies (5% to 0.5%): "we implanted motifs at 6 different frequencies: 5%, 4%, 3%, 2%, 1% and 0.5% of the number of initial sequences"
- [results] nCC evaluation metric: "We used the nucleotide level correlation coefficient (nCC) to evaluate the performance quality"
- [readme] DiNAMO invocation from README: "dinamo -pf signal.fa  -nf control.fa -l 6"
- [readme] Output format is MEME format: "The output file contains the over-represented motifs found in the positive dataset compared to the negative dataset, in the MEME format"
- [abstract] Benchmark ranking finding: "DiNAMO proves to compare favorably with other existing methods and is robust to noise."
