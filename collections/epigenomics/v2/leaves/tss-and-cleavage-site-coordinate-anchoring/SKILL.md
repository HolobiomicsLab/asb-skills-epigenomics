---
name: tss-and-cleavage-site-coordinate-anchoring
description: Use when a motif enrichment analysis requires positional interpretation,
  i.e. you have assay-derived anchor coordinates (TSSs or cleavage sites) and want
  motif enrichment profiled at offsets around that anchor rather than presence/absence
  per sequence.
when_to_use_negative:
- Sequences are genomic windows not anchored at a TSS or cleavage site, so positional
  offsets would be biologically meaningless
- The question is simple motif presence/absence enrichment under a zero-or-one-occurrence-per-sequence
  (ZOOPs) model, where position is irrelevant (e.g. standard HOMER-style MEA)
- De novo motif discovery is needed, since MEPP currently functions only with a fixed
  motif library
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: MEPP
  role: Generates the positional enrichment profile of a motif relative to the anchored
    coordinates (TSS or cleavage sites) using scored sequences
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installation channel for MEPP used before anchoring analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:054940a3eadde456024ad1983bcda0156d92c46fe57ca35513fb88e76e5a7a3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  - pip
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# tss-and-cleavage-site-coordinate-anchoring

## Summary

Anchor genomic sequences to biologically meaningful 5'-end coordinates — transcription start sites from csRNA-seq or Tn5/MNase cleavage sites from ATAC-seq and MNase-seq — so that positional motif enrichment (MEPP) profiles can be interpreted relative to a common center across assays.

## When to use

Use when a motif enrichment analysis requires positional interpretation, i.e. you have assay-derived anchor coordinates (TSSs or cleavage sites) and want motif enrichment profiled at offsets around that anchor rather than presence/absence per sequence.

## When NOT to use

- Sequences are genomic windows not anchored at a TSS or cleavage site, so positional offsets would be biologically meaningless
- The question is simple motif presence/absence enrichment under a zero-or-one-occurrence-per-sequence (ZOOPs) model, where position is irrelevant (e.g. standard HOMER-style MEA)
- De novo motif discovery is needed, since MEPP currently functions only with a fixed motif library

## Inputs

- TSS coordinates from csRNA-seq 5'-ends (GSE203135, LPS-stimulated and resting BMDMs)
- Cleavage-site coordinates from ATAC-seq (Tn5) and H3K27ac MNase-seq 5'-ends
- Reference genome for extracting ±200 bp anchored sequences
- DESeq2 log2 fold changes (LPS vs resting) as per-sequence activity scores
- Scored FASTA with headers of the form ">sequence_name sequence_score"
- Motif matrix file in JASPAR format (e.g. NF-kB motif)

## Outputs

- Per-assay MEPP positional enrichment profiles centered on the anchor coordinate
- Motif heatmaps and enrichment plots at defined offsets (png/svg at 300 dpi)
- Verified positional peak offsets (e.g. 58 bp upstream of TSS in csRNA-seq; ~45 bp downstream of Tn5 site; 81 bp upstream of MNase site)
- Cross-assay concordance comparison of motif positional peaks

## How to apply

For each assay (csRNA-seq, ATAC-seq, MNase-seq) extract ±200 bp genomic sequences centered on the 5'-end coordinate: the transcription start site for csRNA-seq and the cleavage site for ATAC-seq (Tn5) and MNase-seq. Attach a continuous activity score to each anchored sequence — DESeq2 log2 fold change (LPS-stimulated vs resting BMDMs) — encoded in the FASTA header as ">sequence_name sequence_score", since MEPP expects a scored fasta. Center plots on the anchor by leaving --center at its default (half the sequence length) or setting an explicit 0-based offset, then run MEPP per assay with the motif of interest (e.g. NF-kB from JASPAR/HOMER-format matrices). Concordance is judged by whether the same motif shows enrichment peaks at consistent biological positions across assays, e.g. the NF-kB peak 58 bp upstream of the csRNA-seq TSS versus ~45 bp downstream of the Tn5 cleavage site and 81 bp upstream of the MNase-seq cleavage site.

## Related tools

- **MEPP** (Generates the positional enrichment profile of a motif relative to the anchored coordinates (TSS or cleavage sites) using scored sequences) — https://github.com/npdeloss/mepp
- **pip** (Installation channel for MEPP used before anchoring analysis)

## Examples

```
mepp --fa lps_anchored_sequences.fa --motifs nfkb.jaspar --out nfkb_mepp_out
```

## Evaluation signals

- Extracted sequences are exactly ±200 bp and share a common center coordinate (TSS or cleavage site) so MEPP's default plot center coincides with the anchor
- FASTA headers conform to ">sequence_name sequence_score" and scores are DESeq2 log2 fold changes, otherwise MEPP cannot rank sequences by activity
- Reproduced NF-kB positional peaks: positive peak 58 bp upstream of the csRNA-seq TSS, ~45 bp downstream of the Tn5 cleavage site, and 81 bp upstream of the MNase-seq cleavage site, matching Figure 6
- The NF-kB motif shows concordantly positive peaks across all three assays rather than assay-specific artifacts
- Motif match threshold is set by MOODS at --pval 0.0001 with --pcount 0.0001 and default background 0.25 x4

## Limitations

- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work
- No software version specified for MEPP, complicating reproducibility of anchored profile offsets
- Highly degenerate/repetitive sequences should be filtered via the --dgt option (percentage of non-ACGT tolerated before rejection)

## Evidence

- [abstract] MEPP outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points: "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs) within the analyzed sequences while accounting"
- [intro] There is a need for methods that visualize and quantify the spatial relationships between TF binding motifs and biologically relevant anchor points such as TSS.: "There is a need for methods that visualize and quantify the spatial relationships between TF binding motifs and biologically relevant anchor points such as TSS."
- [readme] MEPP quantifies a positional profile of motif enrichment along sequences centered on e.g. transcription start sites or transcription factor binding motifs.: "Motif Enrichment Positional Profiling (MEPP) quantifies a positional profile of motif enrichment along the length of DNA sequences centered on e.g. transcription start sites or transcription factor"
- [readme] MEPP input is a scored fasta with scores in sequence headers.: [evidence span withheld — anchor status: external_doc]
