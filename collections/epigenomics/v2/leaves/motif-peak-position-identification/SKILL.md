---
name: motif-peak-position-identification
description: Use when you have centered genomic sequences (e.g. ±200 bp around TSSs)
  with a continuous per-sequence score (e.g. log-transformed csRNA-seq coverage) and
  need to determine the positional dependence of known motifs relative to the anchor
  point.
when_to_use_negative:
- You need de novo motif discovery/assembly — MEPP currently only functions with a
  fixed motif library
- Your sequences are not centered on a biologically meaningful anchor point (e.g.
  TSS or a motif), since positional profiles are only interpretable relative to such
  anchors
- You only need presence/absence motif enrichment without positional information —
  standard MEA tools like HOMER or MEIRLOP may suffice
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: MEPP
  role: 'Primary tool: computes position-dependent motif enrichment profiles over
    scored sequences and generates MEPP plots from which peak positions are extracted'
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installation mechanism for MEPP into the analysis environment
- name: HOMER
  role: Source of the motif library shipped with MEPP (data/homer.motifs.txt); HOMER
    itself uses a ZOOPs model in contrast to MEPP's score-continuum approach
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:e1b956e6be49d4d70f49518b0e84ac5691544902f2a7880de8e0240dd0137dbb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - HOMER
  - MEPP
  - pip
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# motif-peak-position-identification

## Summary

Uses MEPP (Motif Enrichment Positional Profiling) to compute position-dependent motif enrichment profiles across scored DNA sequences centered on an anchor point (e.g. a TSS), then extracts each motif's peak offset to identify where motif presence functionally correlates with a score such as csRNA-seq coverage.

## When to use

Use when you have centered genomic sequences (e.g. ±200 bp around TSSs) with a continuous per-sequence score (e.g. log-transformed csRNA-seq coverage) and need to determine the positional dependence of known motifs relative to the anchor point.

## When NOT to use

- You need de novo motif discovery/assembly — MEPP currently only functions with a fixed motif library
- Your sequences are not centered on a biologically meaningful anchor point (e.g. TSS or a motif), since positional profiles are only interpretable relative to such anchors
- You only need presence/absence motif enrichment without positional information — standard MEA tools like HOMER or MEIRLOP may suffice

## Inputs

- Scored FASTA file with headers of the form '>sequence_name sequence_score' (sequences centered on the anchor point, e.g. ±200 bp around csRNA-seq TSSs scored by log-transformed coverage)
- Motif matrices file in JASPAR format (e.g. data/ohler_motifs.txt for Drosophila core promoter motifs, or data/homer.motifs.txt)
- Output directory path for MEPP plots and tables

## Outputs

- Positional enrichment profile (MEPP plot) per motif, showing enrichment as a function of position relative to the anchor point
- Peak position (bp offset) for each motif relative to the anchor (e.g. TATA-box −28 bp, DPE +25 bp, Inr 0 bp)
- Clustering table with positional correlation p-values (multiple-testing adjusted, default fdr_by at alpha 0.01)
- MEPP plot images in png and svg formats (default 300 DPI, 10x10 inches)

## How to apply

Install MEPP via pip ($ pip install mepp) and prepare a scored FASTA whose headers are of the form '>sequence_name sequence_score', with sequences centered on the anchor point (e.g. 44,877 high-confidence Drosophila csRNA-seq TSSs, ±200 bp). Supply a motif matrix file in JASPAR format — for Drosophila core promoter analysis use the Ohler Drosophila motif library shipped in data/ohler_motifs.txt. Run MEPP with --center set to the TSS offset (default: half the sequence length, rounded down) so MEPP exploits its score-continuum design (no arbitrary thresholding) and computes a position-dependent enrichment profile per motif while controlling for lower-order nucleotide bias. Extract each motif's peak position from its enrichment profile and record the offset relative to the TSS center; optionally set --dgt to filter sequences with excessive degenerate bases and use the default 1000 permutations for confidence intervals. Correct application is judged by whether extracted peaks recapitulate known biology, e.g. Inr peaking at the TSS center (0 bp), TATA-box at −28 bp, and DPE at +25 bp.

## Related tools

- **MEPP** (Primary tool: computes position-dependent motif enrichment profiles over scored sequences and generates MEPP plots from which peak positions are extracted) — https://github.com/npdeloss/mepp
- **pip** (Installation mechanism for MEPP into the analysis environment)
- **HOMER** (Source of the motif library shipped with MEPP (data/homer.motifs.txt); HOMER itself uses a ZOOPs model in contrast to MEPP's score-continuum approach)

## Examples

```
mepp --fa tss_sequences.fa --motifs data/ohler_motifs.txt --out mepp_profiles --center 200 --dgt 25
```

## Evaluation signals

- Extracted peak offsets recapitulate known Drosophila core promoter element positioning: Inr maximally correlated at the TSS center (offset 0 bp), TATA-box at −28 bp upstream, and DPE at +25 bp downstream
- Scored FASTA headers parse correctly as '>sequence_name sequence_score'; MEPP exits without rejecting sequences (check --dgt degenerate-base rejection is not filtering out data unexpectedly)
- MEPP plots are generated for every motif (default 10 retry attempts, png+svg at 300 DPI) and profiles show a clear maximum near the expected offset rather than flat enrichment
- Positional correlation p-values in the clustering table survive multiple-testing correction (default fdr_by, alpha 0.01, thorough testing enabled)
- Confidence intervals (default 95%) from the default 1000 permutations do not contradict the identified peak positions

## Limitations

- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work
- No software version specified for MEPP and no changelog found for the repository
- Permutation testing (--perms, default 1000) can lead to significant GPU memory usage; GPU job crowding (--gjobs) can cause profile generation to fail, with up to 10 retry attempts by default
- Sequences exceeding the --dgt degenerate-base percentage (default 100, i.e. no filtering unless lowered) are rejected, which is useful for filtering out repeats but can silently drop data

## Evidence

- [abstract] We developed motif enrichment positional profiling (MEPP), a novel MEA method that outputs a positional enrichment profile of a given TF's binding motif: "We developed motif enrichment positional profiling (MEPP), a novel MEA method that outputs a positional enrichment profile of a given TF's binding motif"
- [methods] To install Motif Enrichment Positional Profiling, run this command in your terminal: $ pip install mepp: "To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp"
- [discussion] These results recapitulate known relationships such as the positioning of core promoter elements surrounding drosophila melanogaster TSS: "These results recapitulate known relationships such as the positioning of core promoter elements surrounding drosophila melanogaster TSS"
- [readme] The file data/ohler_motifs.txt contains Drosophila core promoter motifs from Ohler et al.: "The file data/ohler_motifs.txt contains Drosophila core promoter motifs from `Ohler et al. <https://pubmed.ncbi.nlm.nih.gov/12537576/>`_."
- [readme] --fa: Path to a scored fasta file, where sequence headers are of the form: ">sequence_name sequence_score".: "Path to a scored fasta file, where sequence
                                      headers are of the form: ">sequence_name
                                      sequence_score"."
