---
name: luxr-hns-overlap-analysis
description: Use when quantifying LuxR/H-NS binding overlap — genome-wide from ChIP-seq
  peaks (506 LuxR, 302 H-NS MACS2 peaks) or promoter-level via FIMO motif scanning
  (e.g. luxCDABE promoter, H-NS motif TCGATAAATT).
when_to_use_negative:
- Input is RNA-seq expression data alone (no ChIP peaks or promoter sequence) — use
  differential expression workflows instead, since overlap requires binding-site evidence
- Scanning for transcription factors other than H-NS/LuxR, or with motifs other than
  the TCGATAAATT H-NS nucleation motif — a different motif file and validation counts
  would be required
- Promoters from organisms without established LuxR binding site annotations (the
  seven LuxR sites used for overlap comparison are specific to the V. harveyi luxCDABE
  promoter)
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0080
tools:
- name: FIMO
  role: Scans the luxCDABE promoter FASTA with the MEME-format H-NS nucleation motif
    TCGATAAATT under default significance scoring to enumerate putative H-NS binding
    sites
- name: findOverlaps.pl
  role: Identifies sets of overlapping LuxR/H-NS ChIP-seq peaks for Venn diagram comparison
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: getPeakAnno.pl
  role: Adds annotation information to MACS2 peaks so overlaps can be assigned to
    promoters/ORFs
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: findClosest2.pl
  role: Produces a table relating luxR peaks to H-NS double peaks
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: reformatFasta.pl
  role: Cleans and reformats multi-FASTA files (e.g. promoter sequences) before motif
    scanning
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: JBrowse
  role: Hosts the uploaded V. harveyi sequences from which promoter sequences are
    retrieved
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:bd3cd1b205eb8891b580ec6b3b2017d79768afd0350722ed6ed31680370200cb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Juliacvk/LuxR_H-NS
  repo_url: https://github.com/Juliacvk/LuxR_H-NS
  tools:
  - FIMO
  - JBrowse
  - findClosest2.pl
  - findOverlaps.pl
  - getPeakAnno.pl
  - reformatFasta.pl
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# luxr-hns-overlap-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Method for quantifying where LuxR and H-NS binding coincide in V. harveyi, either by FIMO motif scanning of promoter sequences (e.g. the luxCDABE promoter with the high-affinity H-NS nucleation motif TCGATAAATT) or by intersecting ChIP-seq peak sets with the deposited Perl overlap scripts. Overlap identifies promoters where LuxR counter-silences H-NS-repressed quorum sensing loci.

## When to use

Use when quantifying LuxR/H-NS binding overlap — genome-wide from ChIP-seq peaks (506 LuxR, 302 H-NS MACS2 peaks) or promoter-level via FIMO motif scanning (e.g. luxCDABE promoter, H-NS motif TCGATAAATT).

## When NOT to use

- Input is RNA-seq expression data alone (no ChIP peaks or promoter sequence) — use differential expression workflows instead, since overlap requires binding-site evidence
- Scanning for transcription factors other than H-NS/LuxR, or with motifs other than the TCGATAAATT H-NS nucleation motif — a different motif file and validation counts would be required
- Promoters from organisms without established LuxR binding site annotations (the seven LuxR sites used for overlap comparison are specific to the V. harveyi luxCDABE promoter)

## Inputs

- luxCDABE promoter nucleotide sequence (FASTA) from deposited V. harveyi resources (JBrowse session or github.com/Juliacvk/LuxR_H-NS)
- MEME-format motif file encoding the H-NS nucleation motif TCGATAAATT
- MACS2 ChIP-seq peak files for LuxR (506 peaks) and H-NS (302 peaks) from GSE136050/GSE136191
- V. harveyi ORF/gene annotation for peak-to-promoter assignment

## Outputs

