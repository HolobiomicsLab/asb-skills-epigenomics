---
name: gata1-motif-enrichment-visualization
description: Use when you need to test whether a transcription factor motif (e.g.
  GATA1) is enriched at specific positions relative to an anchor point such as ChIP-seq
  peak summits, using score-continuum-ranked sequences.
when_to_use_negative:
- You only need a single global motif enrichment score with no positional information
  — most MEA methods determine motif enrichment independent of motif position, but
  a positional profile is unnecessary if position is irrelevant to the question.
- You need de novo motif discovery — MEPP currently functions only with a fixed motif
  library (e.g. the bundled HOMER motifs in data/homer.motifs.txt) and does not recognize
  or assemble motifs de novo.
- The analysis requires single-nucleotide-resolution positioning around a precise
  anchor such as a TSS — the ChIP-seq summit-centered profile is inherently less sharp
  than TSS-centered analyses.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0203
tools:
- name: MEPP
  role: Computes the position-dependent GATA1 motif enrichment profile across summit-centered,
    log2FC-scored K562 ChIP-seq sequences; installed via pip and run as CLI or imported
    ('import mepp')
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: 'Preferred installation method for MEPP: ''pip install mepp'' yields the most
    recent stable release'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:a43aad0a76d2dffc1b0b7f93d6eb6826e227ef6b5ee0f7c57d10578ecd860443
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  - pip
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gata1-motif-enrichment-visualization

## Summary

Visualize position-dependent enrichment of the GATA1 binding motif by running MEPP (Motif Enrichment Positional Profiling) on K562 GATA1 ChIP-seq MACS2 peak summit sequences (±200 bp) scored with MACS2 log2 fold change, producing a positional enrichment profile anchored at the peak summit.

## When to use

Use when you need to test whether a transcription factor motif (e.g. GATA1) is enriched at specific positions relative to an anchor point such as ChIP-seq peak summits, using score-continuum-ranked sequences.

## When NOT to use

- You only need a single global motif enrichment score with no positional information — most MEA methods determine motif enrichment independent of motif position, but a positional profile is unnecessary if position is irrelevant to the question.
- You need de novo motif discovery — MEPP currently functions only with a fixed motif library (e.g. the bundled HOMER motifs in data/homer.motifs.txt) and does not recognize or assemble motifs de novo.
- The analysis requires single-nucleotide-resolution positioning around a precise anchor such as a TSS — the ChIP-seq summit-centered profile is inherently less sharp than TSS-centered analyses.

## Inputs

- ENCODE K562 GATA1 ChIP-seq peak calls and matched control peak calls
- MACS2 peak summit coordinates from GATA1 ChIP-seq
- ±200 bp summit-centered genomic sequences as a scored FASTA (headers of the form '>sequence_name sequence_score', score = MACS2 log2 fold change)
- GATA1 position weight matrix in JASPAR format

## Outputs

- Position-dependent MEPP motif enrichment profile plot centered on the peak summit (e.g. gata1_mepp_profile.png; PNG/SVG output supported)
- Positional correlation statistics (p-values with FDR adjustment) and clustering table per motif

## How to apply

Retrieve ENCODE K562 GATA1 ChIP-seq and control peak calls, and extract ±200 bp sequences around each MACS2 peak summit so the summit serves as the positional anchor. Score every extracted sequence with the MACS2 log2 fold change of its parent peak, because MEPP requires a score-based continuum of biological interest rather than binary peak/non-peak labels. Install MEPP via pip ('pip install mepp', the preferred method yielding the most recent stable release), import mepp, and run it on the summit-centered, log2FC-scored FASTA with the GATA1 position weight matrix (JASPAR-format motif file) to compute the position-dependent enrichment profile across the anchor point. Plot the resulting profile and compare it to the published Figure 2; expect central GATA1 motif enrichment around the summit that correlates with higher ChIP-over-control coverage, with a profile shape less sharp than single-nucleotide-resolution TSS-centered analyses.

## Related tools

- **MEPP** (Computes the position-dependent GATA1 motif enrichment profile across summit-centered, log2FC-scored K562 ChIP-seq sequences; installed via pip and run as CLI or imported ('import mepp')) — https://github.com/npdeloss/mepp
- **pip** (Preferred installation method for MEPP: 'pip install mepp' yields the most recent stable release)

## Examples

```
mepp --fa k562_gata1_summits.scored.fa --motifs gata1_jaspar.txt --out mepp_gata1_out --formats png,svg
```

## Evaluation signals

- The output file gata1_mepp_profile.png exists and displays centrally positioned GATA1 motif enrichment around the summit anchor.
- The central enrichment qualitatively matches the published Figure 2 profile, with motif presence correlating with higher GATA1 ChIP-over-control coverage (MACS2 log2 fold change).
- The profile shape is broader/less sharp than a single-nucleotide-resolution TSS-centered MEPP analysis, consistent with the less precise positional landmark of peak summits.
- MEPP ran on the full score continuum of sequences (no arbitrary peak/non-peak threshold was applied), and input sequences are summit-centered at ±200 bp.

## Limitations

- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work.
- Positional resolution is limited by the anchor's precision: the summit-centered ChIP-seq profile is less sharp than TSS-centered analyses.
- Permutation testing and GPU processing can be memory-intensive; --perms (default 1000) and --gjobs must be set carefully to avoid jobs crowding each other out of GPU memory, causing profile generation to fail (use --nogpu as fallback).

## Evidence

- [abstract] outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs): "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs)"
- [methods] To install Motif Enrichment Positional Profiling, run this command in your terminal: $ pip install mepp: "To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp"
- [discussion] MEPP is usable by scoring genomic regions across a continuum of scores reflecting two extremes of biological interest.: "MEPP is usable by scoring genomic regions across a continuum of scores reflecting two extremes of biological interest."
- [readme] Path to a scored fasta file, where sequence headers are of the form: ">sequence_name sequence_score".: "Path to a scored fasta file, where sequence
                                      headers are of the form: ">sequence_name
                                      sequence_score"."
- [other] the GATA1 motif is enriched at central positions relative to the peak summit, with a profile shape less sharp than single-nucleotide-resolution TSS-centered analyses: [evidence span withheld — anchor status: not_found]
