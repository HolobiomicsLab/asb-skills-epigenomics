---
name: macs2-log2fc-peak-scoring
description: Use when you have ChIP-seq peak calls and summit-centered sequences (e.g.
  ±200 bp around MACS2 summits) and need a per-sequence continuous score to drive
  MEPP positional enrichment analysis.
when_to_use_negative:
- Input sequences have no associated continuous score (e.g. plain peak lists or unscored
  FASTA) — MEPP needs a score-based continuum, and binary peak presence does not supply
  one
- The goal is de novo motif discovery — MEPP (and thus this scoring workflow) currently
  functions only with a fixed motif library
- The question is motif enrichment independent of position — a conventional MEA without
  positional profiling is the wrong framing
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: MEPP
  role: Consumes the log2FC-scored summit-centered FASTA via --fa and produces the
    positional enrichment profile anchored on the peak summit
  repo: https://github.com/npdeloss/mepp
- name: MACS2
  role: Produces the peak calls and log2 fold change scores (ChIP over control) used
    to score each extracted summit sequence
- name: pip
  role: Installs MEPP ('pip install mepp', the preferred method yielding the most
    recent stable release)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:c6aae96a3e90a2a32feee84894893412b8e53256f3743fc78f7225195f896b71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MACS2
  - MEPP
  - pip
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# macs2-log2fc-peak-scoring

## Summary

Score MEPP input sequences with the MACS2 log2 fold change of their parent ChIP-seq peak, converting peak calls (e.g. GATA1 in K562) into the continuous, score-based continuum of biological interest that MEPP requires for positional motif enrichment profiling.

## When to use

Use when you have ChIP-seq peak calls and summit-centered sequences (e.g. ±200 bp around MACS2 summits) and need a per-sequence continuous score to drive MEPP positional enrichment analysis.

## When NOT to use

- Input sequences have no associated continuous score (e.g. plain peak lists or unscored FASTA) — MEPP needs a score-based continuum, and binary peak presence does not supply one
- The goal is de novo motif discovery — MEPP (and thus this scoring workflow) currently functions only with a fixed motif library
- The question is motif enrichment independent of position — a conventional MEA without positional profiling is the wrong framing

## Inputs

- MACS2 ChIP-seq peak calls with summit positions and log2 fold change values (e.g. ENCODE K562 GATA1 ChIP-seq vs. control)
- Reference genome (for extracting ±200 bp sequences around each MACS2 peak summit)
- JASPAR-format motif matrices file (e.g. GATA1 position weight matrix)

## Outputs

- Summit-centered FASTA file with MACS2 log2 fold change scores encoded in headers as '>sequence_name sequence_score'
- Downstream MEPP positional enrichment profile plots (e.g. gata1_mepp_profile.png) showing motif enrichment relative to the peak summit anchor

## How to apply

Retrieve ChIP-seq and control peak calls (e.g. ENCODE K562 GATA1), extract sequences centered on each MACS2 peak summit (±200 bp), and score every extracted sequence with the log2 fold change (ChIP over control) of its parent peak. MEPP requires a score-based continuum reflecting two extremes of biological interest rather than binary peak presence, so log2FC values must remain continuous and not be thresholded. Encode the score in the FASTA header in MEPP's required format '>sequence_name sequence_score', so the CLI accepts the file via --fa. This score-based approach avoids arbitrary threshold selection while MEPP controls for sequence bias, and the resulting summit-anchored, log2FC-scored FASTA is then analyzed with the GATA1 position weight matrix (JASPAR format) to compute the positional enrichment profile around the peak summit.

## Related tools

- **MEPP** (Consumes the log2FC-scored summit-centered FASTA via --fa and produces the positional enrichment profile anchored on the peak summit) — https://github.com/npdeloss/mepp
- **MACS2** (Produces the peak calls and log2 fold change scores (ChIP over control) used to score each extracted summit sequence)
- **pip** (Installs MEPP ('pip install mepp', the preferred method yielding the most recent stable release))

## Examples

```
mepp --fa gata1_k562_summits.scored.fa --motifs data/homer.motifs.txt --out gata1_mepp_out --center 200
```

## Evaluation signals

- FASTA headers follow MEPP's required format '>sequence_name sequence_score' and scores are continuous MACS2 log2FC values, not binary peak labels
- Sequences are centered on MACS2 peak summits (±200 bp), matching the article's ChIP-seq workflow
- The MEPP run completes and produces the profile image (e.g. gata1_mepp_profile.png exists)
- The resulting profile shows centrally positioned GATA1 motif enrichment around the summit anchor, correlated with higher ChIP-over-control coverage, qualitatively matching published Figure 2
- Profile shape is less sharp than single-nucleotide-resolution TSS-centered analyses, consistent with the less precise positional landmark of peak summits

## Limitations

- ChIP-seq peak summits are less precise positional landmarks than transcription start sites, so profiles are blurrier than single-nucleotide-resolution TSS-centered analyses
- MEPP currently functions only with a fixed motif library; de novo motif recognition/assembly is left to future work
- MEPP is usable by scoring genomic regions across a continuum of scores reflecting two extremes of biological interest — the skill fails if the parent-peak score is absent, degenerate, or thresholded into classes

## Evidence

- [other] Score every extracted sequence with the MACS2 log2 fold change of its parent peak, providing MEPP the score-based continuum of biological interest it requires.: [evidence span withheld — anchor status: not_found]
- [discussion] MEPP is usable by scoring genomic regions across a continuum: "Similar to our previous work with MEIRLOP, MEPP is usable by scoring genomic regions across a continuum of scores reflecting two extremes of biological interest."
- [intro] The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias.: "The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias."
- [readme] Path to a scored fasta file, where sequence headers are of the form: ">sequence_name sequence_score".: "Path to a scored fasta file, where sequence
                                      headers are of the form: ">sequence_name
                                      sequence_score"."