- FIMO hit table of putative H-NS binding site coordinates in the luxCDABE promoter (expected: 13 sites)
- Overlap annotation showing which H-NS sites coincide with the seven established LuxR binding sites
- Genome-wide overlapping LuxR/H-NS peak sets (Venn diagram input) with 88 peaks proximal to an ORF (≤500 bp)
- Peak-to-gene annotation tables (28 co-occupied promoters driving 63 genes)

## How to apply

For promoter-level scanning, retrieve the luxCDABE promoter nucleotide sequence from the deposited V. harveyi resources (JBrowse session linked in the supplement or the github.com/Juliacvk/LuxR_H-NS sequence data), optionally cleaning the multi-FASTA with reformatFasta.pl. Build a MEME-format motif file encoding the high-affinity E. coli H-NS nucleation motif TCGATAAATT and run FIMO against the promoter FASTA with default significance scoring; the validated result is 13 putative H-NS binding sites, many coinciding with the seven established LuxR binding sites. For genome-wide overlap, take the MACS2-called ChIP-seq peaks (506 LuxR, 302 H-NS) and run findOverlaps.pl to define overlapping peak sets for Venn comparison, getPeakAnno.pl/getGenePeakAnno2.pl to attach annotation, and findClosest2.pl to relate luxR peaks to H-NS double peaks. The rationale is that LuxR derepresses QS loci by competing with H-NS, so overlapping sites mark co-occupied promoters where LuxR evicts H-NS as autoinducer concentration rises. Validation thresholds from the paper: 88 overlapping peaks proximal to an ORF (≤500 bp), representing 28 co-occupied promoters driving 63 genes.

## Related tools

- **FIMO** (Scans the luxCDABE promoter FASTA with the MEME-format H-NS nucleation motif TCGATAAATT under default significance scoring to enumerate putative H-NS binding sites)
- **findOverlaps.pl** (Identifies sets of overlapping LuxR/H-NS ChIP-seq peaks for Venn diagram comparison) — https://github.com/Juliacvk/LuxR_H-NS
- **getPeakAnno.pl** (Adds annotation information to MACS2 peaks so overlaps can be assigned to promoters/ORFs) — https://github.com/Juliacvk/LuxR_H-NS
- **findClosest2.pl** (Produces a table relating luxR peaks to H-NS double peaks) — https://github.com/Juliacvk/LuxR_H-NS
- **reformatFasta.pl** (Cleans and reformats multi-FASTA files (e.g. promoter sequences) before motif scanning) — https://github.com/Juliacvk/LuxR_H-NS
- **JBrowse** (Hosts the uploaded V. harveyi sequences from which promoter sequences are retrieved)

## Examples

```
fimo hns_TCGATAAATT.meme luxCDABE_promoter.fasta
```

## Evaluation signals

- FIMO scan of the PluxCDABE promoter with motif TCGATAAATT returns exactly 13 putative H-NS binding sites with default significance scoring
- Positional overlap of predicted H-NS sites against the seven established LuxR binding sites matches the paper's annotation
- Genome-wide peak intersection yields 88 overlapping LuxR/H-NS peaks proximal to an ORF (≤500 bp) out of 506 LuxR and 302 H-NS peaks
- Peak-to-gene annotation recovers 28 co-occupied promoters driving 63 genes
- reformatFasta.pl output is a cleaned, consistently formatted multi-FASTA suitable for FIMO input

## Limitations

- The scan uses the high-affinity E. coli H-NS nucleation motif (TCGATAAATT), which may not capture lower-affinity or V. harveyi-specific H-NS sites
- Transcriptional control of each QS promoter is likely different, so luxCDABE overlap findings may not generalize to all H-NS/LuxR co-occupied promoters
- Whether LuxR also functions alongside H-NS to repress gene expression is unresolved; overlapping sites indicate co-occupancy but additional experimentation is needed for repressive roles
- The genome-wide overlap count depends on MACS2 peak calling and the ≤500 bp ORF-proximity cutoff

## Evidence
