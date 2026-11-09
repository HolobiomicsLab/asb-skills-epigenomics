---
name: ucsc-genome-annotation-retrieval
description: Use when a SeqCode command (e.g. genomeDistribution, produceTSSmaps,
  matchpeaksgenes) requires genome annotation inputs and the correct chromosome-size
  and RefSeq transcript files must be fetched for a specific genome assembly.
when_to_use_negative:
- The analysis does not involve SeqCode or any tool that consumes UCSC chromInfo/refGene
  tables — other annotation formats (e.g. GTF/GENCODE) should not be substituted directly.
- Peak coordinates were mapped to a different assembly than the annotation being retrieved
  (e.g. reads mapped to mm10 but mm9 refGene downloaded).
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0080
tools:
- name: SeqCode
  role: Consumer of the retrieved chromInfo and refGene files; its commands require
    both as inputs for genomic feature annotation and plotting
  repo: https://github.com/eblancoga/seqcode
- name: UCSC genome browser
  role: Source from which the chromosome size file (chromInfo.txt) and RefSeq annotation
    (refGene.txt) are retrieved per assembly
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:389238c810df5c94a105c050b4073ec5e06af47a14728fdcff147bde9d59d4f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - SeqCode
  - UCSC genome browser
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ucsc-genome-annotation-retrieval

## Summary

Retrieve the two genome-specific reference files that SeqCode commands require — the chromosome size list (chromInfo.txt) and the RefSeq gene transcript annotation (refGene.txt) — from the UCSC genome browser downloads for the assembly under analysis (e.g. mouse mm9).

## When to use

Use when a SeqCode command (e.g. genomeDistribution, produceTSSmaps, matchpeaksgenes) requires genome annotation inputs and the correct chromosome-size and RefSeq transcript files must be fetched for a specific genome assembly.

## When NOT to use

- The analysis does not involve SeqCode or any tool that consumes UCSC chromInfo/refGene tables — other annotation formats (e.g. GTF/GENCODE) should not be substituted directly.
- Peak coordinates were mapped to a different assembly than the annotation being retrieved (e.g. reads mapped to mm10 but mm9 refGene downloaded).

## Inputs

- Genome assembly identifier (e.g. mm9) matching the mapped reads
- UCSC genome browser download server URL for the chosen assembly

## Outputs

- chromInfo.txt — list of chromosomes and their sizes for the assembly
- refGene.txt — RefSeq gene transcript annotation table for the assembly

## How to apply

Identify the genome assembly used in the upstream analysis (e.g. mouse mm9 for mESC H3K4me3/H3K27me3 peak calling). Download two files from the UCSC genome browser for that exact assembly: chromInfo.txt (the list of chromosomes and their sizes) and refGene.txt (the RefSeq gene transcript annotations). SeqCode relies on RefSeq reference annotations and can handle any genome/assembly release available from that consortium, so the assembly must match the one used for read mapping (mm9, mm10, hg38, dm3 in the paper). Supply both files as the first arguments to SeqCode commands such as genomeDistribution; a mismatched assembly would mis-assign peaks to promoters, intergenic, or intragenic features.

## Related tools

- **SeqCode** (Consumer of the retrieved chromInfo and refGene files; its commands require both as inputs for genomic feature annotation and plotting) — https://github.com/eblancoga/seqcode
- **UCSC genome browser** (Source from which the chromosome size file (chromInfo.txt) and RefSeq annotation (refGene.txt) are retrieved per assembly)

## Examples

```
bin/genomeDistribution chromInfo.txt refGene.txt bivalent_peaks.bed example
```

## Evaluation signals

- The chromInfo.txt assembly matches the assembly used for read mapping and peak calling (e.g. mm9 peak BEDs paired with mm9 chromInfo/refGene).
- SeqCode commands run without chromosome-length warnings; every chromosome in the peak BED is present in chromInfo.txt.
- genomeDistribution output assigns every peak to promoter, intergenic, or intragenic features with no 'unknown' or unassigned interval class.
- Downloaded refGene.txt corresponds to the RefSeq release SeqCode expects (UCSC table format) rather than a GTF or other annotation format.

## Limitations

- SeqCode relies specifically on RefSeq reference annotations; annotations from other catalogues (e.g. GENCODE) are not the documented input.
- The assembly release must be consistent across read mapping, peak calling, and the retrieved annotation files, or feature assignment (promoter/intergenic/intragenic) will be incorrect.

## Evidence

- [readme] Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations.: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations."
- [readme] Both files can be easily retrieved for every particular genome assembly from the UCSC genome browser.: "Both files can be easily retrieved for every particular genome assembly from the UCSC genome browser."
- [methods] SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release of every organism that is available from this consortium.: "SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release of every organism that is available from this consortium."
- [methods] Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3).: "Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3)."
