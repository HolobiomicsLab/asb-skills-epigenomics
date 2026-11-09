---
name: cross-tool-motif-prediction-alignment
description: Use when comparing motif discovery tools on datasets with known implanted
  motifs (length-6 IUPAC/degenerate or exact strings) at controlled frequencies, and
  a nucleotide-level recovery metric such as nCC is needed.
when_to_use_negative:
- Data are real ChIP-seq peaks with no known implanted motifs, so no ground-truth
  motif exists for nCC alignment scoring
- The question is biological motif discovery itself rather than tool benchmarking
  — use DiNAMO directly on signal/control FASTA pairs
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: DiNAMO
  role: Exhaustive IUPAC motif discovery tool under benchmark; run on each positive/control
    FASTA pair in scanning or fixed-position mode
  repo: https://github.com/bonsai-team/DiNAMO
- name: MEME
  role: Baseline EM-based motif discovery tool compared against DiNAMO (via MEME-CHIP
    for ChIP-seq style benchmarks)
- name: HOMER
  role: Baseline motif discovery tool for ChIP-seq analysis included in the nCC comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:3b1ff70d2a53c7c1b36367cbd326b5d8a7f8429439fad5db0193c6c8cefe6487
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  - HOMER
  - MEME
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cross-tool-motif-prediction-alignment

## Summary

Benchmark DNA motif-discovery tools (DiNAMO vs DREME, MEME, MEME-CHIP, HOMER, Discrover) on synthetic implanted-motif FASTA datasets by aligning each tool's predicted IUPAC motifs against the implanted motifs and scoring recovery with the nucleotide level correlation coefficient (nCC).

## When to use

Use when comparing motif discovery tools on datasets with known implanted motifs (length-6 IUPAC/degenerate or exact strings) at controlled frequencies, and a nucleotide-level recovery metric such as nCC is needed.

## When NOT to use

- Data are real ChIP-seq peaks with no known implanted motifs, so no ground-truth motif exists for nCC alignment scoring
- The question is biological motif discovery itself rather than tool benchmarking — use DiNAMO directly on signal/control FASTA pairs

## Inputs

- Positive (signal) FASTA files with length-6 IUPAC motifs implanted at 6 frequencies (5%–0.5%)
- Control (negative) FASTA files matched to each positive dataset
- Motif length parameter L=6 and list of motif types (degenerate and exact)

## Outputs

- Per-tool predicted motifs in MEME format
- Per-case nCC scores from aligning predicted motifs to implanted motifs
- Aggregated nCC tables by motif type and implantation frequency with tool ranking

## How to apply

Generate synthetic positive/control FASTA pairs with length-6 IUPAC motifs implanted at 5%, 4%, 3%, 2%, 1% and 0.5% of the number of initial sequences, covering degenerate and exact motif types. Run DiNAMO on each pair in scanning mode (default) or fixed-position mode as appropriate, with default Fisher's exact test P-value threshold 0.05; run MEME-CHIP, HOMER and Discrover on the same cases with motif-length settings comparable to L=6. DiNAMO outputs motifs in MEME format, which serves as the common representation for cross-tool comparison. For each case, align every tool's predicted motifs against the implanted motifs and compute the nucleotide level correlation coefficient (nCC) as the recovery metric. Aggregate nCC by motif type (degenerate vs exact) and implantation frequency, and rank the tools; the expected ranking is DiNAMO highest on degenerate motifs, MEME-CHIP good only on exact motifs, Discrover weakest.

## Related tools

- **DiNAMO** (Exhaustive IUPAC motif discovery tool under benchmark; run on each positive/control FASTA pair in scanning or fixed-position mode) — https://github.com/bonsai-team/DiNAMO
- **MEME** (Baseline EM-based motif discovery tool compared against DiNAMO (via MEME-CHIP for ChIP-seq style benchmarks))
- **HOMER** (Baseline motif discovery tool for ChIP-seq analysis included in the nCC comparison)

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6
```

## Evaluation signals

- The reproduced table shows DiNAMO achieving the highest nCC on degenerate implanted motifs among compared tools
- MEME-CHIP performs well only on exact (non-degenerate) implanted motifs
- Discrover yields the weakest nCC value in the comparison
- nCC scores are aggregated separately for each implantation frequency (5% down to 0.5%), showing degradation with lower frequency
- DiNAMO predicted motifs are returned in valid MEME format suitable for alignment against implanted motifs

## Limitations

- Performance degrades at the lowest implantation frequencies (0.5%–1%), where motif signal is weakest
- DiNAMO's lattice enumeration and runtime/memory behavior are bounded resources (runs on a desktop machine with < 8 GB RAM per the paper's Table 3)
- nCC alignment-based scoring depends on correct alignment of predicted to implanted motifs; cross-tool runs must use comparable motif-length settings (L=6)

## Evidence

- [results] Implanted IUPAC motifs of length 6 into positive datasets at 6 frequencies (5% to 0.5%): "we implanted motifs at 6 different frequencies: 5%, 4%, 3%, 2%, 1% and 0.5% of the number of initial sequences"
- [results] nCC used as evaluation metric: "We used the nucleotide level correlation coefficient (nCC) to evaluate the performance quality"
- [readme] DiNAMO CLI usage on signal/control FASTA pairs: "dinamo -pf signal.fa  -nf control.fa -l 6"
- [readme] MEME-format output enables cross-tool motif comparison: "The output file contains the over-represented motifs found in the positive dataset compared to the negative dataset, in the MEME format"
