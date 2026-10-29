---
name: motif-positional-enrichment-profiling
description: Use when you have a set of genomic sequences centered on a biological
  anchor (TSS, ChIP/ATAC/MNase peak summit, or a known motif site) plus a continuous
  score per sequence, and you need to know WHERE a motif enriches relative to that
  anchor rather than merely whether it is enriched.
when_to_use_negative:
- 'De novo motif discovery/assembly is required: MEPP currently functions only with
  a fixed motif library (though ''python -m mepp.learn_motifs'' offers initial motif
  learning)'
- The question is merely whether a motif is enriched, not where it enriches relative
  to an anchor — bin- or quartile-based MEA tools like CentriMo or TFEA, or ZOOPs-model
  tools like HOMER, suffice and give no positional profile
- Input sequences lack a meaningful continuous score per sequence (e.g. raw, unranked
  motif hits), since MEPP's design requires a score-based continuum of biological
  interest
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: MEPP
  role: 'Core tool: computes the position-dependent motif enrichment profile from
    scored, anchor-centered sequences using a Tensorflow/GPU-accelerated convolutional
    design'
  repo: https://github.com/npdeloss/mepp
- name: CentriMo
  role: Baseline bin-based central-enrichment comparator used to cross-check motif
    rankings (e.g. SCL/bHLH motifs in ATAC-seq differential analysis)
- name: TFEA
  role: Baseline quartile-based motif enrichment comparator for method comparison
    against MEPP's positional profiles
- name: HOMER
  role: Baseline ZOOPs-model comparator run on top-10% scored bins; also source of
    the motif matrices shipped in MEPP's data/homer.motifs.txt
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_006
  - task_007
  - task_003
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:df1cc433973c1df6e9dedde9231897794019b8aadba9ca14a469af23cf0598c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - CentriMo
  - HOMER
  - MEPP
  - TFEA
  - pip
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# motif-positional-enrichment-profiling

## Summary

MEPP (Motif Enrichment Positional Profiling) computes a position-dependent enrichment profile for transcription factor binding motifs across scored, anchor-centered genomic sequences (e.g. TSSs, peak summits, or other motif sites), revealing where motif presence correlates with a continuous biological activity score while controlling for lower-order nucleotide bias.

## When to use

Use when you have a set of genomic sequences centered on a biological anchor (TSS, ChIP/ATAC/MNase peak summit, or a known motif site) plus a continuous score per sequence, and you need to know WHERE a motif enriches relative to that anchor rather than merely whether it is enriched.

## When NOT to use

- De novo motif discovery/assembly is required: MEPP currently functions only with a fixed motif library (though 'python -m mepp.learn_motifs' offers initial motif learning)
- The question is merely whether a motif is enriched, not where it enriches relative to an anchor — bin- or quartile-based MEA tools like CentriMo or TFEA, or ZOOPs-model tools like HOMER, suffice and give no positional profile
- Input sequences lack a meaningful continuous score per sequence (e.g. raw, unranked motif hits), since MEPP's design requires a score-based continuum of biological interest

## Inputs

- Scored FASTA file with headers of the form '>sequence_name sequence_score' (e.g. ±200 bp sequences centered on csRNA-seq TSSs, MACS2 peak summits, or GATA1 motif sites, scored by log2 fold change or coverage)
- Motif matrices file in JASPAR format (e.g. data/homer.motifs.txt, reduced-redundancy version, or data/ohler_motifs.txt from the repository)
- Optional: per-sequence biological activity scores from DESeq2 or MACS2 (e.g. LPS vs resting log2 fold change, ChIP-nexus narrowpeak signal scores)

## Outputs

- Positional enrichment profile plots per motif relative to the anchor point (png and svg by default, --dpi 300)
- Central MEPP motif heatmap with brightness scaled by --sigma and colormap --cmap (default gray_r)
- Clustering table of motifs with adjusted positional-correlation p-values (multiple testing via statsmodels multipletests, default fdr_by, alpha 0.01)

## How to apply

