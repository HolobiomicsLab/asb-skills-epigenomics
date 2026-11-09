---
name: allele-frequency-binned-constraint-regression
description: Use when gnomAD SNVs overlap fine-mapped eCLIP enriched windows and you
  need to test whether disruptive variants are depleted from factor-bound sequences
  as evidence of constraint.
when_to_use_negative:
- No matched input/control sample exists for the CLIP dataset, since Skipper requires
  one to build its beta-binomial model before windows can be fine-mapped
- The goal is to learn binding motifs or affinity directly from CLIP data — post hoc
  fine-mapping plus the gkm-SVM model occupancy but does not learn motifs from CLIP
  signal itself
- Variants are indels or structural variants, since the workflow scores gnomAD SNVs
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: LS-GKM (gapped k-mer SVM)
  role: Trains per-RBP sequence models on bound vs. unbound windows and scores SNV
    delta predictions used as the constraint regression response
  repo: https://github.com/Dongwon-Lee/lsgkm
- name: Skipper
  role: Produces the fine-mapped, reproducible enriched windows whose sequences train
    the gkm-SVM and onto which gnomAD variants are intersected
  repo: https://github.com/YeoLab/skipper
- name: gnomAD
  role: Source of human SNVs and allele frequencies binned for the constraint regression
- name: GENCODE GRCh38 primary assembly
  role: Reference FASTA for extracting 75-nt window-centered sequences
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:66454849b6e85684b2ed5dc4dab5a1f4e56892fb84fd22bb5cb0e17170f47d77
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
  - GENCODE GRCh38 primary assembly
  - LS-GKM (gapped k-mer SVM)
  - Skipper
  - gnomAD
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# allele-frequency-binned-constraint-regression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies selective constraint on translation factor occupied binding sites by regressing mean LS-GKM delta scores (variant-predicted binding changes) against gnomAD allele-frequency bins per RBP, then aggregating to RBP-transcript pairs with multiple-testing correction at 10% FDR.

## When to use

Use when gnomAD SNVs overlap fine-mapped eCLIP enriched windows and you need to test whether disruptive variants are depleted from factor-bound sequences as evidence of constraint.

## When NOT to use

- No matched input/control sample exists for the CLIP dataset, since Skipper requires one to build its beta-binomial model before windows can be fine-mapped
- The goal is to learn binding motifs or affinity directly from CLIP data — post hoc fine-mapping plus the gkm-SVM model occupancy but does not learn motifs from CLIP signal itself
- Variants are indels or structural variants, since the workflow scores gnomAD SNVs

## Inputs

- Fine-mapped, reproducible enriched window coordinates from Skipper eCLIP output
- GENCODE GRCh38 primary assembly FASTA (e.g., GRCh38.primary_assembly.genome.fa)
- gnomAD SNVs overlapping the enriched windows, with allele frequencies
- Positive (bound window) and negative (length-matched unbound) FASTA sequence sets per RBP

## Outputs

- Per-variant delta scores (reference- minus alternate-allele LS-GKM predictions)
- Per-RBP regression of mean delta score against gnomAD allele-frequency bin
- constrained_rbp_transcript_pairs.tsv — transcript-level constrained pairs at 10% FDR (expected to reproduce the 65 pairs from Figure 7C / Table S9)

## How to apply

Extract 75-nt genomic sequences centered on fine-mapped, reproducible enriched windows from Skipper output for each translation factor eCLIP sample, using the GENCODE GRCh38 primary assembly FASTA as reference. Train an LS-GKM gapped k-mer SVM per RBP on bound window sequences versus length-matched unbound background sequences using gkmtrain, then score every gnomAD SNV overlapping the windows with gkmpredict, computing each variant's delta score as the reference-allele prediction minus the alternate-allele prediction. Bin variants by gnomAD allele frequency within each RBP and regress mean delta score against allele-frequency bin; a negative relationship (disruptive deltas enriched at low-frequency bins) signals constraint. Aggregate the constraint signal to the transcript level per RBP and apply multiple-testing correction at a 10% FDR threshold to nominate constrained RBP-transcript pairs.

## Related tools

- **LS-GKM (gapped k-mer SVM)** (Trains per-RBP sequence models on bound vs. unbound windows and scores SNV delta predictions used as the constraint regression response) — https://github.com/Dongwon-Lee/lsgkm
- **Skipper** (Produces the fine-mapped, reproducible enriched windows whose sequences train the gkm-SVM and onto which gnomAD variants are intersected) — https://github.com/YeoLab/skipper
- **gnomAD** (Source of human SNVs and allele frequencies binned for the constraint regression)
- **GENCODE GRCh38 primary assembly** (Reference FASTA for extracting 75-nt window-centered sequences)

## Examples

```
gkmtrain <posfile> <negfile> <outprefix>  # e.g., gkmtrain bound_75nt.fa unbound_75nt.fa RBP1_gkmsvm, then gkmpredict for reference and alternate alleles
```

## Evaluation signals

- The final constrained pairs table (constrained_rbp_transcript_pairs.tsv) reproduces the 65 constrained RBP-transcript pairs reported at 10% FDR in Figure 7C / Table S9
- Each RBP's regression shows mean delta score against gnomAD allele-frequency bin consistent with depletion of disruptive variation in occupied sites
- Delta scores are computable for every gnomAD SNV overlapping the fine-mapped windows using the trained LS-GKM model for that RBP
- Training sequences are 75 nt, centered on Skipper fine-mapped windows, with negatives length-matched and unbound

## Limitations

- The gapped k-mer SVM used to model translation factor binding site occupancy is less sophisticated than emerging deep learning tools
- Aggregating constraint at the transcript level is an intermediate, hypothesis-generating approach rather than a definitive measure of functional binding
- Downstream follow-up (e.g., RPS19 variant ORFs in HEK293T cells) may not reflect the true disease state

## Evidence

- [abstract] Depletion of genetic variation is observed in occupied sites, and transcripts subject to selective constraint because of translation factor occupancy are nominated: "we observe depletion of genetic variation in occupied sites and nominate transcripts subject to selective constraint because of translation factor occupancy"
- [discussion] Transcript-level aggregation is a well-powered constraint approach: "Our results show that aggregating constraint at the level of transcripts is a well-powered intermediate approach to generate hypotheses for functional transcript binding"
- [discussion] gkm-SVM is easy to use but simpler than deep learning: "The gapped k-mer SVM we use to model translation factor binding site occupancy is easy to use but less sophisticated than emerging tools based on deep learning."
- [readme] gkmtrain trains the SVM from positive and negative FASTA files: [evidence span withheld — anchor status: external_doc]
