---
name: conservation-distribution-boxplot-visualization
description: Use when a research question requires scoring genomic intervals for evolutionary
  conservation, e.g. ranking candidate regulatory regions by PhastCons average scores
  across annotated gene features.
when_to_use_negative:
- The input is already a per-region or per-feature conservation score table — only
  the boxplot aggregation in R remains, not the scorePhastCons track-scoring step
- The task concerns raw high-throughput sequencing signal (BAM/SAM ChIP-seq or RNA-seq)
  rather than genome conservation tracks — use SeqCode's sequencing-oriented commands
  like produceTSSplots or recoverChIPlevels instead
- The genome assembly is not available as a UCSC chromosome-size file and RefSeq annotation
  pair, since SeqCode relies on RefSeq reference annotations for the genome release
  being analyzed
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3129
- http://edamontology.org/topic_0622
tools:
- name: SeqCode
  role: Provides the scorePhastCons binary that computes average PhastCons conservation
    scores on genomic coordinate sets against the phastCons track; internally generates
    the R scripts used for the boxplot PDF output
  repo: https://github.com/eblancoga/seqcode
- name: R
  role: Basic R core packages aggregate the per-feature-class score distributions
    and generate the comparative boxplot as a PDF
- name: UCSC genome browser
  role: Source of the phastCons30way conservation track, chromInfo.txt and refGene.txt
    files for the mm9 assembly
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:d9e9d291aceeb568f06053925ae8263171ab66ab947adb8402731f252f43f63d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - R
  - SeqCode
  - UCSC genome browser
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3129
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# conservation-distribution-boxplot-visualization

## Summary

Scores evolutionary conservation of mouse gene feature classes (CDS, 5'UTR, 3'UTR, promoters, introns, intergenic regions) against a UCSC PhastCons multiple-alignment track using SeqCode's scorePhastCons, then aggregates per-feature distributions into a comparative boxplot (PDF) that ranks regions by their potential to contain conserved regulatory blocks.

## When to use

Use when a research question requires scoring genomic intervals for evolutionary conservation, e.g. ranking candidate regulatory regions by PhastCons average scores across annotated gene features.

## When NOT to use

- The input is already a per-region or per-feature conservation score table — only the boxplot aggregation in R remains, not the scorePhastCons track-scoring step
- The task concerns raw high-throughput sequencing signal (BAM/SAM ChIP-seq or RNA-seq) rather than genome conservation tracks — use SeqCode's sequencing-oriented commands like produceTSSplots or recoverChIPlevels instead
- The genome assembly is not available as a UCSC chromosome-size file and RefSeq annotation pair, since SeqCode relies on RefSeq reference annotations for the genome release being analyzed

## Inputs

- Mouse mm9 phastCons30way multiple-alignment conservation track (UCSC download, uncompressed)
- UCSC mm9 chromInfo.txt (list of chromosomes and their sizes)
- UCSC mm9 refGene.txt (RefSeq transcript annotations)
- Coordinate sets per genomic feature class (CDS, 5'UTR, 3'UTR, promoters, downstream regions, introns, intergenic regions) derived from refGene

## Outputs

- Per-region PhastCons average conservation scores for each feature class
- Comparative boxplot of conservation score distributions across feature sets (PDF, plus the R scripts used to generate it)

## How to apply

Download the mouse mm9 phastCons30way conservation track plus chromInfo.txt and refGene.txt from the UCSC genome browser downloads and uncompress them, since SeqCode commands require the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations. Parse the RefSeq refGene annotations to derive coordinate sets for each feature class (CDS, 5'UTR, 3'UTR, promoters, downstream regions, introns, intergenic regions). Run the SeqCode scorePhastCons binary on each feature coordinate set against the phastCons track to compute per-region average conservation scores, using the -v verbose option to monitor the run. Aggregate scores per feature class and use basic R core packages (the R scripts that SeqCode internally generates for plots) to draw a comparative boxplot of the score distributions, exported as a valid PDF.

## Related tools

- **SeqCode** (Provides the scorePhastCons binary that computes average PhastCons conservation scores on genomic coordinate sets against the phastCons track; internally generates the R scripts used for the boxplot PDF output) — https://github.com/eblancoga/seqcode
- **R** (Basic R core packages aggregate the per-feature-class score distributions and generate the comparative boxplot as a PDF)
- **UCSC genome browser** (Source of the phastCons30way conservation track, chromInfo.txt and refGene.txt files for the mm9 assembly)

## Evaluation signals

- The score distributions follow the expected biological pattern of Fig. 4: conserved elements such as CDS and promoters score higher than intergenic regions
- The final boxplot is written as a valid, openable PDF file
- Per-feature-class coordinate sets are correctly derived from refGene.txt (CDS, 5'UTR, 3'UTR, promoters, downstream, introns, intergenic all present and non-overlapping as appropriate)
- Running SeqCode with the -v verbose option reports basic steps of the procedure without errors
- Each feature class yields a non-empty distribution of PhastCons scores in the output folder together with the R script used to render the plot

## Limitations

- The workflow validates only qualitatively against the expected conservation pattern (e.g. CDS/promoters above intergenic), not against a numeric threshold
- The command-line release is documented as intended for local processing, while the SeqCode website is intended as a teaching resource with restrictions on input file sizes
- Generation of genome-wide profiles in bigwig format is not supported and deferred to future versions, so scores must be consumed as per-region values rather than genome-wide tracks

## Evidence

- [methods] SeqCode is a family of applications designed to develop high-quality images and perform genome-wide calculations from high-throughput sequencing experiments.: "SeqCode is a family of applications designed to develop high-quality images and perform genome-wide calculations from high-throughput sequencing experiments."
- [readme] Calculate the evolutionary conservation score on a set of genomic regions: "Calculate the evolutionary conservation score on a set of genomic regions"
- [readme] Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations.: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations."
- [readme] Graphical outputs are produced by running R scripts that will be stored in the resulting folder as well.: "Graphical outputs are produced by running R scripts that will be stored in the resulting folder as well."
