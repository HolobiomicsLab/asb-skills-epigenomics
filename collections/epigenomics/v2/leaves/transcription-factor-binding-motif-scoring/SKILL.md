---
name: transcription-factor-binding-motif-scoring
description: Use when you have anchor-centered sequences (FASTA with per-sequence
  scores in headers) and a motif PWM, and need motif match positions/scores relative
  to the anchor for positional enrichment profiling.
when_to_use_negative:
- You need de novo motif discovery — MEPP currently only functions with a fixed motif
  library and does not recognize or assemble new motifs
- You need a single aggregate motif enrichment score independent of position — MEPP
  outputs a positional profile rather than a position-independent enrichment statistic
- Your sequences are not centered on a biologically meaningful anchor point (e.g.
  TSS or peak summit) — the positional profile requires a common anchor to be interpretable
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: MEPP
  role: Runs the motif scoring (via MOODS) and positional enrichment profiling on
    the scored, anchor-centered sequences
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installs MEPP from PyPI ('pip install mepp'), the preferred method yielding
    the most recent stable release
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:629d33f2d6955c23efb00ed88f055dfdd801b295ec852bee5768beba9a46ab1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  - pip
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# transcription-factor-binding-motif-scoring

## Summary

Score centered genomic sequences (e.g. ±200 bp around MACS2 ChIP-seq peak summits) for TF binding motif occurrences using a position weight matrix (e.g. the GATA1 PWM from JASPAR/HOMER-format motif files), producing motif match positions and scores that MEPP uses to build a position-dependent enrichment profile around the anchor point.

## When to use

Use when you have anchor-centered sequences (FASTA with per-sequence scores in headers) and a motif PWM, and need motif match positions/scores relative to the anchor for positional enrichment profiling.

## When NOT to use

- You need de novo motif discovery — MEPP currently only functions with a fixed motif library and does not recognize or assemble new motifs
- You need a single aggregate motif enrichment score independent of position — MEPP outputs a positional profile rather than a position-independent enrichment statistic
- Your sequences are not centered on a biologically meaningful anchor point (e.g. TSS or peak summit) — the positional profile requires a common anchor to be interpretable

## Inputs

- Scored FASTA file with headers of the form '>sequence_name sequence_score' (e.g. ±200 bp sequences around MACS2 peak summits scored by log2 fold change)
- Motif matrices file in JASPAR format (e.g. GATA1 PWM; HOMER-derived motifs available in mepp's data/homer.motifs.txt)
- ENCODE ChIP-seq peak calls and control peak calls (e.g. K562 GATA1)

## Outputs

- Per-position motif match scores across all centered sequences (motif heatmaps)
- Positional enrichment profile of the motif relative to the anchor point (e.g. gata1_mepp_profile.png and .svg plots, 300 DPI default)
- Clustering table with positional correlation p-values adjusted via statsmodels multiple testing (default fdr_by, alpha 0.01)

## How to apply

Retrieve the peak calls of interest (e.g. ENCODE K562 GATA1 ChIP-seq and control), extract ±200 bp sequences around each MACS2 peak summit, and attach the parent peak's MACS2 log2 fold change as the sequence score so MEPP receives the score-based continuum of biological interest it requires. Prepare the motif matrix in JASPAR format (HOMER-derived motif files such as data/homer.motifs.txt ship with the repo, with a reduced-redundancy clustered version for faster analysis); motif matching is performed via MOODS with default parameters: p-value 0.0001, pseudocount 0.0001, and a 0.25/0.25/0.25/0.25 background, with orientations limited to '+' or '+/-'. Pass the scored FASTA (headers of the form '>sequence_name sequence_score') and motif file to mepp, optionally filtering sequences with excessive degenerate bases via --dgt and controlling match smoothing with --margin (default 2 bases). The tool scores every position of every sequence for motif presence, blurs matches by the margin, and correlates positional motif density with the sequence scores to yield the positional enrichment profile; sequences are batched through Tensorflow datasets (default batch 1000) with 1000 permutations for confidence intervals.

## Related tools

- **MEPP** (Runs the motif scoring (via MOODS) and positional enrichment profiling on the scored, anchor-centered sequences) — https://github.com/npdeloss/mepp
- **pip** (Installs MEPP from PyPI ('pip install mepp'), the preferred method yielding the most recent stable release) — https://github.com/npdeloss/mepp

## Examples

```
mepp --fa gata1_summits_scored.fa --motifs jaspar_gata1.txt --out mepp_out
```

## Evaluation signals

- The output plot (e.g. gata1_mepp_profile.png) exists and displays centrally positioned GATA1 motif enrichment around the summit anchor, qualitatively matching the published Figure 2 profile
- The GATA1 motif enrichment at central positions correlates with higher MACS2 log2 fold change (ChIP over control), as expected biologically
- For promoter applications, the profile recapitulates known relationships such as core promoter element positioning around Drosophila melanogaster TSS
- Sequence headers follow the required '>sequence_name sequence_score' format, and the motif file parses as valid JASPAR-format matrices
- Profile shape is less sharp than single-nucleotide-resolution TSS-centered analyses when using peak summits as anchors, consistent with coarser positional landmarks

## Limitations

- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work
- No software version is specified for MEPP
- Permutation testing can lead to significant GPU memory usage; --gjobs must be set carefully to avoid jobs crowding each other out of GPU memory, causing profile generation to fail
- Profiles from coarser anchors (e.g. ChIP-seq peak summits) are less sharp than single-nucleotide-resolution TSS-centered analyses
- Sequences exceeding the --dgt degenerate-base percentage threshold are rejected from the analysis, which can filter out repeat-rich regions

## Evidence

- [supplementary] MEPP analysis of scored GATA1 ChIP-seq peak summit sequences found centrally positioned enrichment of the GATA1 motif correlated with higher coverage in GATA1 ChIP over control, as expected.: [evidence span withheld — anchor status: not_found]
- [readme] Motif matching is done via MOODS with configurable p-value threshold.: "--pval FLOAT                    P-value for setting motif match threshold
                                      via MOODS. Default: 0.0001"
- [readme] Motif files in HOMER format ship in the data subdirectory.: "These are motifs from the `HOMER <http://homer.ucsd.edu/homer/>`_ suite in data/homer.motifs.txt, as well as a reduced-redundancy version with similar motifs clustered"
- [abstract] MEPP outputs positional enrichment profiles relative to anchor points while controlling sequence bias.: "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs)"
- [discussion] MEPP requires a score-based continuum of biological interest, as with MEIRLOP.: "Similar to our previous work with MEIRLOP, MEPP is usable by scoring genomic regions across a continuum of scores reflecting two extremes of biological interest."
- [methods] Install MEPP via pip as the preferred method.: "To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp"
