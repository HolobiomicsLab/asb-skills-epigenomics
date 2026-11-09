---
name: genomic-bed-file-strand-annotation
description: Use when strand-oriented genomic features (e.g. GENCODE v33 genes, PolyASite
  PAS clusters, UCSC tfbsConsSites TFBSs, JASPAR/FIMO motif tracks) must be converted
  into 4-column BED files before running Asymmetron asymmetry analyses.
when_to_use_negative:
- The feature is palindromic, so no biologically meaningful '+'/'-' orientation can
  be assigned (Asymmetron's asymmetries rely on non-palindromic orientation)
- The analysis goal is nucleotide-composition or mutation strand asymmetry computed
  from sequences rather than strand-assigned BED features
- The feature lacks strand annotation entirely and there is no overlapping oriented
  element to derive orientation from — that is the separate Asymmetron 'orientation'
  function's task
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0749
tools:
- name: Asymmetron
  role: Downstream consumer of the strand-annotated BED files; its asymmetry functions
    require DNA features provided with strand annotation
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Python wrapper driving Asymmetron's core BED-formatted file comparison functions
    on the prepared files
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: bedtools
  role: Provides the 'bedtools closest' function used to pair elements (e.g. SP1 with
    YY1) within a distance window after BED preparation
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: pandas
  role: Used to filter source annotations (e.g. GENCODE v33 to protein-coding genes)
    and write them as strand-oriented BED files
provenance:
  source_task_ids:
  - task_004
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:bbd60b0b533b4dce569f680f06d878d4e941152df9e1aca57014cff2be269b9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - bedtools
  - matplotlib
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# genomic-bed-file-strand-annotation

## Summary

Prepares strand-annotated BED files (chromosome, start, end, +/− strand) for DNA features such as TFBSs, polyadenylation signals, transposable elements and gene regions so they can be consumed by Asymmetron's consecutive, contained and pairwise strand-asymmetry functions.

## When to use

Use when strand-oriented genomic features (e.g. GENCODE v33 genes, PolyASite PAS clusters, UCSC tfbsConsSites TFBSs, JASPAR/FIMO motif tracks) must be converted into 4-column BED files before running Asymmetron asymmetry analyses.

## When NOT to use

- The feature is palindromic, so no biologically meaningful '+'/'-' orientation can be assigned (Asymmetron's asymmetries rely on non-palindromic orientation)
- The analysis goal is nucleotide-composition or mutation strand asymmetry computed from sequences rather than strand-assigned BED features
- The feature lacks strand annotation entirely and there is no overlapping oriented element to derive orientation from — that is the separate Asymmetron 'orientation' function's task

## Inputs

- Strand-oriented regions annotation (e.g. GENCODE v33 GTF filtered to protein-coding genes)
- Strand-oriented motif occurrence tracks (e.g. UCSC hg19/hg38 tfbsConsSites table for SP1 and YY1; JASPAR MA0139.1 CTCF FIMO track)
- Strand-oriented motif/element catalogs (e.g. PolyASite atlas PAS clusters on hg38; RepeatMasker hg38 rmsk LINE/SINE/LTR entries)

## Outputs

- Strand-annotated BED files with columns chromosome, start, end, strand ('+' or '-') for each feature
- Paired regions/motifs BED files ready as inputs to Asymmetron's consecutive_patterns, contained_asymmetries or pairwise_asymmetries

## How to apply

Export each feature set as a BED file with exactly four fields: chromosome, start coordinate, end coordinate, and strand sign ('+' or '-'), preserving the source track's strand convention (e.g. for JASPAR MA0139.1 CTCF motif occurrences, '+' encodes the An orientation and '-' the Bn orientation). Filter the source annotation first as the article does — GENCODE v33 filtered to protein-coding genes, RepeatMasker rmsk entries filtered to LINE/SINE/LTR, or UCSC tfbsConsSites extracts for motifs like SP1 and YY1. Keep all files on a single genome build (the article uses hg38 throughout; liftOver Repli-seq-style tracks if needed). These strand-annotated BEDs are then the direct inputs to Asymmetron's consecutive_patterns, contained_asymmetries and pairwise_asymmetries functions, whose BED-file comparisons run on Pybedtools and 'bedtools closest'. Only non-palindromic features carry meaningful strand labels, so verify the motif/element is orientable before annotating.

## Related tools

- **Asymmetron** (Downstream consumer of the strand-annotated BED files; its asymmetry functions require DNA features provided with strand annotation) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Python wrapper driving Asymmetron's core BED-formatted file comparison functions on the prepared files) — https://github.com/Ahituv-lab/Asymmetron
- **bedtools** (Provides the 'bedtools closest' function used to pair elements (e.g. SP1 with YY1) within a distance window after BED preparation) — https://github.com/Ahituv-lab/Asymmetron
- **pandas** (Used to filter source annotations (e.g. GENCODE v33 to protein-coding genes) and write them as strand-oriented BED files)

## Examples

```
python pairwise_asymmetries.py SP1_tfbsConsSites.bed YY1_tfbsConsSites.bed --min_distance=1 --max_distance=200
```

## Evaluation signals

- Each output BED file contains exactly the four fields chromosome, start, end, strand, with strand values restricted to '+' and '-'
- Strand labels follow the source track's convention (e.g. '+' for the An and '-' for the Bn orientation of JASPAR MA0139.1 CTCF motifs)
- All BED files share one genome build (article standard is hg38) before any comparison step
- Downstream Asymmetron runs succeed and reproduce reported values, e.g. PAS-in-protein-coding-gene same/opposite ratio of 0.72 with P = 2e-323, or the SP1–YY1 same/opposite and convergent/divergent ratios in pairwise_asymmetries_results.tsv
- Non-palindromic status of the element is confirmed, since palindromic sequences cannot be oriented relative to one another

## Limitations

- Only non-palindromic sequences can be oriented relative to one another; palindromic motifs yield uninformative strand labels
- Genome build mismatches (tutorial used UCSC hg19 tfbsConsSites while the article uses hg38) silently break overlap/closest comparisons unless lifted over
- Strand sign semantics are source-track dependent ('+' for An vs '-' for Bn), so misinterpreting the convention inverts the reported asymmetry direction

## Evidence

- [abstract] Asymmetron takes as input DNA features provided with strand annotation and outputs strand asymmetries for consecutive occurrences of a single DNA feature or between pairs of features.: "Asymmetron takes as input DNA features provided with strand annotation and outputs strand asymmetries for consecutive occurrences of a single DNA feature or between pairs of features."
- [methods] its core BED-formatted file comparison functions use the package Pybedtools: "its core BED-formatted file comparison functions use the package Pybedtools"
- [methods] we use the ‘bedtools closest’ function to determine the closest element in S2: "we use the ‘bedtools closest’ function to determine the closest element in S2"
- [readme] Sequences that are non palindromic can be oriented relative to one another.: "Sequences that are non palindromic can be oriented relative to one another."
- [readme] Asymmetron can identify strand asymmetries within consecutive occurrences of a single genomic element and for pairs of overlapping and non-overlapping genomic elements.: [evidence span withheld — anchor status: external_doc]
