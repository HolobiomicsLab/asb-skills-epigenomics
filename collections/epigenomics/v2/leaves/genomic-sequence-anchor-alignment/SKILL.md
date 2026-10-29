---
name: genomic-sequence-anchor-alignment
description: Use when sequences are centered on an anchor point (e.g. TSS) with a
  continuous per-sequence score, and the goal is to find motif positions whose occurrence
  correlates with that score.
when_to_use_negative:
- The question is motif presence/absence across uncentered sequences without regard
  to motif position — standard MEA tools like HOMER under a ZOOPs model are more appropriate
- 'You need de novo motif discovery: MEPP currently only functions with a fixed motif
  library; de novo recognition/assembly is left to future work'
- Sequences lack a continuous per-sequence score or a meaningful shared anchor point,
  so positional correlations relative to the anchor cannot be computed
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3179
tools:
- name: MEPP
  role: 'Core tool: computes the positional enrichment profile of each motif in a
    fixed JASPAR-format library relative to anchor points within scored sequences,
    accounting for lower-order nucleotide bias'
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installation channel for MEPP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:27bc934b5bd3536637d0697971318f0d2a7dbc3c0002c730b69d12aa56353a25
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
  - http://edamontology.org/topic_3179
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# genomic-sequence-anchor-alignment

## Summary

Positional motif enrichment profiling with MEPP: compute position-dependent enrichment profiles for a fixed motif library across scored DNA sequences centered on biologically meaningful anchor points (e.g. TSSs), revealing where motif presence correlates with a score continuum.

## When to use

Use when sequences are centered on an anchor point (e.g. TSS) with a continuous per-sequence score, and the goal is to find motif positions whose occurrence correlates with that score.

## When NOT to use

- The question is motif presence/absence across uncentered sequences without regard to motif position — standard MEA tools like HOMER under a ZOOPs model are more appropriate
- You need de novo motif discovery: MEPP currently only functions with a fixed motif library; de novo recognition/assembly is left to future work
- Sequences lack a continuous per-sequence score or a meaningful shared anchor point, so positional correlations relative to the anchor cannot be computed

## Inputs

- Scored FASTA file with headers of the form '>sequence_name sequence_score' (e.g. ±200 bp sequences centered on 44,877 high-confidence Drosophila csRNA-seq TSSs scored by log-transformed csRNA-seq coverage)
- Motif matrices file in JASPAR format (e.g. data/ohler_motifs.txt for Drosophila core promoter motifs, data/homer.motifs.txt for HOMER motifs)

## Outputs

- Position-dependent motif enrichment profiles (MEPP plots) centered on the anchor point, in png and svg formats at 300 DPI
- Motif heatmaps with adaptive brightness scaling (sigma * std of nonzero match scores)
- Clustering table of MEPP profiles with FDR-adjusted positional correlation p-values (fdr_by, alpha 0.01)

## How to apply

Prepare a scored FASTA in which each header carries the sequence's continuous score (e.g. log-transformed csRNA-seq coverage), so MEPP can exploit its score-continuum design without arbitrary thresholding. Install MEPP via pip and select a motif matrix file in JASPAR format — the README ships HOMER motifs (data/homer.motifs.txt, with a reduced-redundancy version) and Ohler Drosophila core promoter motifs (data/ohler_motifs.txt). Run 'mepp' with --fa, --motifs and --out, optionally setting --center (0-based offset to anchor plots on, default half the sequence length), --dgt to reject sequences exceeding a degenerate-base percentage, --perms for permutation testing/confidence intervals (default 1000), and --pval 0.0001 with --pcount 0.0001 for MOODS match thresholds. MEPP profiles positional enrichment relative to the anchor point while controlling for lower-order nucleotide bias, producing motif heatmaps and clustering tables with multiple-testing-corrected positional correlation p-values. Extract each motif's peak position from its enrichment profile and interpret it as an offset from the anchor; validation is done by checking peaks against known biology (e.g. Inr at 0 bp, TATA-box at −28 bp, DPE at +25 bp around Drosophila TSSs).

## Related tools

- **MEPP** (Core tool: computes the positional enrichment profile of each motif in a fixed JASPAR-format library relative to anchor points within scored sequences, accounting for lower-order nucleotide bias) — https://github.com/npdeloss/mepp
- **pip** (Installation channel for MEPP)

## Examples

```
pip install mepp; mepp --fa tss_scored.fa --motifs data/ohler_motifs.txt --out mepp_tss_profiles --perms 1000 --pval 0.0001
```

## Evaluation signals

- TATA-box enrichment profile peaks at −28 bp upstream of the csRNA-seq TSS anchor, DPE at +25 bp downstream, and Inr at the TSS center (offset 0 bp), matching known Drosophila core promoter organization
- Scored FASTA headers parse as '>sequence_name sequence_score'; sequences exceeding the --dgt degenerate-base percentage are rejected from the analysis
- Positional profile confidence intervals derive from the specified --perms permutation count (default 1000), and clustering-table p-values are multiple-testing adjusted (default fdr_by at alpha 0.01)
- Profiles exploit the score continuum with no arbitrary thresholding of sequences, and lower-order nucleotide bias is controlled as stated in the abstract

## Limitations

- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work (convolutional network architecture noted as a path forward)
- No software version specified for MEPP and no changelog found for the repository
- Permutation testing (--perms) can lead to significant GPU memory usage; GPU job counts (--gjobs) must be set carefully to avoid jobs crowding each other out of GPU memory

## Evidence

- [abstract] outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs) within the analyzed sequences while accounting: "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs) within the analyzed sequences while accounting"
- [discussion] MEPP currently functions with a fixed motif library. However, the underlying convolutional network architecture lends itself easily to future work for recognizing and assembling de novo motifs based: "MEPP currently functions with a fixed motif library. However, the underlying convolutional network architecture lends itself easily to future work for recognizing and assembling de novo motifs based"
- [readme] Path to a scored fasta file, where sequence headers are of the form: ">sequence_name sequence_score".: [evidence span withheld — anchor status: external_doc]
- [readme] The file data/ohler_motifs.txt contains Drosophila core promoter motifs from Ohler et al.: "The file data/ohler_motifs.txt contains Drosophila core promoter motifs from Ohler et al."
- [discussion] These results recapitulate known relationships such as the positioning of core promoter elements surrounding drosophila melanogaster TSS: "These results recapitulate known relationships such as the positioning of core promoter elements surrounding drosophila melanogaster TSS"
