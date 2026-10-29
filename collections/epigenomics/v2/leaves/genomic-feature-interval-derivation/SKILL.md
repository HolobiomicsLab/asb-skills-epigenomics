---
name: genomic-feature-interval-derivation
description: Use when you have a UCSC refGene.txt transcript annotation file plus
  chromInfo.txt for a genome assembly and need BED-like coordinate sets per feature
  class to feed feature-level analyses (e.g. scorePhastCons conservation scores).
when_to_use_negative:
- When you only have peak coordinates from MACS or BAM files and need gene matching
  rather than feature-class interval derivation — use SeqCode's matchpeaksgenes-style
  functions instead
- When the annotation is not RefSeq (SeqCode relies on RefSeq reference annotations
  and is designed around that catalog)
- When the analysis needs sequencing read signal per region (quality control, read
  mapping, counting) rather than annotation-derived intervals
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0749
tools:
- name: SeqCode
  role: Suite of ANSI C applications that consume the chromInfo.txt and refGene.txt
    annotation files; its derived feature coordinate inputs feed per-feature calculations
    such as scorePhastCons conservation scoring
  repo: https://github.com/eblancoga/seqcode
- name: UCSC genome browser
  role: Source of the mm9 chromInfo.txt and refGene.txt annotation files used to derive
    feature coordinate sets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:cd7e88402841b45cbab728ecdff581cb87da569258b18339fb454d5d77b713ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - R
  - SeqCode
  - UCSC genome browser
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# genomic-feature-interval-derivation

## Summary

Derives coordinate sets for genomic feature classes (CDS, 5'UTR, 3'UTR, promoters, downstream regions, introns, intergenic regions) by parsing RefSeq refGene transcript annotations, as a prerequisite for downstream per-feature calculations such as PhastCons conservation scoring with SeqCode.

## When to use

Use when you have a UCSC refGene.txt transcript annotation file plus chromInfo.txt for a genome assembly and need BED-like coordinate sets per feature class to feed feature-level analyses (e.g. scorePhastCons conservation scores).

## When NOT to use

- When you only have peak coordinates from MACS or BAM files and need gene matching rather than feature-class interval derivation — use SeqCode's matchpeaksgenes-style functions instead
- When the annotation is not RefSeq (SeqCode relies on RefSeq reference annotations and is designed around that catalog)
- When the analysis needs sequencing read signal per region (quality control, read mapping, counting) rather than annotation-derived intervals

## Inputs

- UCSC chromInfo.txt chromosome name/size list for the target assembly (e.g. mouse mm9)
- UCSC refGene.txt RefSeq gene transcript annotation file
- Genome assembly identifier (e.g. mm9, mm10, hg38, dm3)

## Outputs

- Coordinate interval sets per genomic feature class (CDS, 5'UTR, 3'UTR, promoters, downstream regions, introns, intergenic regions) in a SeqCode-compatible region format
- Derived feature coordinate files consumable by SeqCode binaries such as scorePhastCons

## How to apply

Retrieve the genome assembly's chromInfo.txt and refGene.txt from the UCSC genome browser downloads and uncompress them for use with SeqCode. Parse the RefSeq refGene transcript annotations to derive coordinate sets for each feature class — CDS, 5'UTR, 3'UTR, promoters, downstream regions, introns, and intergenic regions — using the exon/CDS start-stop fields and chromosome sizes listed in chromInfo.txt. SeqCode relies on RefSeq annotations and can deal with the genome and assembly release of every organism available from that consortium, so the parsed intervals stay consistent with the annotation catalog used by all SeqCode binaries. These derived feature coordinate files then serve as input to downstream tools (e.g. the scorePhastCons binary, which computes per-region conservation scores against a phastCons track).

## Related tools

- **SeqCode** (Suite of ANSI C applications that consume the chromInfo.txt and refGene.txt annotation files; its derived feature coordinate inputs feed per-feature calculations such as scorePhastCons conservation scoring) — https://github.com/eblancoga/seqcode
- **UCSC genome browser** (Source of the mm9 chromInfo.txt and refGene.txt annotation files used to derive feature coordinate sets)

## Examples

```
make all && bin/scorePhastCons chromInfo.txt refGene.txt CDS_coords.txt phastCons30way_mm9 output_CDS -v
```

## Evaluation signals

- Each feature-class coordinate file contains only intervals whose chromosomes appear in chromInfo.txt and whose coordinates fall within the listed chromosome sizes
- Feature sets partition annotation space consistently: CDS, UTR, intron, promoter, downstream, and intergenic sets derived from the same refGene.txt should not overlap in unintended ways (e.g. CDS vs 5'UTR within the same transcript)
- Downstream qualitative validation: conservation score distributions computed on these intervals follow the expected biological pattern, with conserved elements such as CDS and promoters scoring higher than intergenic regions (cf. Fig. 4)
- SeqCode commands run on the derived inputs without errors and write results, including any R-generated plots, into the output folder (e.g. a valid PDF boxplot)

## Limitations

- SeqCode relies on the RefSeq reference annotations, so annotations from other catalogs may not parse correctly
- Most SeqCode commands require the user to provide two genome-specific files — the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations — for the particular genome being analyzed; using mismatched assemblies breaks the workflow
- Promoter/downstream window definitions are not standardized across pipelines, so derived feature boundaries depend on the parsing choices made when interpreting refGene fields

## Evidence

- [other] Parse the RefSeq refGene transcript annotations to derive the coordinate sets for each genomic feature class (CDS, 5'UTR, 3'UTR, promoters, downstream regions, introns, intergenic regions).: [evidence span withheld — anchor status: not_found]
- [readme] Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations.: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations."
- [methods] SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release of every organism that is available from this consortium.: "SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release of every organism that is available from this consortium."
- [other] Retrieve the mouse mm9 phastCons30way multiple-alignment conservation track and the mm9 chromInfo.txt and refGene.txt annotation files from the UCSC genome browser downloads: [evidence span withheld — anchor status: not_found]
