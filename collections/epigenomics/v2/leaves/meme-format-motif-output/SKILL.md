---
name: meme-format-motif-output
description: Use when DiNAMO has finished filtering over-represented IUPAC motifs
  and you must emit them in MEME format for downstream tools, validation, or JASPAR
  matrix comparison.
when_to_use_negative:
- The motifs must be consumed by a tool that cannot parse MEME-format records — use
  a raw IUPAC/degenerate string export instead
- No statistical filtering has been performed yet — motifs without Holm–Bonferroni-adjusted
  Fisher's exact test P-values should not be written as final MEME motif records
- The run is in DiNAMO's fixed-position mode and you need position-specific error
  motifs rather than scanning-mode over-representation output
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0160
tools:
- name: DiNAMO
  role: Discovers over-represented IUPAC motifs and writes the filtered results to
    the output file in MEME format
  repo: https://github.com/bonsai-team/DiNAMO
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:f7b2f17c93378559add524f46c05a6573a8c7042c1d76198578e3ed85e270567
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# meme-format-motif-output

## Summary

Skill for writing DiNAMO's significantly over-represented IUPAC motifs (surviving Holm–Bonferroni-adjusted Fisher's exact test at threshold 0.05) into an output file in MEME motif format, so discovered motifs can be directly compared against frequency matrices such as JASPAR.

## When to use

Use when DiNAMO has finished filtering over-represented IUPAC motifs and you must emit them in MEME format for downstream tools, validation, or JASPAR matrix comparison.

## When NOT to use

- The motifs must be consumed by a tool that cannot parse MEME-format records — use a raw IUPAC/degenerate string export instead
- No statistical filtering has been performed yet — motifs without Holm–Bonferroni-adjusted Fisher's exact test P-values should not be written as final MEME motif records
- The run is in DiNAMO's fixed-position mode and you need position-specific error motifs rather than scanning-mode over-representation output

## Inputs

- Set of over-represented IUPAC motifs with Holm–Bonferroni-adjusted Fisher's exact test P-values (< 0.05 by default) from DiNAMO
- Positive dataset (signal.fa) and negative dataset (control.fa) in multi-fasta format used by the DiNAMO run

## Outputs

- Motif file in MEME format (e.g. dinamo_motifs.txt or overrepresented_motifs.meme) containing only significantly over-represented motifs

## How to apply

After DiNAMO counts L-mers, builds the IUPAC motif lattice, simplifies it by mutual information, and filters motifs with the Holm–Bonferroni-adjusted Fisher's exact test (default threshold 0.05, configurable via '-t'), write only the surviving over-represented motifs to the output file in MEME format. The default output name from the reproduced workflow is dinamo_motifs.txt, and DiNAMO's '-o' parameter can override the output file. Validate that the file parses as well-formed MEME-format records and contains only motifs with adjusted P-value below the significance threshold. The MEME output is deliberately chosen so the top motif can be compared against JASPAR frequency matrices, as done in the paper's ChIP-seq case studies (GATA1, SOX2, OCT4, STAT3, KLF1).

## Related tools

- **DiNAMO** (Discovers over-represented IUPAC motifs and writes the filtered results to the output file in MEME format) — https://github.com/bonsai-team/DiNAMO

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6 -t 0.05 -o dinamo_motifs.txt
```

## Evaluation signals

- The output file parses as valid MEME-format motif records (per http://meme-suite.org/doc/meme-format.html)
- Every retained motif has a Holm–Bonferroni-adjusted Fisher's exact test P-value strictly below the 0.05 threshold (or the '-t' value used)
- The number of motifs in the output matches the number of non-dominated, statistically significant lattice motifs from the DiNAMO run
- The top motif, when converted to a frequency matrix, aligns with the expected factor's JASPAR matrix (e.g. GATA1, SOX2, OCT4, STAT3, KLF1)
- On the SSE dataset, the output reproduces the NNRGGT top motif (adjusted P-value < 10−324) confirming the error-associated GGT motif

## Limitations

- Motifs are degenerate IUPAC strings of fixed length L; MEME records derived from them do not carry position-specific base frequencies beyond the consensus itself
- The significance threshold must be controlled via '-t'; omitting statistical filtering means non-significant motifs could be emitted
- Clustering of overlapping motifs (minimum overlap of half the motif size) is optional, so unclustered redundant motifs may appear in the output

## Evidence

- [readme] README output specification: "The output file contains the over-represented motifs found in the positive dataset compared to the negative dataset, in the MEME format"
- [readme] README input description: "DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa)."
- [methods] Statistical filtering step: [evidence span withheld — anchor status: not_found]
- [results] JASPAR comparison validation: "comparing them against the frequency matrices of the JASPAR database [34]"
