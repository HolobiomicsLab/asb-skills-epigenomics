---
name: variant-delta-score-scoring
description: Use when you need per-variant sequence-effect scores for SNVs in RBP
  binding windows — e.g. to test whether disruptive variants (low gnomAD AF) show
  larger predicted binding changes than common variants.
when_to_use_negative:
- Your goal is de novo motif discovery or binding-site calling from raw CLIP reads
  — use the Skipper window/enrichment pipeline instead; the post hoc fine-mapping
  does not learn sequence motifs or binding affinity directly from CLIP data
- Variants of interest are not SNVs overlapping scored binding windows (e.g. indels
  or SVs without modeled allele sequences), since delta scores are defined only for
  reference/alternate sequence pairs the gkm-SVM can score
- You need state-of-the-art variant-effect estimates for functional follow-up — the
  paper notes the gapped k-mer SVM 'is easy to use but less sophisticated than emerging
  tools based on deep learning'
edam_operation: http://edamontology.org/operation_0262
tools:
- name: LS-GKM (gapped k-mer SVM)
  role: Trains per-RBP sequence models on bound vs. unbound window sequences and scores
    reference/alternate allele 75-mers to produce delta scores
  repo: https://github.com/Dongwon-Lee/lsgkm
- name: Skipper
  role: Supplies the fine-mapped, reproducible enriched binding windows whose centered
    75-nt sequences are scored
  repo: https://github.com/YeoLab/skipper
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:1a7e69d53b4a35089e62a526019011afd424778f6d89c23a3c9e335e0720802b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/YeoLab/skipper
  repo_url: https://github.com/Dongwon-Lee/lsgkm
  tools:
  - LS-GKM (gapped k-mer SVM)
  - Skipper
  edam_operation: http://edamontology.org/operation_0262
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# variant-delta-score-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores gnomAD SNVs overlapping fine-mapped Skipper binding windows with a trained LS-GKM gapped k-mer SVM, computing a per-variant delta score (reference minus alternate allele prediction) that is then binned by allele frequency and regressed to test for selective constraint on translation factor occupied sites.

## When to use

Use when you need per-variant sequence-effect scores for SNVs in RBP binding windows — e.g. to test whether disruptive variants (low gnomAD AF) show larger predicted binding changes than common variants.

## When NOT to use

- Your goal is de novo motif discovery or binding-site calling from raw CLIP reads — use the Skipper window/enrichment pipeline instead; the post hoc fine-mapping does not learn sequence motifs or binding affinity directly from CLIP data
- Variants of interest are not SNVs overlapping scored binding windows (e.g. indels or SVs without modeled allele sequences), since delta scores are defined only for reference/alternate sequence pairs the gkm-SVM can score
- You need state-of-the-art variant-effect estimates for functional follow-up — the paper notes the gapped k-mer SVM 'is easy to use but less sophisticated than emerging tools based on deep learning'

## Inputs

- Fine-mapped, reproducible enriched windows from Skipper output (BED-like, per translation factor eCLIP sample)
- GENCODE GRCh38 primary assembly FASTA (GRCh38.primary_assembly.genome.fa)
- gnomAD SNVs overlapping the binding windows, with allele frequency annotations
- Trained LS-GKM model files (.model.txt / .model.txt.gz) per RBP
- Length-matched unbound background sequences and bound window sequences (FASTA) used for gkmtrain

## Outputs

- Per-variant delta scores (difference between reference- and alternate-allele LS-GKM predictions)
- Mean delta score per gnomAD allele-frequency bin per RBP
- Transcript-level aggregated constraint statistics with 10% FDR correction
- constrained_rbp_transcript_pairs.tsv table of nominated constrained RBP-transcript pairs

## How to apply

Extract 75-nt genomic sequences centered on fine-mapped, reproducible enriched windows from Skipper output, using the GENCODE GRCh38 primary assembly FASTA as reference, and train an LS-GKM gapped k-mer SVM per RBP on bound versus length-matched unbound background sequences. For each gnomAD SNV overlapping a window, build reference and alternate allele versions of the 75-mer, score both with the trained LS-GKM model (gkmpredict), and define the delta score as the difference between reference- and alternate-allele model predictions. Bin variants by gnomAD allele frequency within each RBP, compute the mean delta score per bin, and regress mean delta score against allele-frequency bin to quantify constraint. Aggregate the constraint signal to the transcript level per RBP and apply multiple-testing correction at a 10% FDR threshold to nominate constrained RBP-transcript pairs.

## Related tools

- **LS-GKM (gapped k-mer SVM)** (Trains per-RBP sequence models on bound vs. unbound window sequences and scores reference/alternate allele 75-mers to produce delta scores) — https://github.com/Dongwon-Lee/lsgkm
- **Skipper** (Supplies the fine-mapped, reproducible enriched binding windows whose centered 75-nt sequences are scored) — https://github.com/YeoLab/skipper

## Examples

```
gkmtrain bound_windows.fa unbound_background.fa RBP_model; gkmpredict gnomAD_snvs_ref_and_alt.fa RBP_model.model.txt delta_score_predictions.txt
```

## Evaluation signals

- The nominated set of constrained RBP-transcript pairs at 10% FDR reproduces the 65 constrained pairs reported in Figure 7C / Table S9 of the paper
- Delta scores equal zero exactly when reference and alternate allele predictions are identical; sign reflects predicted binding decrease/increase
- Mean delta score per allele-frequency bin should decrease (or constraint regression slope should be significant) for genuinely constrained RBPs, consistent with depletion of disruptive variants in occupied sites
- Every scored SNV must lie within a 75-nt window centered on a fine-mapped, reproducible Skipper enriched window on the correct strand/genome build (GRCh38 primary assembly)
- Variant bins must be populated: check each gnomAD allele-frequency bin per RBP contains sufficient variants before regression

## Limitations

- The gapped k-mer SVM used to model translation factor binding site occupancy is easy to use but less sophisticated than emerging tools based on deep learning
- Post hoc fine-mapping does not learn sequence motifs or binding affinity directly from CLIP data; eCLIP data alone cannot determine the mechanism underlying observed binding changes
- Transcript-level aggregation is 'a well-powered intermediate approach to generate hypotheses for functional transcript binding' rather than a definitive per-site test

## Evidence

- [methods] delta scoring rationale: [evidence span withheld — anchor status: not_found]
- [methods] constraint regression rationale: [evidence span withheld — anchor status: not_found]
- [readme] gkm-SVM executables: [evidence span withheld — anchor status: not_found]
