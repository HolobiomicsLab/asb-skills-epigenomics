---
name: gata1-tal1-motif-spacing-analysis
description: Use when testing whether cell-type-specific chromatin accessibility associates
  with the spacing of a co-occurring TF motif (e.g. SCL/bHLH) relative to an anchor
  motif (e.g. GATA1) in scored centered sequences.
when_to_use_negative:
- The question is plain motif presence/absence enrichment without positional information
  — most MEA methods analyze sequences 'without regard to the motif's position within
  the sequence'; use a standard MEA instead.
- De novo motif discovery is required — MEPP currently functions only with a fixed
  motif library and does not perform de novo motif recognition or assembly.
- Sequences are not centered on a biologically meaningful anchor point (e.g. TSS or
  a reference motif), since MEPP's profiles are defined relative to such anchor points.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3175
tools:
- name: MEPP
  role: 'Primary method: generates positional enrichment profiles of motifs relative
    to the GATA1 anchor while accounting for lower-order nucleotide bias'
  repo: https://github.com/npdeloss/mepp
- name: CentriMo
  role: Baseline bin-based central-enrichment comparison tool used to validate that
    SCL/bHLH motifs rank top
- name: TFEA
  role: Baseline quartile-based MEA tool used in the method comparison (does not rank
    SCL/bHLH top)
- name: HOMER
  role: Baseline MEA tool run on top-10% bins under a ZOOPs model for the method comparison
- name: pip
  role: Installs MEPP ('pip install mepp' or 'pip install git+https://github.com/npdeloss/mepp@main')
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:be975f691b35f71677535285a5a88cce8ca0b621ebe3b1cd651042980d19ed9f
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
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# gata1-tal1-motif-spacing-analysis

## Summary

Reproduce the MEPP positional-profile analysis showing the SCL (TAL1) motif preferentially positioned ~12 bp upstream of GATA1 motifs in ±100 bp GATA1-anchored sequences scored by HCT116-vs-K562 ATAC-seq log2 fold change, and benchmark MEPP against CentriMo, TFEA, and HOMER.

## When to use

Use when testing whether cell-type-specific chromatin accessibility associates with the spacing of a co-occurring TF motif (e.g. SCL/bHLH) relative to an anchor motif (e.g. GATA1) in scored centered sequences.

## When NOT to use

- The question is plain motif presence/absence enrichment without positional information — most MEA methods analyze sequences 'without regard to the motif's position within the sequence'; use a standard MEA instead.
- De novo motif discovery is required — MEPP currently functions only with a fixed motif library and does not perform de novo motif recognition or assembly.
- Sequences are not centered on a biologically meaningful anchor point (e.g. TSS or a reference motif), since MEPP's profiles are defined relative to such anchor points.

## Inputs

- Scored FASTA of ±100 bp sequences centered on GATA1 motif instances, headers of the form '>sequence_name sequence_score' (ATAC-seq log2 fold-change scores)
- Motif matrix file in JASPAR format (e.g. from http://jaspar.genereg.net/downloads/, or the bundled data/homer.motifs.txt)
- HCT116 and K562 ATAC-seq data from GEO accession GSE203135 / ENCODE human cell line data

## Outputs

- MEPP positional enrichment profile (heatmap + plot, png/svg) per motif relative to the GATA1 anchor, with confidence intervals from permutation testing
- Clustering table of positional correlations with multiple-testing-adjusted p-values
- Comparison table of motif rankings from MEPP, CentriMo, TFEA, and HOMER (reproducing Table S5)

## How to apply

Score ±100 bp windows centered on GATA1 motifs with a HCT116-vs-K562 ATAC-seq log2 fold-change (GEO GSE203135 / ENCODE cell line data), writing a FASTA whose headers carry '>sequence_name sequence_score'. Install MEPP via pip ('pip install mepp') and run motif enrichment positional profiling on the scored, GATA1-anchored sequences; MEPP outputs a positional enrichment profile per motif while accounting for lower-order nucleotide bias and avoiding arbitrary threshold selection. Inspect the SCL (TAL1) profile for the ~12 bp upstream enrichment peak and the K562-specific negative-scored valley. For validation, run the baseline MEA tools on the same scored bins — CentriMo (bin-based), TFEA (quartile-based), and HOMER (top-10% bins, ZOOPs model) — and compile a comparison table (Table S5) in which MEPP and CentriMo, but not TFEA or HOMER, rank SCL/bHLH motifs at the top.

## Related tools

- **MEPP** (Primary method: generates positional enrichment profiles of motifs relative to the GATA1 anchor while accounting for lower-order nucleotide bias) — https://github.com/npdeloss/mepp
- **CentriMo** (Baseline bin-based central-enrichment comparison tool used to validate that SCL/bHLH motifs rank top)
- **TFEA** (Baseline quartile-based MEA tool used in the method comparison (does not rank SCL/bHLH top))
- **HOMER** (Baseline MEA tool run on top-10% bins under a ZOOPs model for the method comparison)
- **pip** (Installs MEPP ('pip install mepp' or 'pip install git+https://github.com/npdeloss/mepp@main')) — https://github.com/npdeloss/mepp

## Examples

```
pip install git+https://github.com/npdeloss/mepp@main && mepp --fa gata1_scored.fa --motifs jaspar_motifs.txt --out mepp_out
```

## Evaluation signals

- MEPP positional profile shows the SCL (TAL1) motif enrichment peak ~12 bp upstream of the GATA1 anchor
- The profile exhibits a negative-scored valley specific to K562 accessibility
- In the compiled Table S5 comparison, MEPP and CentriMo but not TFEA or HOMER rank SCL/bHLH motifs at the top
- Input FASTA headers parse correctly as '>sequence_name sequence_score' and sequences pass the --dgt degenerate-base filter
- Permutation-based confidence intervals (default --perms 1000) and adjusted positional-correlation p-values (fdr_by, alpha 0.01) are reported in the clustering table

## Limitations

- MEPP currently functions with a fixed motif library; de novo motif recognition/assembly is left to future work.
- No software version is specified for MEPP and no changelog is provided in the repository.
- Permutation testing can lead to significant GPU memory usage; --gjobs must be set carefully to avoid jobs crowding each other out of GPU memory, or GPU use must be disabled with --nogpu.

## Evidence

- [abstract] outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs) within the analyzed sequences while accounting: "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs) within the analyzed sequences while accounting"
- [methods] To install Motif Enrichment Positional Profiling, run this command in your terminal: $ pip install mepp: "To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp"
- [discussion] we find that our method recovers the positional relevance of SCL-to-GATA1 motif spacing to K562 cells: "we find that our method recovers the positional relevance of SCL-to-GATA1 motif spacing to K562 cells, a result supported by the previous characterization of ternary complex formation on a composite"
- [readme] Path to a scored fasta file, where sequence headers are of the form: ">sequence_name sequence_score".: "Path to a scored fasta file, where sequence
                                      headers are of the form: ">sequence_name
                                      sequence_score"."
- [supplementary] All raw data generated for this study can be accessed at NCBI Gene Expression Omnibus (GEO; https://www.ncbi.nlm.nih.gov/geo/) accession number GSE203135: "All raw data generated for this study can be accessed at NCBI Gene Expression Omnibus (GEO; https://www.ncbi.nlm.nih.gov/geo/) accession number GSE203135"
