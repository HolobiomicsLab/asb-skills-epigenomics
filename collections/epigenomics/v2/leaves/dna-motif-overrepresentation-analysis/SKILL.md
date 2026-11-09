---
name: dna-motif-overrepresentation-analysis
description: Use when searching a large signal (e.g. ChIP-seq peak) FASTA against
  a control FASTA for over-represented degenerate IUPAC motifs of a fixed length,
  without any prior restriction on the motif.
when_to_use_negative:
- When motifs must be exact strings rather than degenerate IUPAC models — tools like
  RSAT handle exact strings instead
- When a position-specific frequency-matrix (EM-based) search is required, e.g. the
  MEME algorithm, rather than exact IUPAC motif enumeration
- When only a single sequence set is available with no control/negative dataset, since
  scoring relies on a 2x2 contingency table over positive vs negative datasets
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: DiNAMO
  role: Implements the exhaustive IUPAC motif lattice construction, MI scoring, and
    Holm-Bonferroni-corrected Fisher's exact test filtering; outputs MEME-format motifs.
  repo: https://github.com/bonsai-team/DiNAMO
- name: RSAT
  role: Referenced as a prior approach that searches exact strings rather than degenerate
    IUPAC motifs — useful to contrast scope, not used directly by this skill.
- name: MEME
  role: Referenced as an EM-based motif discovery method; its output format is used
    by DiNAMO for reporting over-represented motifs.
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:b42855bf7e15eb5a080e1b79530359039856de001f331c95514801640a4e4035
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  - MEME
  - RSAT
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# dna-motif-overrepresentation-analysis

## Summary

An exact, exhaustive method for discovering over-represented degenerate IUPAC DNA motifs in a positive sequence dataset compared to a negative control, implemented in the DiNAMO software. It uses mutual information (MI) as an occurrence-count-independent objective and Fisher's exact test with Holm-Bonferroni correction for significance.

## When to use

Use when searching a large signal (e.g. ChIP-seq peak) FASTA against a control FASTA for over-represented degenerate IUPAC motifs of a fixed length, without any prior restriction on the motif.

## When NOT to use

- When motifs must be exact strings rather than degenerate IUPAC models — tools like RSAT handle exact strings instead
- When a position-specific frequency-matrix (EM-based) search is required, e.g. the MEME algorithm, rather than exact IUPAC motif enumeration
- When only a single sequence set is available with no control/negative dataset, since scoring relies on a 2x2 contingency table over positive vs negative datasets

## Inputs

- Positive (signal) dataset in FASTA format, e.g. signal.fa (e.g. ChIP-seq peaks or error-containing reads)
- Negative (control) dataset in FASTA format, e.g. control.fa
- Motif length L (parameter -l)
- Optional: maximum number of degenerate letters i (parameter -d)
- Optional: fixed-position offset i relative to the end of each sequence (parameter -p)
- Optional: Fisher's exact test P-value threshold f (parameter -t, default 0.05)

## Outputs

- Over-represented IUPAC motifs in MEME format (e.g. overrepresented_motifs.meme)
- Per-motif MI scores and adjusted Fisher's exact test P-values (Holm-Bonferroni) for retained motifs
- Optionally, clusters of overlapping over-represented motifs and secondary motifs

## How to apply

Run DiNAMO with a positive dataset FASTA (-pf), a negative/control dataset FASTA (-nf), and a motif length (-l); optionally cap the number of degenerate letters with -d and restrict to a fixed position with -p. The algorithm counts all L-mers in both files into hashtables, builds the lattice of IUPAC motifs whose instances are all present in the positive dataset, assigns each lattice vertex a 2x2 contingency table (a, b, c, d), and computes mutual information MI(O;C) between motif presence/absence and dataset condition — MI captures the dependency between a motif and a dataset independently from its number of occurrences, unlike Fisher's exact test P-value. Motifs are ranked by decreasing MI and dominated descendants/ancestors are pruned; selected motifs are then filtered by Fisher's exact test P-values adjusted with the Holm-Bonferroni method (default threshold 0.05, configurable via -t). Surviving significantly over-represented motifs are written in MEME format; optional post-processing can mask instances to detect secondary motifs and cluster overlapping motifs by sequence similarity.

## Related tools

- **DiNAMO** (Implements the exhaustive IUPAC motif lattice construction, MI scoring, and Holm-Bonferroni-corrected Fisher's exact test filtering; outputs MEME-format motifs.) — https://github.com/bonsai-team/DiNAMO
- **RSAT** (Referenced as a prior approach that searches exact strings rather than degenerate IUPAC motifs — useful to contrast scope, not used directly by this skill.)
- **MEME** (Referenced as an EM-based motif discovery method; its output format is used by DiNAMO for reporting over-represented motifs.)

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6
```

## Evaluation signals

- Every retained motif has a Holm-Bonferroni-adjusted Fisher's exact test P-value strictly below the significance threshold (default 0.05, set via -t).
- The retained MI-selected motif set satisfies the dominance invariant: each kept motif has MI greater than each of its descendants and is not dominated by any ancestor.
- MI values recomputed from the stored 2x2 contingency tables (a, b, c, d) match the reported MI scores within numerical tolerance.
- The output file parses as valid MEME-format motif records (per http://meme-suite.org/doc/meme-format.html).
- The final lattice contains exactly all IUPAC motifs whose instances are all present in the positive dataset — verifying no candidate motifs were dropped during lattice construction.

## Limitations

- Requires both a positive and a negative dataset to form the 2x2 contingency tables; single-set analyses are outside this skill's scope.
- Position modes differ: default scanning mode parses all sliding windows; fixed-position mode (-p) only considers motifs at one offset from the sequence end — choosing the wrong mode changes results.
- MI is used to compare motifs independently of occurrence count, so very rare motifs are not automatically prioritized; downstream Fisher's exact test with Holm-Bonferroni is what controls significance.
- Runs on a desktop machine with less than 8 GB of RAM per the article's Table 3 context, but execution time and memory scale with dataset size and motif length.

## Evidence

- [methods] MI scoring is occurrence-count independent: "the MI (mutual information) captures the dependency between a motif and a dataset independently from its number of occurrences."
- [methods] Final significance step: Fisher's exact test + Holm-Bonferroni: [evidence span withheld — anchor status: not_found]
- [methods] Lattice completeness invariant: "At the end of the process, the lattice contains exactly the set of all IUPAC motifs for which all instances are present in"
- [readme] README: basic invocation and parameters: "dinamo -pf signal.fa  -nf control.fa -l 6"
- [readme] README: output format and threshold parameter: "change Fisher's exact test p-value threshold *f* (default: 0.05)"
