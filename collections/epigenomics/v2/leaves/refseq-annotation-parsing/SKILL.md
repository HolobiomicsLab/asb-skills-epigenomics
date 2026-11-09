---
name: refseq-annotation-parsing
description: Use when running SeqCode (or reproducing its workflows) on sequencing
  data for a genome assembly, since most commands require the chromosome list and
  RefSeq gene transcript annotation files as inputs.
when_to_use_negative:
- The analysis relies on non-RefSeq gene models (e.g. Gencode/Ensembl GTF), since
  SeqCode's unit of information is the RefSeq annotation file
- The target genome assembly is not available from the RefSeq/UCSC consortium releases
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: SeqCode
  role: Consumes the parsed chromInfo.txt and refGene.txt files as the two mandatory
    annotation inputs for most command-line applications
  repo: https://github.com/eblancoga/seqcode
- name: UCSC genome browser
  role: Source from which the chromosome-size and RefSeq refGene annotation files
    are retrieved per assembly
- name: R
  role: Used to aggregate scores and render boxplots/PDFs derived from feature coordinate
    sets parsed from refGene annotations
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:d0657c824881f427d2df51458202eeeebb6c5e10356161f1299bb9fee26829b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - R
  - SAMtools
  - SeqCode
  - UCSC genome browser
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# refseq-annotation-parsing

## Summary

Retrieve and parse the UCSC genome-browser chromosome-size file (chromInfo.txt) and RefSeq refGene.txt transcript annotations required by every SeqCode command, deriving coordinate sets for feature classes (CDS, 5'UTR, 3'UTR, promoters, introns, intergenic) on a chosen genome assembly.

## When to use

Use when running SeqCode (or reproducing its workflows) on sequencing data for a genome assembly, since most commands require the chromosome list and RefSeq gene transcript annotation files as inputs.

## When NOT to use

- The analysis relies on non-RefSeq gene models (e.g. Gencode/Ensembl GTF), since SeqCode's unit of information is the RefSeq annotation file
- The target genome assembly is not available from the RefSeq/UCSC consortium releases

## Inputs

- UCSC chromInfo.txt chromosome-size file for the target assembly (e.g. mm9)
- UCSC refGene.txt RefSeq transcript annotation file for the target assembly
- Genome assembly identifier used for read mapping (mm9, mm10, hg38, dm3)

## Outputs

- Per-feature-class coordinate sets (CDS, 5'UTR, 3'UTR, promoters, downstream, introns, intergenic) as BED-like interval files
- Chromosome list and sizes file ready for SeqCode commands
- Validated annotation inputs for downstream SeqCode functions (produceTSSplots, produceTSSmaps, scorePhastCons)

## How to apply

Download the chromInfo.txt (list of chromosomes and their sizes) and refGene.txt (RefSeq gene transcript annotations) for the specific assembly (e.g. mm9) from the UCSC genome browser downloads, and uncompress them before use. Parse the refGene transcript records to derive coordinate sets for each feature class — CDS, 5'UTR, 3'UTR, promoters, downstream regions, introns, and intergenic regions — as in the PhastCons conservation-landscape workflow (task_007), or supply the files directly to commands like produceTSSplots/produceTSSmaps alongside the chromosome file, BAM, and a target gene list. SeqCode relies on RefSeq reference annotations and can deal with the genome and assembly release of every organism available from that consortium, so match the annotation release to the assembly used for read mapping (e.g. mm9, mm10, hg38, dm3 mapped with Bowtie). Verify the parsed coordinates by running the SeqCode binaries (built via 'make all' into bin/) with the verbose option (-v) and confirming the expected output folder and plots are generated.

## Related tools

- **SeqCode** (Consumes the parsed chromInfo.txt and refGene.txt files as the two mandatory annotation inputs for most command-line applications) — https://github.com/eblancoga/seqcode
- **UCSC genome browser** (Source from which the chromosome-size and RefSeq refGene annotation files are retrieved per assembly)
- **R** (Used to aggregate scores and render boxplots/PDFs derived from feature coordinate sets parsed from refGene annotations)

## Examples

```
bin/produceTSSmaps tests/inputs/ChromInfo.txt tests/inputs/refGene.txt tests/inputs/H3K4me3_chr10.bam tests/inputs/genes_chr10.txt test_1 5000
```

## Evaluation signals

- Both chromInfo.txt and refGene.txt for the exact assembly (e.g. mm9) are present, uncompressed, and parse without errors before running SeqCode binaries
- Derived feature-class coordinate sets (CDS, promoters, UTRs, introns, intergenic) are non-empty and lie within chromosome sizes listed in chromInfo.txt
- SeqCode runs with -v complete all stages and the output folder contains the expected data files and R scripts
- Downstream plots (e.g. PlotHEATmap PDFs or conservation boxplots) are generated as valid PDF files from the annotation-driven workflow
- Conservation/coverage distributions follow the expected biological pattern (CDS and promoters score higher than intergenic regions)

## Limitations

- SeqCode relies on RefSeq reference annotations; annotations from other catalogs are not supported
- The annotation release must match the genome assembly used for read mapping (mm9 vs mm10 vs hg38 vs dm3 mismatches will corrupt coordinate interpretation)

## Evidence

- [discussion] The unit of information in SeqCode is the RefSeq annotation file, which is provided by most genome browsers: "The unit of information in SeqCode is the RefSeq annotation file, which is provided by most genome browsers"
- [methods] SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release of every organism that is available from this consortium.: "SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release of every organism that is available from this consortium."
- [readme] Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations.: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations."
- [other] Retrieve the mouse mm9 phastCons30way multiple-alignment conservation track and the mm9 chromInfo.txt and refGene.txt annotation files from the UCSC genome browser downloads, uncompressing them: [evidence span withheld — anchor status: not_found]
