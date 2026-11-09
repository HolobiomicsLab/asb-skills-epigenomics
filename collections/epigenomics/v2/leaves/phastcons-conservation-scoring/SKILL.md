---
name: phastcons-conservation-scoring
description: Use when you need to score evolutionary conservation of genomic regions
  (e.g. rank candidate regulatory blocks) using a PhastCons multiple-alignment track
  plus RefSeq annotation, and compare score distributions across feature classes.
when_to_use_negative:
- Your input is already a per-region conservation score table or conservation profile
  in bigWig format — SeqCode defers bigWig genome-wide profile generation to future
  versions
- You need conservation scoring for an organism/assembly lacking a UCSC PhastCons
  track or RefSeq annotation, since both files are mandatory inputs
- You are working through the SeqCode web portal, which is intended as a teaching
  resource with restrictions on input set/size rather than large local conservation
  scans
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SeqCode
  role: Provides the scorePhastCons ANSI C binary that computes the evolutionary conservation
    score on a set of genomic regions against the PhastCons track
  repo: https://github.com/eblancoga/seqcode
- name: R
  role: Runs the plotting scripts stored in the output folder to generate the comparative
    boxplot PDF of conservation score distributions
- name: UCSC genome browser
  role: Source of the PhastCons track, chromInfo.txt and refGene.txt files required
    for the analyzed assembly
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:dd7d27aff17b2367dce81507450db1c1aedc737c5612d41a39256aac5902b494
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - R
  - SeqCode
  - UCSC genome browser
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# phastcons-conservation-scoring

## Summary

Compute PhastCons evolutionary conservation scores over sets of genomic intervals (CDS, UTRs, promoters, introns, intergenic) using SeqCode's scorePhastCons application, then aggregate per feature class to build a comparative conservation landscape that can rank intervals by their potential to contain conserved regulatory blocks.

## When to use

Use when you need to score evolutionary conservation of genomic regions (e.g. rank candidate regulatory blocks) using a PhastCons multiple-alignment track plus RefSeq annotation, and compare score distributions across feature classes.

## When NOT to use

- Your input is already a per-region conservation score table or conservation profile in bigWig format — SeqCode defers bigWig genome-wide profile generation to future versions
- You need conservation scoring for an organism/assembly lacking a UCSC PhastCons track or RefSeq annotation, since both files are mandatory inputs
- You are working through the SeqCode web portal, which is intended as a teaching resource with restrictions on input set/size rather than large local conservation scans

## Inputs

- PhastCons conservation track for the target genome assembly (e.g. mm9 phastCons30way multiple-alignment track from UCSC)
- chromInfo.txt chromosome sizes file for the assembly from UCSC
- refGene.txt RefSeq transcript annotation file from UCSC
- Coordinate sets (BED-like interval lists) per genomic feature class: CDS, 5'UTR, 3'UTR, promoters, downstream regions, introns, intergenic

## Outputs

- Per-region PhastCons average conservation scores for each feature coordinate set
- Comparative boxplot of conservation score distributions across feature classes, written as a valid PDF (with R plotting scripts and data files stored in the output folder)

## How to apply

Retrieve the genome's PhastCons track (e.g. mouse mm9 phastCons30way), chromInfo.txt (chromosome sizes) and refGene.txt from the UCSC genome browser downloads and uncompress them, since SeqCode requires the chromosome list and RefSeq transcript annotations for the analyzed assembly. Parse refGene to derive coordinate sets for each feature class (CDS, 5'UTR, 3'UTR, promoters, downstream regions, introns, intergenic). Run the scorePhastCons binary per coordinate set against the PhastCons track to obtain per-region average conservation scores. Aggregate scores per class and use core R (SeqCode stores and runs the plotting scripts itself) to draw a comparative boxplot; decide correctness by whether the distributions match expected biology, e.g. CDS and promoters scoring above intergenic regions.

## Related tools

- **SeqCode** (Provides the scorePhastCons ANSI C binary that computes the evolutionary conservation score on a set of genomic regions against the PhastCons track) — https://github.com/eblancoga/seqcode
- **R** (Runs the plotting scripts stored in the output folder to generate the comparative boxplot PDF of conservation score distributions)
- **UCSC genome browser** (Source of the PhastCons track, chromInfo.txt and refGene.txt files required for the analyzed assembly)

## Examples

```
bin/scorePhastCons ChromInfo.txt refGene.txt phastCons30way/ cds_regions.txt cds_scores
```

## Evaluation signals

- The conservation score distributions follow the expected biological pattern: conserved elements such as CDS and promoters score higher than intergenic regions (qualitatively matching Fig. 4 of the paper)
- The final boxplot is written as a valid, openable PDF file in the SeqCode output folder alongside the R plotting scripts and data files
- Every feature class (CDS, 5'UTR, 3'UTR, promoters, downstream, introns, intergenic) yields a non-empty per-region score set derived from its parsed refGene coordinates
- All scores fall within the valid PhastCons range (0–1), with low/zero scores concentrated in fast-evolving intergenic/intronic classes
- The correct assembly-specific chromInfo.txt and refGene.txt pair was used, matching the PhastCons track's genome release (e.g. mm9)

## Limitations

- Conservation scoring is restricted to organisms/assemblies with available UCSC PhastCons tracks and RefSeq annotations; SeqCode relies on the RefSeq reference annotations
- Generation of genome-wide profiles in bigwig format is not supported and deferred to future versions
- The web tools impose limits on input set/size; large conservation scans must use the standalone command line, and graphical outputs require R to be installed

## Evidence

- [readme] Calculate the evolutionary conservation score on a set of genomic regions: "Calculate the evolutionary conservation score on a set of genomic regions"
- [readme] Most programs require chromInfo and RefSeq annotations from UCSC: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations."
- [readme] Graphical outputs are produced by running R scripts: "Graphical outputs are produced by running R scripts that will be stored in the resulting folder as well."
- [methods] SeqCode performs genome-wide calculations from sequencing experiments: "SeqCode is a family of applications designed to develop high-quality images and perform genome-wide calculations from high-throughput sequencing experiments."
