---
name: low-information-sequence-filtering
description: Use when running the Delila-PY pipeline and, after cycle 1's malign/malin
  alignment, some input sequences from the ChIP-derived sites file show information
  content of 0 bits or less, contaminating the motif with non-matching sequences.
when_to_use_negative:
- The input is already a curated set of previously identified binding sites with uniformly
  positive information content, so the ri-based ≤0-bit filter would remove nothing
- The goal is de novo motif discovery without positional boundaries — generic tools
  like MEME (expectation maximization) or BioProspector (Gibbs sampling) fit better
  and require no GenBank plus sites-file setup
- Sequences are unaligned and cycle 1 (malign/malin) has not been run — the ri filter
  operates on per-sequence information content from the cycle-1-aligned book
edam_topics:
- http://edamontology.org/topic_0779
tools:
- name: Delila-PY
  role: 'Python3/Docker pipeline orchestrating the cycle-2 filter step: information-content
    computation, ri-based removal, and delila book regeneration'
  repo: https://github.com/GLBRC/delila
- name: ri
  role: Program that removes poorly matching sequences with ≤0 information content
    from the aligned book
- name: alist
  role: Computes the per-sequence information content values that the ri filter applies
    its ≤0-bit threshold to
- name: rseq
  role: Generates per-sequence information along with alist and encode for the filtering
    decision in cycle 2
- name: delila
  role: Regenerates the updated book file from the filtered, information-positive
    sequences before the final cycle-3 motif generation
  repo: https://github.com/GLBRC/delila
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/mra.00160-21
    title: Delila-PY, a Pipeline for Utilizing the Delila Suite of Software to Identify
      Potential DNA Binding Motifs
  source_content_hash: sha256:7a5d63ca5f44fc76066cbcb3212dabbb25f4b11a02e5b7935e54687e92c273cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/delila
  tools:
  - Delila-PY
  - Docker
  - alist
  - catal
  - dbbk
  - delila
  - makelogo
  - malign
  - ri
  - rseq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0779
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# low-information-sequence-filtering

## Summary

Delila-PY's cycle-2 filtering step, which computes per-sequence information content (alist, encode, cmp, rseq) and removes poorly matching sequences with ≤0 bits using the ri program before regenerating the book with delila, so that the final motif (cycle 3) is built only from sequences that carry positive information.

## When to use

Use when running the Delila-PY pipeline and, after cycle 1's malign/malin alignment, some input sequences from the ChIP-derived sites file show information content of 0 bits or less, contaminating the motif with non-matching sequences.

## When NOT to use

- The input is already a curated set of previously identified binding sites with uniformly positive information content, so the ri-based ≤0-bit filter would remove nothing
- The goal is de novo motif discovery without positional boundaries — generic tools like MEME (expectation maximization) or BioProspector (Gibbs sampling) fit better and require no GenBank plus sites-file setup
- Sequences are unaligned and cycle 1 (malign/malin) has not been run — the ri filter operates on per-sequence information content from the cycle-1-aligned book

## Inputs

- Updated Delila book of sequences from cycle 1 (produced by delila after malign/malin)
- GenBank file for the target genome (e.g., NC_007493.2 for R. sphaeroides)
- Tab-delimited four-column sites file (chromosome name, unique site name, strand as 'forward'/'reverse', base pair position) with left/right boundaries relative to each position (e.g., -l -10 -r -10)

## Outputs

- Regenerated Delila book file containing only sequences with information content > 0 bits
- Per-sequence information content values (from alist, encode, cmp, rseq) used by the ri filter
- Filtered sequence set feeding the final cycle-3 logo, PWM text file, and PostScript/PDF sequence logo

## How to apply

After cycle 1 (malign and malin find the alignment with the highest information content and delila regenerates the book), run alist, encode, cmp, and rseq on the updated book to compute the per-sequence information content. Apply the ri program to remove sequences with ≤0 information content — this is the explicit threshold Delila-PY uses to define poorly matching sequences. Feed the retained sequences back into delila to generate a new updated book file. Cycle 3 (alist, encode, cmp, rseq, dalvec, makelogo) then builds the DNA motif and PWM from the filtered set only. This filtering rationale is why the Delila-PY FnrL ChIP-chip logo resembles the logo from previously identified FnrL binding sites.

## Related tools

- **Delila-PY** (Python3/Docker pipeline orchestrating the cycle-2 filter step: information-content computation, ri-based removal, and delila book regeneration) — https://github.com/GLBRC/delila
- **ri** (Program that removes poorly matching sequences with ≤0 information content from the aligned book)
- **alist** (Computes the per-sequence information content values that the ri filter applies its ≤0-bit threshold to)
- **rseq** (Generates per-sequence information along with alist and encode for the filtering decision in cycle 2)
- **delila** (Regenerates the updated book file from the filtered, information-positive sequences before the final cycle-3 motif generation) — https://github.com/GLBRC/delila

## Examples

```
delila_pipeline.py -g Rsphaeroides_GenBank.gbff -l -10 -r -10 -s Location_File_Example.txt -t "Logo Title"
```

## Evaluation signals

- Every sequence retained in the regenerated book has per-sequence information content strictly greater than 0 bits; sequences at ≤0 bits were removed by ri
- The cycle-3 outputs (makelogo logo in PostScript and PDF, PWM text file) are parseable and per-position information content falls within 0–2 bits
- The final FnrL sequence logo (Fig. 1B) visually resembles the logo from previously identified FnrL binding sites (Fig. 1C)
- The pipeline ran reproducibly via the glbrc/delila Docker image on the example files before applying the filter to real ChIP-chip-derived genomic locations

## Limitations

- The filter is a hard ≤0-information-content cutoff via ri; it does not offer graded or user-tunable thresholds beyond what Delila-PY's standard (often default) values provide
- Filtering depends on cycle 1's malign/malin alignment having already been computed; running ri on an unaligned book is not part of the pipeline
- Delila-PY has been tested on MacOS and Linux (Ubuntu and CentOS); other operating systems may require additional work, and Docker (recommended) with volume mounting is needed for reproducible runs

## Evidence

- [abstract] Delila-PY removes poorly matching sequences (≤0 information content, ri program). Then, a new file is generated and fed back to the delila program to generate an updated book file: "Delila-PY removes poorly matching sequences (≤0 information content, ri program). Then, a new file is generated and fed back to the delila program to generate an updated book file"
- [abstract] The final cycle generates a DNA motif (alist, encode, cmp, rseq, dalvec, and makelogo): "The final cycle generates a DNA motif (alist, encode, cmp, rseq, dalvec, and makelogo)"
- [intro] A sites file which is tab delimited with four columns (no column names): "A sites file which is tab delimited with four columns (no column names)"
- [readme] Delila-PY uses standard (often default) values and allows the user to quickly and reproducibly process a file consisting of genomic locations to identify a potential logo.: "Delila-PY uses standard (often default) values and allows the user to quickly and reproducibly process a file consisting of genomic locations to identify a potential logo."
