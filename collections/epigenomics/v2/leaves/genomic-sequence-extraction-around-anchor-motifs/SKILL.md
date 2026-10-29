---
name: genomic-sequence-extraction-around-anchor-motifs
description: Use when a positional motif enrichment analysis (e.g. MEPP) requires
  centered sequences around anchors such as deduplicated, RepeatMasker-filtered motif
  sites, TSSs, or cleavage sites, with a score attached per sequence.
when_to_use_negative:
- Sequences are already centered and scored for MEPP — re-extraction would change
  positional offsets relative to the anchor
- The goal is de novo motif discovery rather than profiling known motifs relative
  to anchors — MEPP currently functions only with a fixed motif library
- Motif enrichment is needed independent of position within the sequence — standard
  MEA methods (e.g. HOMER, MEIRLOP under a ZOOPs model) suffice and do not require
  centered anchors
edam_operation: http://edamontology.org/operation_0384
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0749
tools:
- name: MEPP
  role: Downstream consumer of the extracted anchor-centered scored FASTA; requires
    sequences with headers '>sequence_name sequence_score' and rejects sequences above
    the --dgt degenerate-base percentage
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installation channel for MEPP used after sequence extraction ('pip install
    mepp')
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:d7cc2226d2723707a119465cc63472e7dffec0d43ad58d4babf0b43b138aebda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  - pip
  edam_operation: http://edamontology.org/operation_0384
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# genomic-sequence-extraction-around-anchor-motifs

## Summary

Extract fixed-width genomic sequences centered on biologically meaningful anchor points (transcription factor motif sites, transcription start sites, or Tn5/MNase cleavage sites) so that positional motif enrichment profiles can be computed relative to those anchors with tools like MEPP.

## When to use

Use when a positional motif enrichment analysis (e.g. MEPP) requires centered sequences around anchors such as deduplicated, RepeatMasker-filtered motif sites, TSSs, or cleavage sites, with a score attached per sequence.

## When NOT to use

- Sequences are already centered and scored for MEPP — re-extraction would change positional offsets relative to the anchor
- The goal is de novo motif discovery rather than profiling known motifs relative to anchors — MEPP currently functions only with a fixed motif library
- Motif enrichment is needed independent of position within the sequence — standard MEA methods (e.g. HOMER, MEIRLOP under a ZOOPs model) suffice and do not require centered anchors

## Inputs

- Anchor coordinate list (deduplicated, RepeatMasker-filtered motif sites, TSSs, or cleavage sites) in a genome assembly such as mm10
- Reference genome FASTA for sequence extraction (e.g. mm10)
- Per-anchor biological activity scores (e.g. ChIP-seq signal from GSE203135, DESeq2 log2 fold change LPS-stimulated vs resting)
- Window width specification (±200 bp around each anchor)

## Outputs

- Scored FASTA file with headers of the form '>sequence_name sequence_score', sequences centered on the anchor
- Anchor-centered sequence set ready for MEPP positional profiling (--fa input)

## How to apply

Define the anchor coordinates first: deduplicated, RepeatMasker-filtered motif sites (e.g. Nanog motif sites in mm10), 5'-end TSS coordinates for csRNA-seq, or cleavage sites for ATAC-seq and MNase-seq. Extract ±200 bp of sequence around each anchor from a reference genome assembly (e.g. mm10), with the anchor at the center so that positional offsets in downstream profiles are directly interpretable in bp relative to the anchor. Attach a biological activity score to each sequence in the FASTA header (format '>sequence_name sequence_score'), such as Nanog ChIP-seq signal from GSE203135 or DESeq2 log2 fold change (LPS-stimulated vs resting), which lets MEPP correlate motif presence with activity across a score continuum rather than arbitrary thresholds. Filter sequences with excessive degenerate bases using MEPP's --dgt percentage (rejecting sequences with N content above the threshold) to exclude repeats. Verify centering by checking that downstream MEPP peaks occur at interpretable offsets, e.g. a Sox2 motif periodicity of ~10 bp flanking central Nanog motifs, or an NF-kB peak 58 bp upstream of the csRNA-seq TSS.

## Related tools

- **MEPP** (Downstream consumer of the extracted anchor-centered scored FASTA; requires sequences with headers '>sequence_name sequence_score' and rejects sequences above the --dgt degenerate-base percentage) — https://github.com/npdeloss/mepp
- **pip** (Installation channel for MEPP used after sequence extraction ('pip install mepp')) — https://github.com/npdeloss/mepp

## Examples

```
mepp --fa nanog_sites_200bp.scored.fa --motifs jaspar_2022_motifs.txt --out mepp_output --dgt 20
```

## Evaluation signals

- Sequence length is uniform (e.g. 401 bp for ±200 bp extraction) and the anchor sits at the center such that MEPP's default plot center (--center, half the sequence length) coincides with the anchor
- Each FASTA header parses as '>sequence_name sequence_score' — MEPP treats the header score field as the activity continuum
- Sequences with degenerate base content above the --dgt percentage are excluded (default 100, lowered to filter repeats)
- Downstream MEPP profile shows positionally interpretable peaks: ~10 bp periodic Sox2 enrichment flanking central Nanog motifs, or NF-kB peaks at 58 bp upstream of csRNA-seq TSS, ~45 bp downstream of Tn5 cleavage site, and 81 bp upstream of MNase-seq cleavage site
- Anchors are deduplicated and RepeatMasker-filtered so duplicated or repetitive loci do not dominate the profile

## Limitations

- MEPP analyzes only the extracted window; peaks are interpretable only if the anchor is exactly centered (default --center is half the sequence length rounded down)
- MEPP currently functions with a fixed motif library; no de novo motif recognition is performed on the extracted sequences
- Sequences with high degenerate (non-ACGT) content, e.g. from repeats, are rejected based on the --dgt percentage and are silently dropped from the analysis

## Evidence

- [other] Extract ±200 bp sequences centered on the deduplicated, RepeatMasker-filtered Nanog motif sites from the mm10 genome assembly.: [evidence span withheld — anchor status: not_found]
- [readme] README scored fasta header format: [evidence span withheld — anchor status: external_doc]
- [readme] README --dgt filter description: [evidence span withheld — anchor status: external_doc]
- [abstract] Article anchor points statement: "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs)"
- [other] Source card TSS/cleavage site anchors: [evidence span withheld — anchor status: not_found]