Prepare a scored FASTA whose headers are of the form '>sequence_name sequence_score' (e.g. ±200 bp around MACS2 peak summits scored by log2 fold change, or csRNA-seq TSSs scored by log-transformed coverage), and a motif matrices file in JASPAR format (HOMER-derived or Ohler Drosophila promoter motifs ship in the repo's data subdirectory). Install via pip and run 'mepp' with required options --fa, --motifs, and --out; tune --orientations (use '+/-' when the assay does not resolve motif strand, as with ChIP-seq; use '+' for strand-resolved ChIP-nexus), --dgt to reject repeat-heavy sequences, and --perms for permutation-based confidence intervals. MEPP uses MOODS with p-value 0.0001 and pseudocount 0.0001 to set match thresholds, avoids arbitrary threshold selection by using the score continuum, and accounts for lower-order nucleotide bias via its convolutional design on Tensorflow datasets (GPU optional via --nogpu). Validate the resulting positional profile by checking that known motif-anchor offsets are recovered, e.g. TATA-box peaking at -28 bp and DPE at +25 bp relative to a Drosophila TSS, or a ~10 bp helical periodicity of Sox2 around central Nanog motifs.

## Related tools

- **MEPP** (Core tool: computes the position-dependent motif enrichment profile from scored, anchor-centered sequences using a Tensorflow/GPU-accelerated convolutional design) — https://github.com/npdeloss/mepp
- **CentriMo** (Baseline bin-based central-enrichment comparator used to cross-check motif rankings (e.g. SCL/bHLH motifs in ATAC-seq differential analysis))
- **TFEA** (Baseline quartile-based motif enrichment comparator for method comparison against MEPP's positional profiles)
- **HOMER** (Baseline ZOOPs-model comparator run on top-10% scored bins; also source of the motif matrices shipped in MEPP's data/homer.motifs.txt)

## Examples

```
pip install mepp; mepp --fa scored_summits.fa --motifs jaspar_motifs.txt --out mepp_out --orientations +/-
```

## Evaluation signals

- Output directory (--out) contains per-motif MEPP plots in the requested formats (default png,svg at 300 dpi) showing a positional profile centered on the --center offset
- Known anchor-motif offsets are recovered: Inr at 0 bp, TATA-box at -28 bp, DPE at +25 bp relative to Drosophila csRNA-seq TSSs
- Motif spacing structure appears as expected, e.g. SCL/TAL1 enrichment peak ~12 bp upstream of GATA1 motifs in K562 accessibility-scored sequences, or Sox2 ~10 bp helical periodicity around Nanog motifs
- Clustering table p-values are adjusted with the configured multiple-testing method (default fdr_by, alpha 0.01) and pass the stated FWER threshold
- For ChIP-nexus data, the profile shows a sharp central Nanog motif peak with flanking Oct4-Sox2-TCF-Nanog twin peaks, sharper than the broader ChIP-seq profile run in parallel

## Limitations

- MEPP currently functions only with a fixed motif library; de novo motif recognition/assembly is left to future work
- No software version is specified in the article, and the repository ships no changelog
- Permutation testing can consume significant GPU memory (--perms default 1000); GPU job crowding can cause profile generation to fail, mitigated by --gjobs tuning, --attempts retries, or --nogpu
- Positional profiles from low-resolution landmarks (e.g. ChIP-seq summits) are less sharp than single-nucleotide-resolution TSS-centered analyses

## Evidence

- [abstract] we demonstrate MEPP's utility in determining the sequence positions where motif presence correlates with measures of biological activity, inferring positional dependencies of binding site function.: "we demonstrate MEPP's utility in determining the sequence positions where motif presence correlates with measures of biological activity, inferring positional dependencies of binding site function."
- [methods] To install Motif Enrichment Positional Profiling, run this command in your terminal: $ pip install mepp: "To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp"
- [readme] Path to a scored fasta file, where sequence headers are of the form: ">sequence_name sequence_score".: "Path to a scored fasta file, where sequence headers are of the form: ">sequence_name
                                      sequence_score"."
- [discussion] MEPP currently functions with a fixed motif library. However, the underlying convolutional network architecture lends itself easily to future work for recognizing and assembling de novo motifs: "MEPP currently functions with a fixed motif library. However, the underlying convolutional network architecture lends itself easily to future work for recognizing and assembling de novo motifs based"
- [readme] Motif files for use with this program can be found in the data subdirectory. These are motifs from the HOMER suite in data/homer.motifs.txt: "Motif files for use with this program can be found in the data subdirectory.
These are motifs from the `HOMER <http://homer.ucsd.edu/homer/>`_ suite in data/homer.motifs.txt"
