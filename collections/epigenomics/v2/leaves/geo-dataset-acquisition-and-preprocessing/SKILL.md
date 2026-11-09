---
name: geo-dataset-acquisition-and-preprocessing
description: Use when a replication or comparison task requires downloading a public
  ChIP-Seq dataset with matched treated and control runs and formatting short reads
  (BED/ELAND, single- or pair-end) for HPeak-style peak calling.
when_to_use_negative:
- The dataset is not a sequencing-based ChIP experiment (e.g., ChIP-chip probe data
  such as the STAT1 ChIP-chip comparison set) and lacks short reads to extend into
  HDFs.
- Only treated data are available without a control run, and the analysis plan requires
  HPeak's treated+control refinement mode.
- The goal is histone-modification comparative analysis, which the article notes is
  the domain of methods like ChIPDiff rather than HPeak.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: HPeak
  role: Consumes the preprocessed BED-format treated and control read files (-t/-pe
    with -c) and produces WIG, FASTA, and annotation outputs; flags -fmin/-fmax, -r,
    and -isize carry the preprocessing metadata (fragment-length bounds, read length,
    insert size).
  repo: https://github.com/cauyrd/HPeak
- name: Genomatix suite
  role: Downstream recipient of the FASTA sequences exported from peak regions, scanned
    with MatInspector over Genomatix MatBase 7.0 vertebrate motif matrices.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:f5547f31c432592bee99b34f98909d73e979863f4b483a389960bf20f862079b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cauyrd/HPeak
  tools:
  - ChIPseeqer
  - CisGenome
  - FindPeaks
  - Genomatix suite
  - HPeak
  - MACS
  - MatInspector
  - PeakFinder
  - SISSRs
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# geo-dataset-acquisition-and-preprocessing

## Summary

Acquire a public ChIP-Seq dataset (e.g., the NRSF ChIP-Seq data, GEO accession GSE13047, Illumina platform) with both ChIP-enriched (treated) and control sequencing runs, and preprocess the reads into BED-format input suitable for HPeak peak calling and downstream motif-enrichment comparison.

## When to use

Use when a replication or comparison task requires downloading a public ChIP-Seq dataset with matched treated and control runs and formatting short reads (BED/ELAND, single- or pair-end) for HPeak-style peak calling.

## When NOT to use

- The dataset is not a sequencing-based ChIP experiment (e.g., ChIP-chip probe data such as the STAT1 ChIP-chip comparison set) and lacks short reads to extend into HDFs.
- Only treated data are available without a control run, and the analysis plan requires HPeak's treated+control refinement mode.
- The goal is histone-modification comparative analysis, which the article notes is the domain of methods like ChIPDiff rather than HPeak.

## Inputs

- GEO accession GSE13047 NRSF ChIP-Seq dataset (Illumina platform), including ChIP-enriched (treated) and control sequencing runs
- Short-read alignment files in BED format (ELAND also supported); pair-end input requires insert-size information

## Outputs

- Treatment and control input files formatted for HPeak (e.g., TREATMENT.inp, CONTROL.inp)
- HPeak output files: WIG visualization file, FASTA sequence file of peak regions for downstream motif analysis, and annotation file

## How to apply

Retrieve the dataset from GEO (GSE13047, NRSF ChIP-Seq on the Illumina platform), confirming that both the ChIP-enriched (treated) and control sequencing runs are present since HPeak's treated+control mode requires the control sample for refinement. Convert the raw read alignments to BED format (ELAND is also supported) and record the read length, which is passed via the -r flag and is critical for HPeak's directional extension of each short read into a hypothetical DNA fragment (HDF). For single-end data use the -t TREATMENT.inp and -c CONTROL.inp arguments; for pair-ended data use -pe with -isize specifying the total insert size of the pair (e.g., 200). Set fragment-length bounds (e.g., -fmin 100 -fmax 300 for single-end) and request the annotation, WIG, sequence (FASTA), and intermediate outputs (-ann -wig -seq -interfiles) so peak regions can later be exported as FASTA for MatInspector motif scanning.

## Related tools

- **HPeak** (Consumes the preprocessed BED-format treated and control read files (-t/-pe with -c) and produces WIG, FASTA, and annotation outputs; flags -fmin/-fmax, -r, and -isize carry the preprocessing metadata (fragment-length bounds, read length, insert size).) — https://github.com/cauyrd/HPeak
- **Genomatix suite** (Downstream recipient of the FASTA sequences exported from peak regions, scanned with MatInspector over Genomatix MatBase 7.0 vertebrate motif matrices.)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n NRSF -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- Both treated and control sequencing runs from GSE13047 are retrieved and formatted; the treated-only and treated+control HPeak runs can both be launched.
- Peak counts from downstream peak calling fall within the reported range of 1,935 peaks (PeakFinder) to 5,243 peaks (SISSRs) on the NRSF data.
- The FASTA sequence file of called peaks is generated (via -seq) and is scannable by MatInspector without format errors.
- Read length supplied via -r matches the actual Illumina read length of the downloaded runs, since the README notes it is important from experience.
- The WIG output visualizes peaks correctly when loaded into the UCSC genome browser.

## Limitations

- The HPeak README lists supported input formats as default BED and ELAND, with SAM and BAM only planned ('Will add SAM and BAM'), so modern BAM files must be converted first.
- Evaluation of peak-calling results on experimental data is inherently uncertain because very little true positive/negative information is available, so reproduced peak counts should be interpreted with caution.
- The current HPeak algorithm does not distinguish reads of different orientation within a peak, limiting preprocessing strategies that rely on strand information for summit pinpointing.

## Evidence

- [discussion] HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] we followed the method used by Robertson et al. [11] to extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF): "we followed the method used by Robertson et al. [11] to extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF)"
- [readme] perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles: "perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles"
- [other] Download the NRSF ChIP-Seq dataset (GEO accession GSE13047, Illumina platform) including both ChIP-enriched (treated) and control sequencing runs.: [evidence span withheld — anchor status: not_found]
- [discussion] These include a wiggle (WIG) format file for visualization, a sequence file for subsequent motif analysis and an annotation file: "These include a wiggle (WIG) format file for visualization, a sequence file for subsequent motif analysis and an annotation file"
