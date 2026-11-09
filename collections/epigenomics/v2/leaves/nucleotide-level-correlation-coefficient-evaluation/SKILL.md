---
name: nucleotide-level-correlation-coefficient-evaluation
description: Use when comparing motif discovery tools (e.g. DiNAMO vs MEME-CHIP, HOMER,
  Discrover) on data where ground-truth implanted motifs exist and per-nucleotide
  recovery quality must be quantified.
when_to_use_negative:
- Real datasets with no known implanted ground-truth motifs — nCC against implanted
  motifs cannot be computed
- Evaluating tools on motif properties other than nucleotide-level recovery (e.g.
  runtime, memory) where the article reports separate metrics (Table 3)
- Motif lengths other than the benchmarked L=6 without re-validating the pipeline
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0091
tools:
- name: DiNAMO
  role: Primary motif discovery tool benchmarked; run on each positive/control FASTA
    pair with -l 6 to produce predicted motifs in MEME format
  repo: https://github.com/bonsai-team/DiNAMO
- name: MEME
  role: Comparison tool (MEME-CHIP) run on the same synthetic cases; performs well
    only on exact implanted motifs
  repo: http://meme-suite.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:ca30c58cd46158b55a9970b7554d5a41add17c387a5daf169652cc0305710af8
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# nucleotide-level-correlation-coefficient-evaluation

## Summary

Benchmark DNA motif discovery tools by implanting known IUPAC motifs into synthetic positive/control FASTA pairs and scoring each tool's predicted motifs with the nucleotide level correlation coefficient (nCC), the metric used to show DiNAMO outperforming MEME-CHIP, HOMER and Discrover on degenerate motifs.

## When to use

Use when comparing motif discovery tools (e.g. DiNAMO vs MEME-CHIP, HOMER, Discrover) on data where ground-truth implanted motifs exist and per-nucleotide recovery quality must be quantified.

## When NOT to use

- Real datasets with no known implanted ground-truth motifs — nCC against implanted motifs cannot be computed
- Evaluating tools on motif properties other than nucleotide-level recovery (e.g. runtime, memory) where the article reports separate metrics (Table 3)
- Motif lengths other than the benchmarked L=6 without re-validating the pipeline

## Inputs

- Synthetic positive FASTA files with length-6 IUPAC motifs implanted at 5%, 4%, 3%, 2%, 1%, 0.5% frequencies
- Matching negative/control FASTA files
- Implanted motif definitions (degenerate and exact IUPAC strings) as ground truth
- Predicted motif sets from each tool in MEME format

## Outputs

- Per-case nCC scores aligning predicted motifs to implanted motifs
- Aggregated nCC tables by motif type and implantation frequency
- Tool ranking (DiNAMO > MEME-CHIP/HOMER > Discrover on degenerate motifs)

## How to apply

Generate synthetic evaluation cases by implanting length-6 IUPAC motifs (degenerate and exact types) into positive FASTA files at frequencies of 5%, 4%, 3%, 2%, 1% and 0.5% of the number of initial sequences, each paired with a control FASTA. Run DiNAMO on each positive/control pair (`dinamo -pf signal.fa -nf control.fa -l 6`, default Fisher P-value threshold 0.05, optional -d for maximum degenerate letters) in scanning or fixed-position mode, and run MEME-CHIP and Discrover with comparable motif-length settings. Align each tool's predicted motifs (output in MEME format) against the implanted motifs and compute the nucleotide level correlation coefficient (nCC) per case. Aggregate nCC by motif type and implantation frequency; correct application is confirmed when the reproduced ranking shows DiNAMO with the highest nCC on degenerate motifs, MEME-CHIP good only on exact motifs, and Discrover weakest.

## Related tools

- **DiNAMO** (Primary motif discovery tool benchmarked; run on each positive/control FASTA pair with -l 6 to produce predicted motifs in MEME format) — https://github.com/bonsai-team/DiNAMO
- **MEME** (Comparison tool (MEME-CHIP) run on the same synthetic cases; performs well only on exact implanted motifs) — http://meme-suite.org/

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6
```

## Evaluation signals

- DiNAMO achieves the highest nCC on degenerate implanted motifs among DiNAMO, MEME-CHIP, HOMER and Discrover
- MEME-CHIP scores well only on exact (non-degenerate) implanted motifs
- Discrover yields the weakest nCC value in the reproduced comparison
- nCC is computed for all 6 implantation frequencies (5% down to 0.5%), including the hardest low-frequency cases
- Every predicted motif is aligned against its implanted counterpart before scoring, not compared by occurrence count alone

## Limitations

- The benchmark covers only length-6 IUPAC motifs; performance at other motif lengths is not established by this evaluation
- Discrover and MEME-CHIP were run with default motif-length settings made comparable to L=6, so exact parameter parity across tools is not guaranteed
- Synthetic implantation frequencies as low as 0.5% represent weak signals; tool rankings at such low frequencies may be noisy

## Evidence

- [results] we implanted motifs at 6 different frequencies: 5%, 4%, 3%, 2%, 1% and 0.5% of the number of initial sequences: "we implanted motifs at 6 different frequencies: 5%, 4%, 3%, 2%, 1% and 0.5% of the number of initial sequences"
- [results] nCC used for evaluation: "We used the nucleotide level correlation coefficient (nCC) to evaluate the performance quality"
- [readme] README usage: "dinamo -pf signal.fa  -nf control.fa -l 6"
- [methods] MI rationale: "the MI (mutual information) captures the dependency between a motif and a dataset independently from its number of occurrences."
