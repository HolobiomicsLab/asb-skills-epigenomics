---
name: genome-browser-track-retrieval
description: Use when a SeqCode command (e.g. scorePhastCons, produceTSSplots) requires
  the chromInfo, refGene, or a conservation track for a specific assembly (e.g. mouse
  mm9) that must first be downloaded and uncompressed from UCSC.
when_to_use_negative:
- The analysis targets an organism or assembly for which UCSC does not distribute
  the required track (e.g. no phastCons track for the assembly) or the annotation
  is not RefSeq-based
- The input is already a processed per-region score or signal table — track retrieval
  is a pre-analysis data acquisition step, not a scoring or plotting step
- The workflow needs genome-wide profiles in bigWig format, which SeqCode does not
  support and defers to future versions
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: SeqCode
  role: Consumes the retrieved chromInfo, refGene, and phastCons track files as required
    inputs to its command-line binaries (e.g. scorePhastCons)
  repo: https://github.com/eblancoga/seqcode
- name: UCSC genome browser
  role: Source of the chromInfo.txt, refGene.txt, and phastCons30way conservation
    track downloads for the target assembly
- name: R
  role: Generates the comparative boxplot of conservation score distributions across
    feature sets after track-based scoring
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:3057295f81508263bd4fa4dcf6256203a8b517e8a051ce44a5dbb16e4d036481
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - R
  - SeqCode
  - UCSC genome browser
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# genome-browser-track-retrieval

## Summary

Retrieve and prepare genome-assembly-specific reference files (chromosome size list, RefSeq refGene annotations, and tracks such as phastCons30way conservation) from the UCSC genome browser downloads so that SeqCode command-line applications can run on a chosen genome release.

## When to use

Use when a SeqCode command (e.g. scorePhastCons, produceTSSplots) requires the chromInfo, refGene, or a conservation track for a specific assembly (e.g. mouse mm9) that must first be downloaded and uncompressed from UCSC.

## When NOT to use

- The analysis targets an organism or assembly for which UCSC does not distribute the required track (e.g. no phastCons track for the assembly) or the annotation is not RefSeq-based
- The input is already a processed per-region score or signal table — track retrieval is a pre-analysis data acquisition step, not a scoring or plotting step
- The workflow needs genome-wide profiles in bigWig format, which SeqCode does not support and defers to future versions

## Inputs

- UCSC genome browser download URL for a chosen assembly (e.g. mouse mm9)
- chromInfo.txt chromosome sizes file from UCSC
- refGene.txt RefSeq transcript annotation file from UCSC
- phastCons30way multiple-alignment conservation track for mm9
- Genomic coordinate sets (e.g. BED-style feature lists for CDS, 5'UTR, 3'UTR, promoters, introns, intergenic regions) derived from refGene

## Outputs

- Uncompressed chromInfo.txt, refGene.txt, and phastCons track files ready for SeqCode binaries
- Per-feature-class coordinate files (CDS, 5'UTR, 3'UTR, promoters, downstream, introns, intergenic) parsed from refGene.txt
- PhastCons average conservation score distributions per genomic feature class from scorePhastCons

## How to apply

Identify the genome assembly used in the analysis (e.g. mm9 for the PhastCons conservation landscape task) and download three file classes from the UCSC genome browser downloads: the chromosome list with sizes (chromInfo.txt), the RefSeq refGene.txt transcript annotations, and the relevant track data (e.g. the mm9 phastCons30way multiple-alignment conservation track). Uncompress the downloaded files so SeqCode can read them, since most SeqCode commands require exactly these two genome files (chromosome sizes and RefGene annotations) as their first arguments, plus the experiment- or track-specific data file. Keep the assembly consistent across all files: SeqCode relies on RefSeq reference annotations and can handle any genome and assembly release available from that consortium, but mixing assemblies (e.g. mm9 annotations with mm10 tracks) invalidates coordinate matching. Feed the uncompressed files directly into the downstream SeqCode binary together with the coordinate set of interest.

## Related tools

- **SeqCode** (Consumes the retrieved chromInfo, refGene, and phastCons track files as required inputs to its command-line binaries (e.g. scorePhastCons)) — https://github.com/eblancoga/seqcode
- **UCSC genome browser** (Source of the chromInfo.txt, refGene.txt, and phastCons30way conservation track downloads for the target assembly)
- **R** (Generates the comparative boxplot of conservation score distributions across feature sets after track-based scoring)

## Examples

```
bin/scorePhastCons chromInfo.txt refGene.txt phastCons30way.wigFix cds_coordinates.txt cds_scores
```

## Evaluation signals

- All three downloaded files correspond to the same assembly release (mm9 chromInfo, mm9 refGene, mm9 phastCons30way) with matching chromosome names and coordinates
- Files are successfully uncompressed and readable by the SeqCode binary — the command reports progress with the verbose option (-v) and terminates without parse errors on the chromInfo/refGene files
- scorePhastCons produces per-region conservation scores for every input feature coordinate set without dropping regions
- Aggregate boxplot follows the expected biological pattern: conserved elements (CDS, promoters) score higher than intergenic regions, qualitatively matching Fig. 4
- The comparative boxplot is written as a valid PDF file by the R script stored in the output folder

## Limitations

- SeqCode depends on RefSeq reference annotations; alternative gene catalogs are not supported by these commands
- Generation of genome-wide profiles in bigWig format is not supported and is deferred to future versions
- The website mode limits input sizes; only the command-line release has no limits on input file size, so large tracks should be processed locally
- Depending on the user platform, warning messages about the pthread library during SAMtools compilation can appear but can be safely skipped

## Evidence

- [other] Retrieve the mouse mm9 phastCons30way multiple-alignment conservation track and the mm9 chromInfo.txt and refGene.txt annotation files from the UCSC genome browser downloads: [evidence span withheld — anchor status: not_found]
- [readme] Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations.: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations."
- [methods] SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release of every organism that is available from this consortium.: "SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release of every organism that is available from this consortium."
- [discussion] Generation of genome-wide profiles in bigwig format62,63, for instance, should be considered in future versions.: "Generation of genome-wide profiles in bigwig format62,63, for instance, should be considered in future versions."
