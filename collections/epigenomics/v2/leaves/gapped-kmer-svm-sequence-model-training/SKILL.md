---
name: gapped-kmer-svm-sequence-model-training
description: Use when you have fine-mapped, reproducible enriched CLIP windows from
  Skipper and need sequence-driven models to predict how SNVs change transcription/translation
  factor binding at occupied sites.
when_to_use_negative:
- The goal is de novo motif discovery or binding affinity estimation directly from
  CLIP data — the article notes the post hoc gkm-SVM approach 'does not learn sequence
  motifs or binding affinity directly from CLIP data'
- State-of-the-art variant-effect accuracy is required and a deep-learning sequence
  model is available — the article notes the gkm-SVM 'is easy to use but less sophisticated
  than emerging tools based on deep learning'
- Sequences come from a peak-caller output rather than Skipper's tiled-window, fine-mapped
  enriched windows — the training design (75-nt windows centered on fine-mapped enriched
  windows vs. length-matched unbound background) presumes Skipper output
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3511
- http://edamontology.org/topic_0102
tools:
- name: LS-GKM (gapped k-mer SVM)
  role: Trains per-RBP gkm-SVM classifiers (gkmtrain) on bound vs. unbound window
    sequences and scores reference/alternate alleles (gkmpredict) to compute variant
    delta scores
  repo: https://github.com/Dongwon-Lee/lsgkm
- name: Skipper
  role: Produces the fine-mapped, reproducible enriched windows whose 75-nt sequences
    form the positive (bound) training set and defines the unbound background
  repo: https://github.com/YeoLab/skipper
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:369b1dfd76cab681110cd5cdfdc5c1ed35799841b31457d0e7cacd0f1ae1b430
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
  - gapped k-mer SVM
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3511
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gapped-kmer-svm-sequence-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains per-RBP gapped k-mer SVM models (LS-GKM) on 75-nt bound window sequences from Skipper eCLIP output versus length-matched unbound background sequences, then scores reference and alternate alleles to compute variant delta scores for selective constraint analysis.

## When to use

Use when you have fine-mapped, reproducible enriched CLIP windows from Skipper and need sequence-driven models to predict how SNVs change transcription/translation factor binding at occupied sites.

## When NOT to use

- The goal is de novo motif discovery or binding affinity estimation directly from CLIP data — the article notes the post hoc gkm-SVM approach 'does not learn sequence motifs or binding affinity directly from CLIP data'
- State-of-the-art variant-effect accuracy is required and a deep-learning sequence model is available — the article notes the gkm-SVM 'is easy to use but less sophisticated than emerging tools based on deep learning'
- Sequences come from a peak-caller output rather than Skipper's tiled-window, fine-mapped enriched windows — the training design (75-nt windows centered on fine-mapped enriched windows vs. length-matched unbound background) presumes Skipper output

## Inputs

- Fine-mapped, reproducible enriched window coordinates from Skipper eCLIP output (per RBP/translation factor)
- GENCODE GRCh38 primary assembly genome FASTA (e.g., GRCh38.primary_assembly.genome.fa) for extracting 75-nt window sequences
- Bound (enriched) and length-matched unbound background sequences in FASTA format (positive/negative sets for gkmtrain)
- gnomAD SNVs overlapping the enriched windows for delta-score prediction

## Outputs

- Per-RBP LS-GKM model files (<outprefix>.model.txt) from gkmtrain
- Variant delta scores (reference- minus alternate-allele gkm-SVM prediction differences) for gnomAD SNVs
- Constraint regressions (mean delta score vs. gnomAD allele-frequency bin per RBP) aggregated to transcript level
- constrained_rbp_transcript_pairs.tsv — nominated constrained RBP-transcript pairs at 10% FDR

## How to apply

Extract 75-nt genomic sequences centered on fine-mapped, reproducible enriched windows from Skipper output for each translation factor eCLIP sample, using the GENCODE GRCh38 primary assembly FASTA as the reference. For each RBP, supply the bound (enriched) window sequences as the positive FASTA and length-matched unbound background sequences as the negative FASTA to LS-GKM's gkmtrain (kernel 2, gkm, by default). Score gnomAD SNVs overlapping the windows with gkmpredict on both reference and alternate alleles, defining each variant's delta score as the reference-minus-alternate model prediction difference. Bin variants by gnomAD allele frequency and regress mean delta score against allele-frequency bin per RBP, then aggregate constraint to the transcript level and correct at 10% FDR to nominate constrained RBP-transcript pairs.

## Related tools

- **LS-GKM (gapped k-mer SVM)** (Trains per-RBP gkm-SVM classifiers (gkmtrain) on bound vs. unbound window sequences and scores reference/alternate alleles (gkmpredict) to compute variant delta scores) — https://github.com/Dongwon-Lee/lsgkm
- **Skipper** (Produces the fine-mapped, reproducible enriched windows whose 75-nt sequences form the positive (bound) training set and defines the unbound background) — https://github.com/YeoLab/skipper

## Examples

```
gkmtrain -t 2 bound_windows.fa unbound_background.fa rbp_model
```

## Evaluation signals

- gkmtrain produces a model file (<outprefix>.model.txt) for every RBP with a non-empty bound FASTA and matched negative FASTA of equal sequence lengths
- Every gnomAD SNV in an enriched window yields two gkmpredict scores (reference and alternate), and delta score = reference prediction minus alternate prediction
- The final constrained_rbp_transcript_pairs.tsv reproduces the 65 constrained RBP-transcript pairs reported in Figure 7C / Table S9 at 10% FDR
- Constraint signal is aggregated per RBP-transcript with multiple-testing correction applied before nomination (transcript-level, not window-level, calls)

## Limitations

- The gapped k-mer SVM is easy to use but less sophisticated than emerging deep learning tools for modeling binding site occupancy
- Post hoc fine-mapping with gkm-SVM does not learn sequence motifs or binding affinity directly from CLIP data
- gkm-SVM's performance depends on the quality of Skipper's enriched window calls; Skipper itself requires a matched input sample, so CLIP datasets without input controls cannot feed this pipeline

## Evidence
