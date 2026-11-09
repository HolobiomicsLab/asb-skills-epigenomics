---
name: drosophila-core-promoter-motif-analysis
description: Use when you have scored DNA sequences centered on an anchor point (e.g.
  csRNA-seq TSSs) and need to determine whether motif presence correlates with biological
  activity at specific positions relative to that anchor, such as core promoter element
  positioning.
when_to_use_negative:
- MEPP currently only functions with a fixed motif library, so this skill is the wrong
  choice if the goal is de novo motif discovery/assembly (left to future work per
  the discussion)
- The input is a single unscored sequence or sequences lacking a meaningful continuous
  biological score — MEPP's positional profiles require sequences scored across a
  continuum reflecting two extremes of biological interest
- Sequences are dominated by degenerate/repeat bases beyond the accepted threshold
  (--dgt, default 100) or contain no anchor point to center the positional profile
  on
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3168
tools:
- name: MEPP
  role: 'Core method: computes a position-dependent enrichment profile for each motif
    relative to the TSS anchor point across scored sequences, controlling for lower-order
    nucleotide bias'
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Package installation channel used to install MEPP into the environment ($
    pip install mepp)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:9a09f1865e823d9f331ac45017c1e5ba7702c145217599d2f391b6f3061f27eb
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# drosophila-core-promoter-motif-analysis

## Summary

Applies MEPP (Motif Enrichment Positional Profiling) to ±200 bp sequences centered on high-confidence Drosophila csRNA-seq TSSs scored by log-transformed coverage, using the fixed Ohler Drosophila core promoter motif library to compute position-dependent enrichment profiles for motifs such as Inr, TATA-box, and DPE relative to the TSS anchor point.

## When to use

Use when you have scored DNA sequences centered on an anchor point (e.g. csRNA-seq TSSs) and need to determine whether motif presence correlates with biological activity at specific positions relative to that anchor, such as core promoter element positioning.

## When NOT to use

- MEPP currently only functions with a fixed motif library, so this skill is the wrong choice if the goal is de novo motif discovery/assembly (left to future work per the discussion)
- The input is a single unscored sequence or sequences lacking a meaningful continuous biological score — MEPP's positional profiles require sequences scored across a continuum reflecting two extremes of biological interest
- Sequences are dominated by degenerate/repeat bases beyond the accepted threshold (--dgt, default 100) or contain no anchor point to center the positional profile on

## Inputs

- Scored FASTA file with ±200 bp sequences centered on the 44,877 high-confidence Drosophila csRNA-seq TSSs, headers of the form '>sequence_name sequence_score' with log-transformed csRNA-seq coverage as the score
- Motif matrices file in JASPAR format (e.g. the fixed Ohler Drosophila core promoter motif library, data/ohler_motifs.txt in the MEPP repository)

## Outputs

- Position-dependent motif enrichment profiles (MEPP plots) for each motif, centered on the TSS anchor point (png/svg per --formats)
- Peak offsets for Inr, TATA-box, and DPE relative to the TSS center (e.g. TATA-box at −28 bp, DPE at +25 bp, Inr at 0 bp)
- Clustering table of positional correlation p-values with multiple-testing adjustment (default fdr_by, alpha 0.01)

## How to apply

Install MEPP with pip ($ pip install mepp) and load ±200 bp sequences centered on high-confidence Drosophila csRNA-seq TSSs, where sequence scores are log-transformed csRNA-seq coverage. Supply the fixed Ohler Drosophila core promoter motif library shipped with the package (data/ohler_motifs.txt, JASPAR format) rather than a de novo motif set. Run MEPP over the scored sequences to compute a position-dependent enrichment profile per motif relative to the TSS anchor point, exploiting MEPP's score-continuum design (no arbitrary thresholding) and its control for lower-order nucleotide bias. Extract each motif's peak position from its enrichment profile and record the Inr, TATA-box, and DPE peak offsets relative to the TSS center. The score-based enrichment avoids arbitrary threshold selection while accounting for sequence composition, and a 0-based --center offset can be given to center plots on the TSS instead of the default midpoint of the sequence.

## Related tools

- **MEPP** (Core method: computes a position-dependent enrichment profile for each motif relative to the TSS anchor point across scored sequences, controlling for lower-order nucleotide bias) — https://github.com/npdeloss/mepp
- **pip** (Package installation channel used to install MEPP into the environment ($ pip install mepp))

## Examples

```
pip install mepp; mepp --fa drosophila_csRNA_TSS.scored.fa --motifs data/ohler_motifs.txt --out mepp_tss_profiles
```

## Evaluation signals

- Inr positional profile peaks maximally correlated at the TSS center (offset 0 bp)
- TATA-box positional profile peaks upstream of the TSS at −28 bp
- DPE positional profile peaks 25 bp downstream of the TSS (+25 bp), matching expected Drosophila promoter organization
- MEPP output directory contains generated MEPP plot files in the requested formats (default png,svg) with the profile centered on the TSS anchor
- Positional correlation p-values in the clustering table are adjusted via the default thorough multiple testing (fdr_by, alpha 0.01)

## Limitations

- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work
- No software version specified for MEPP and no changelog found for the repository
- Permutation testing (--perms, default 1000) can lead to significant GPU memory usage; GPU jobs must be set carefully to avoid crowding jobs out of GPU memory, causing profile generation to fail

## Evidence

- [full_text] MEPP's positional profiles recover known core promoter element positions: "These results recapitulate known relationships such as the positioning of core promoter elements surrounding drosophila melanogaster TSS"
- [abstract] MEPP outputs positional enrichment profiles relative to anchor points while accounting for sequence bias: "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs) within the analyzed sequences while accounting"
- [intro] Score-continuum design avoids arbitrary thresholds: "The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias."
- [methods] Installation via pip: "To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp"
- [readme] Ohler Drosophila motif library shipped with the package: "The file data/ohler_motifs.txt contains Drosophila core promoter motifs from `Ohler et al. <https://pubmed.ncbi.nlm.nih.gov/12537576/>`_."
- [readme] Input FASTA format requirement: [evidence span withheld — anchor status: external_doc]
- [discussion] MEPP limited to fixed motif library: "MEPP currently functions with a fixed motif library. However, the underlying convolutional network architecture lends itself easily to future work for recognizing and assembling de novo motifs based"
