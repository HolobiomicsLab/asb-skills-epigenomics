---
name: genomic-domain-boundary-refinement
description: Use when calling broad ChIP-seq enrichment domains (e.g., H3K27me3/H3K9me3,
  kb-to-Mb) and you must convert multi-scale coarse-grained '1' blocks into precisely
  bounded candidate domains prior to Poisson/FDR significance assessment.
when_to_use_negative:
- Sharp, punctuate transcription-factor ChIP-seq peaks are the target — RECOGNICER
  is designed for broad domains (kb–Mb) like H3K27me3/H3K9me3, not local point-source
  peak calling
- Two biologically distinct domains separated by fine-scale gaps must be kept strictly
  separate — coarse-graining can falsely merge them into one coarse-scale domain
- Parallel RECOGNICER runs from the same directory are needed — hard-coded temporary
  file names make concurrent instances from one directory interfere and produce absurd
  outcomes
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: RECOGNICER
  role: Implements the coarse-graining, domain retrieval/boundary refinement, and
    Poisson/FDR significance determination pipeline invoked via RECOGNICER.sh
  repo: https://github.com/zanglab/recognicer
- name: SICER
  role: Predecessor broad-domain caller for histone modification ChIP-seq, used as
    a performance comparison baseline with default parameters
- name: RSEG
  role: HMM-based broad epigenomic domain caller used as a comparison baseline
- name: MUSIC
  role: Multiscale decomposition peak caller used as a comparison baseline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:38c568d280b1c789f4ee1de119ab41c4a0178ef4c9220e1c79336460f3182e67
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/zanglab/recognicer
  tools:
  - MUSIC
  - RECOGNICER
  - RSEG
  - SICER
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# genomic-domain-boundary-refinement

## Summary

RECOGNICER's domain retrieval and boundary determination step: after recursive L=3 block-transform coarse-graining, fine-scale '1' blocks are clustered under coarse-scale '1' blocks and candidate broad-domain boundaries are shrunk/expanded, merged, and gap-closed before significance testing.

## When to use

Use when calling broad ChIP-seq enrichment domains (e.g., H3K27me3/H3K9me3, kb-to-Mb) and you must convert multi-scale coarse-grained '1' blocks into precisely bounded candidate domains prior to Poisson/FDR significance assessment.

## When NOT to use

- Sharp, punctuate transcription-factor ChIP-seq peaks are the target — RECOGNICER is designed for broad domains (kb–Mb) like H3K27me3/H3K9me3, not local point-source peak calling
- Two biologically distinct domains separated by fine-scale gaps must be kept strictly separate — coarse-graining can falsely merge them into one coarse-scale domain
- Parallel RECOGNICER runs from the same directory are needed — hard-coded temporary file names make concurrent instances from one directory interfere and produce absurd outcomes

## Inputs

- ChIP-seq mapped reads BED file (ChIP sample)
- Control sample mapped reads BED file
- Genome chromosome lengths for the chosen build (e.g., hg18/hg19/hg38)
- Configuration parameters: WINDOW_SIZE (default 200), FRAGMENT_SIZE (default 150), GENOME_FRACTION (default 0.74), CHIPTHRESHOLD (default 1)

## Outputs

- $SAMPLE-fdr0.01_broadPeak.bed — significant domains in ENCODE broadPeak format (5th column = fold change, 7th column = ChIP read count)
- $SAMPLE.cgsummary — candidate domains with chrom, start, end, ChIP read count, control read count, P-value, fold change, Q-value
- $SAMPLE-W200.bedgraph — 200 bp resolution read pile-up track for genome browser visualization
- $SAMPLE-nonredundant.bed — non-redundant filtered ChIP reads

## How to apply

First partition chromosomes into 200 bp non-overlapping windows, shift single-end read 5' coordinates by half the fragment size (75 bp for the default 150 bp mono-nucleosomal fragment), and binarize windows whose read counts exceed the random-placement average as '1' blocks. Then apply recursive block transformations with block transformation factor L=3 under the majority rule, running all L=3 phases each round and keeping the phase that yields the largest number of '1' blocks; compute auto-correlation values C(1) and C(2) at the coarsest and previous scales and fit the characteristic length lambda with C(r) ~ C0 * e^(-r/lambda), terminating when lambda exceeds 1 at both present and previous scales. Trace back from coarsest to finest scale: cluster fine-scale '1' blocks under coarse-scale '1' blocks to form candidate domains, shrink or expand each boundary by one fine-scale block depending on neighboring block values, merge overlapping domains, and merge adjacent domains separated by a gap of at most one fine-scale block. Finally, compare ChIP vs control read counts per candidate domain, compute a fold-enrichment score and p-value under a local Poisson model, and apply Benjamini-Hochberg FDR control (e.g., 0.01).

## Related tools

- **RECOGNICER** (Implements the coarse-graining, domain retrieval/boundary refinement, and Poisson/FDR significance determination pipeline invoked via RECOGNICER.sh) — https://github.com/zanglab/recognicer
- **SICER** (Predecessor broad-domain caller for histone modification ChIP-seq, used as a performance comparison baseline with default parameters)
- **RSEG** (HMM-based broad epigenomic domain caller used as a comparison baseline)
- **MUSIC** (Multiscale decomposition peak caller used as a comparison baseline)

## Examples

```
sh RECOGNICER.sh ChIP.bed control.bed 0.01
```

## Evaluation signals

- Output file $SAMPLE-fdr0.01_broadPeak.bed exists and parses in ENCODE broadPeak format
- All reported domains satisfy the FDR (Q-value) <= 0.01 threshold
- broadPeak 5th-column score carries the ChIP/control fold change and 7th-column signalValue carries the ChIP read count
- Identified H3K27me3 domain sizes span kb to Mb (as observed in human CD4+ T cells, Fig. 3C)
- Auto-correlation length behaves as expected during coarse-graining (~100 kb for synthetic data; growing for real ChIP-seq data, Fig. 3A–3B)

## Limitations

- RECOGNICER may falsely merge two domains separated by gaps on the fine scale into one domain on the coarse scale; users should specifically monitor such cases
- Domain merging cases may require incorporating other information such as specific featured regions or masked regions
- Implemented in python2 with a scipy prerequisite; parallel runs from the same directory interfere via hard-coded temporary file names
- The whole run may take several hours

## Evidence

- [other] Trace back from the coarsest to the finest scale to retrieve candidate domains: cluster fine-scale '1' blocks under coarse-scale '1' blocks, shrink/expand domain boundaries by one fine-scale block depending on neighboring block values: [evidence span withheld — anchor status: not_found]
- [other] fit the characteristic length lambda with a single exponential C(r) ~ C0 * e^(-r/lambda), and terminate coarse-graining when lambda at both the present and previous scales exceeds 1: [evidence span withheld — anchor status: not_found]
- [other] compute a fold-enrichment score and a p-value under a local Poisson model, and apply the Benjamini-Hochberg procedure to control the FDR at 0.01: [evidence span withheld — anchor status: not_found]
