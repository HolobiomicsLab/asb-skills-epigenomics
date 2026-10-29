---
name: csrna-seq-tss-sequence-scoring
description: Use when you have csRNA-seq TSS calls and need score-continuum sequences
  centered on each TSS for positional motif enrichment analysis (MEPP) of core promoter
  elements such as Inr, TATA-box, and DPE.
when_to_use_negative:
- You need de novo motif discovery from the scored sequences — MEPP currently only
  functions with a fixed motif library (e.g. the Ohler or HOMER motif files), so novel
  motifs cannot be assembled with this skill.
- Your sequences are not centered on an anchor point (e.g. TSS or motif) — the method
  requires centered genomic sequences to produce a positional profile relative to
  an anchor.
- You only have a binary (thresholded) high/low activity grouping rather than a continuous
  score — MEPP exploits a score continuum and would lose its main advantage if scores
  are collapsed to two classes.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: MEPP
  role: Consumes the score-continuum TSS-centered sequences and computes position-dependent
    motif enrichment profiles relative to the TSS anchor, controlling for lower-order
    nucleotide bias
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installation channel for the mepp package used to score and profile the TSS-centered
    sequences
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:71e7c2c23341c765109979d6bdc9d3b8e96cd8bbec7d12d99d57cfe73497d02e
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
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# csRNA-seq-tss-sequence-scoring

## Summary

Preparation of ±200 bp DNA sequences centered on high-confidence csRNA-seq transcription start sites (44,877 Drosophila TSSs), each scored with log-transformed csRNA-seq coverage, so that MEPP can compute position-dependent motif enrichment profiles relative to the TSS anchor point.

## When to use

Use when you have csRNA-seq TSS calls and need score-continuum sequences centered on each TSS for positional motif enrichment analysis (MEPP) of core promoter elements such as Inr, TATA-box, and DPE.

## When NOT to use

- You need de novo motif discovery from the scored sequences — MEPP currently only functions with a fixed motif library (e.g. the Ohler or HOMER motif files), so novel motifs cannot be assembled with this skill.
- Your sequences are not centered on an anchor point (e.g. TSS or motif) — the method requires centered genomic sequences to produce a positional profile relative to an anchor.
- You only have a binary (thresholded) high/low activity grouping rather than a continuous score — MEPP exploits a score continuum and would lose its main advantage if scores are collapsed to two classes.

## Inputs

- High-confidence csRNA-seq TSS coordinates (44,877 Drosophila high-confidence TSSs) with per-TSS csRNA-seq coverage values
- Reference genome assembly for extracting ±200 bp sequences around each TSS
- Ohler Drosophila core promoter motif library in JASPAR format (data/ohler_motifs.txt from the mepp repository)
- Scored FASTA convention: sequence headers of the form '>sequence_name sequence_score' with log-transformed csRNA-seq coverage as the score

## Outputs

- Scored FASTA of 401 bp sequences centered on each csRNA-seq TSS, headers carrying log-transformed coverage scores
- MEPP positional enrichment profiles per motif (Inr, TATA-box, DPE) relative to the TSS anchor point
- Extracted peak offsets for each core promoter motif relative to the TSS center

## How to apply

Extract ±200 bp genomic sequences centered on each of the 44,877 high-confidence Drosophila csRNA-seq TSSs, and encode each sequence's biological activity as the log-transformed csRNA-seq coverage at that TSS. Format the data as a scored FASTA where headers carry the score ('>sequence_name sequence_score'), since MEPP requires sequences scored on a continuum rather than binary thresholded sets — this avoids arbitrary threshold selection and lets the positional profile correlate motif position with a quantitative activity measure. Center plots on the TSS by setting the 0-based --center offset to half the sequence length (the default for 401 bp centered sequences) or explicitly to the middle base. Supply the fixed Ohler Drosophila core promoter motif library (data/ohler_motifs.txt, JASPAR format) so MEPP computes position-dependent enrichment for Inr, TATA-box, and DPE while controlling for lower-order nucleotide bias. Validate the result by checking that extracted peak offsets recapitulate known promoter organization: Inr at the TSS center (0 bp), TATA-box at −28 bp, and DPE at +25 bp.

## Related tools

- **MEPP** (Consumes the score-continuum TSS-centered sequences and computes position-dependent motif enrichment profiles relative to the TSS anchor, controlling for lower-order nucleotide bias) — https://github.com/npdeloss/mepp
- **pip** (Installation channel for the mepp package used to score and profile the TSS-centered sequences) — https://github.com/npdeloss/mepp

## Examples

```
mepp --fa tss_centered_scored.fa --motifs data/ohler_motifs.txt --out mepp_tss_profiles
```

## Evaluation signals

- Extracted TATA-box enrichment peak lands upstream of the TSS at approximately −28 bp
- Extracted DPE enrichment peak lands downstream of the TSS at approximately +25 bp
- Inr motif profile is maximally correlated at the TSS center (offset 0 bp)
- All sequences are exactly 401 bp (±200 bp around the TSS) and headers carry a numeric log-transformed coverage score parseable by MEPP's '>sequence_name sequence_score' format
- Sequence degeneracy passes MEPP's --dgt filter (default: 100% allowed, lower if filtering repeats)

## Limitations

- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work, so only known core promoter motifs (e.g. the Ohler Drosophila set) can be profiled.
- No software version is specified for MEPP, and no changelog exists for the repository, so reproducibility of the exact profiling behavior may vary across repository states.
- MEPP requires a GPU-capable Tensorflow setup for efficiency; permutation testing (--perms, default 1000) can lead to significant GPU memory usage, and crowded GPU jobs can cause profile generation to fail.
- Raw csRNA-seq data underlying the TSS calls are available at NCBI GEO accession GSE203135; obtaining equivalent high-confidence TSS sets for other organisms requires similar coverage-based scoring.

## Evidence

- [full_text] Load the ±200 bp sequences centered on the 44,877 high-confidence Drosophila csRNA-seq TSS, scored by log-transformed csRNA-seq coverage, and the fixed Ohler Drosophila promoter motif library shipped with the package.: [evidence span withheld — anchor status: not_found]
- [intro] MEPP exploits score-continuum design without arbitrary thresholds while controlling sequence bias.: "The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias."
- [discussion] The findings confirm TATA-box peaks at −28 bp and DPE at +25 bp relative to the TSS.: "These results recapitulate known relationships such as the positioning of core promoter elements surrounding drosophila melanogaster TSS"
- [readme] Path to a scored fasta file, where sequence headers are of the form: ">sequence_name sequence_score". [required]: "Path to a scored fasta file, where sequence headers are of the form: ">sequence_name
                                      sequence_score"."
- [readme] The file data/ohler_motifs.txt contains Drosophila core promoter motifs from Ohler et al.: "The file data/ohler_motifs.txt contains Drosophila core promoter motifs from `Ohler et al. <https://pubmed.ncbi.nlm.nih.gov/12537576/>`_."
- [abstract] MEPP outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites).: "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs)"
