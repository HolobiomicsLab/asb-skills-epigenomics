---
name: signal-score-continuum-enrichment-analysis
description: Use when you have centered genomic sequences (e.g. ±200 bp around peak
  summits) plus a per-sequence continuous score (e.g. MACS2 narrowpeak signal scores)
  and need motif enrichment as a function of position rather than a single enrichment
  statistic.
when_to_use_negative:
- 'You need de novo motif recognition or motif assembly: MEPP currently functions
  with a fixed motif library only'
- Your sequences are not centered on a biologically meaningful anchor point (e.g.
  TSS or peak summit), since positional enrichment profiles are relative to that center
- You only have unscored sequences with no per-sequence continuum of biological activity,
  as MEPP's --fa option requires a score in every sequence header
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: MEPP
  role: 'Primary tool: computes the position-dependent motif enrichment profile across
    scored sequences centered on anchor points, generating MEPP plots and clustering
    tables'
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installation channel for MEPP into the execution environment (pip install
    mepp)
- name: CentriMo
  role: Contrasted bin-based approach to motif-position analysis that MEPP improves
    upon; not used in the workflow
- name: TFEA
  role: Contrasted quartile-based approach to motif-position analysis that MEPP improves
    upon; not used in the workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:a3e98637b8c33dde4316598634e9bc72f8c579ee64e1bcf29a24d9b3ed0919be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - CentriMo
  - MEPP
  - TFEA
  - pip
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# signal-score-continuum-enrichment-analysis

## Summary

Score-based motif enrichment analysis (MEA) using MEPP, which outputs a positional enrichment profile of a transcription factor binding motif relative to anchor points by scoring sequences along a continuum of biological activity (e.g. MACS2 narrowpeak signal scores), avoiding arbitrary peak thresholds.

## When to use

Use when you have centered genomic sequences (e.g. ±200 bp around peak summits) plus a per-sequence continuous score (e.g. MACS2 narrowpeak signal scores) and need motif enrichment as a function of position rather than a single enrichment statistic.

## When NOT to use

- You need de novo motif recognition or motif assembly: MEPP currently functions with a fixed motif library only
- Your sequences are not centered on a biologically meaningful anchor point (e.g. TSS or peak summit), since positional enrichment profiles are relative to that center
- You only have unscored sequences with no per-sequence continuum of biological activity, as MEPP's --fa option requires a score in every sequence header

## Inputs

- Scored FASTA file with headers of the form '>sequence_name sequence_score' (sequences centered on peak summits, e.g. ±200 bp)
- Motif matrices file in JASPAR format (e.g. Nanog motif; HOMER motifs available in data/homer_motifs.txt, Drosophila core promoter motifs in data/ohler_motifs.txt)
- Per-sequence continuum of biological activity scores (e.g. MACS2 narrowpeak signal scores from ~39,000 Nanog ChIP-nexus peaks or ~28,000 Nanog ChIP-seq peaks)

## Outputs

- Per-motif positional enrichment profile plots relative to sequence centers (png and svg, default 300 dpi)
- Motif match heatmaps colored by a matplotlib colormap (default gray_r)
- Clustering table of positional correlations with multiple-testing-adjusted p-values (default fdr_by, alpha 0.01)
- Side-by-side positional profiles enabling comparison of positional specificity between datasets

## How to apply

Extract fixed-width genomic sequences centered on anchor points (e.g. ±200 bp around each Nanog ChIP-nexus or ChIP-seq peak summit) and encode each sequence's MACS2 narrowpeak signal score in the FASTA header as '>sequence_name sequence_score', the format MEPP's --fa option requires. Install MEPP with pip (pip install mepp, or pip install git+https://github.com/npdeloss/mepp@main) and import it (import mepp) or run the mepp CLI with a JASPAR-format motif matrix file. Let the continuous signal scores drive enrichment so no arbitrary threshold selection is needed, while MEPP controls for sequence bias; motif matches are called via MOODS with default pval 0.0001 and pcount 0.0001. For ChIP-seq-derived sequences scan both strands using --orientations +/-, since ChIP-seq does not resolve motif strand, whereas ChIP-nexus retains strand resolution. MEPP uses 1000 permutations (--perms) for permutation testing and confidence intervals (default 95.0% CI), and adjusts positional-correlation p-values with statsmodels multipletests (default fdr_by at alpha 0.01). Render per-motif positional profiles relative to peak centers side by side across datasets to compare positional specificity.

## Related tools

- **MEPP** (Primary tool: computes the position-dependent motif enrichment profile across scored sequences centered on anchor points, generating MEPP plots and clustering tables) — https://github.com/npdeloss/mepp
- **pip** (Installation channel for MEPP into the execution environment (pip install mepp))
- **CentriMo** (Contrasted bin-based approach to motif-position analysis that MEPP improves upon; not used in the workflow)
- **TFEA** (Contrasted quartile-based approach to motif-position analysis that MEPP improves upon; not used in the workflow)

## Examples

```
pip install mepp; mepp --fa nanog_nexus_summits.fa --motifs jaspar_motifs.txt --out mepp_nexus_output --orientations +/- --perms 1000
```

## Evaluation signals

- For Nanog ChIP-nexus peaks (~39,000), the positional profile must show a sharp central Nanog motif peak centered directly on the peak summit with resolvable flanking Oct4-Sox2-TCF-Nanog twin peaks
- The corresponding Nanog ChIP-seq profile (~28,000 peaks) must be broader and of lower amplitude, matching the reported Figure 5 comparison demonstrating greater ChIP-nexus positional specificity
- Output directory contains MEPP plots in both png and svg formats at the requested DPI (default 300)
- Clustering table p-values are multiple-testing adjusted with fdr_by at alpha 0.01, and profiles are clustered with scipy linkage using the correlation metric (default)
- Permutation-based confidence intervals (default 95.0%) are computed with --perms 1000 without GPU memory failures across processing batches (default --batch 1000)

## Limitations

- MEPP currently functions only with a fixed motif library; de novo motif recognition/assembly is left to future work
- Permutation testing (--perms, default 1000) can lead to significant GPU memory usage; --gjobs must be set carefully to avoid jobs crowding each other out of GPU memory, causing profile generation to fail
- No software version specified for MEPP and no changelog found for the repository
- Sequences exceeding the degenerate-base tolerance (--dgt, percentage of sequence that can be non-ACGT) are rejected from the analysis, which can filter out repetitive regions

## Evidence

- [abstract] MEPP abstract claim: "We developed motif enrichment positional profiling (MEPP), a novel MEA method that outputs a positional enrichment profile of a given TF's binding motif"
- [intro] Score-based principle: "The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias."
- [methods] pip installation: "To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp"
- [readme] Scored FASTA requirement: "Path to a scored fasta file, where sequence
                                      headers are of the form: ">sequence_name
                                      sequence_score"."
- [readme] Degenerate base filtering: "Percentage of sequence that can be
                                      degenerate (Not A, C, G, or T) before being
                                      rejected from the analysis."
- [readme] Purpose statement: "quantifies a positional profile of motif enrichment along the length of DNA sequences centered on e.g. transcription start sites"
