---
name: genbank-annotation-matching
description: 'Use when GEO series expression tables must be anchored to official gene
  annotations: you have a GenBank .gbff for the organism and GEOparse-organized per-series
  tables whose gene identifiers need to be matched, filtered, and annotated before
  joining.'
when_to_use_negative:
- The data have not yet been organized by GEOannotate — gffMatch expects GEOparse-organized
  series tables, not raw .soft.gz GSE files
- No GenBank .gbff annotation file exists for the organism — COnTORT requires the
  user to provide it and cannot anchor expression data to annotations without it
- The goal is raw expression retrieval without gene-anchoring, e.g. downloading only
  processed GEO data via the FTP mirror steps
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: COnTORT
  role: Pipeline whose gffMatch module performs the GenBank annotation matching and
    writes the GeneOrf_match_output files
  repo: https://github.com/GLBRC/contort
- name: Bio (Biopython)
  role: Parses the .gbff file to build dictionaries of locus tag, old locus tag, gene
    name, synonyms, and product
- name: GEOparse
  role: Produces the organized per-series expression/metadata tables that gffMatch
    filters against the GenBank dictionaries
- name: pandas
  role: Handles the tab-delimited series tables, column construction, and joins during
    matching
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_001
  - task_004
  source_papers:
  - doi: 10.1128/mra.00587-20
    title: 'COnTORT: COmprehensive Transcriptomic ORganizational Tool for Simultaneously
      Retrieving and Organizing Numerous Gene Expression Data Sets from the NCBI Gene
      Expression Omnibus Database'
  source_content_hash: sha256:3d8da7808879053f198c455e1edc996f11f22f2e99173051840ab1f80d792b5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/contort
  tools:
  - Bio
  - Bio (Biopython)
  - COnTORT
  - GEOparse
  - Python 3
  - argparse
  - ftplib
  - pandas
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# genbank-annotation-matching

## Summary

COnTORT's gffMatch step matches GEOparse-organized GEO expression data against dictionaries built from a user-provided GenBank (.gbff) file, retaining only rows whose genes match GenBank annotations and prepending five annotation columns (Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product) that later serve as join keys.

## When to use

Use when GEO series expression tables must be anchored to official gene annotations: you have a GenBank .gbff for the organism and GEOparse-organized per-series tables whose gene identifiers need to be matched, filtered, and annotated before joining.

## When NOT to use

- The data have not yet been organized by GEOannotate — gffMatch expects GEOparse-organized series tables, not raw .soft.gz GSE files
- No GenBank .gbff annotation file exists for the organism — COnTORT requires the user to provide it and cannot anchor expression data to annotations without it
- The goal is raw expression retrieval without gene-anchoring, e.g. downloading only processed GEO data via the FTP mirror steps

## Inputs

- User-provided GenBank gene annotation file for the organism (.gbff, downloaded from NCBI)
- GEOparse-organized expression tables produced by the GEOannotate step (per-GSE series files with gene metadata concatenated to normalized log2 data)

## Outputs

- Per-series tab-delimited files in the GeneOrf_match_output directory with five annotation columns (Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product) followed by log2 normalized expression columns keyed by gene annotation

## How to apply

Load the organism's .gbff file with the Biopython (Bio) module and build dictionaries mapping each gene record to its locus tag, old locus tag, gene name, gene synonyms, and product. Read the GEOparse-organized expression tables produced by the GEOannotate step, then use the GenBank-derived dictionaries to search the organized data and retain only rows with annotation matches. Write per-series files to the GeneOrf_match_output directory with the five annotation columns followed by the log2 normalized expression columns. Fill 'N/A' where an annotation field is absent from the GenBank file and retain all blanks for consistency; the five annotation columns then act as the key for mean-centering and joining all series into a single tab-delimited matrix.

## Related tools

- **COnTORT** (Pipeline whose gffMatch module performs the GenBank annotation matching and writes the GeneOrf_match_output files) — https://github.com/GLBRC/contort
- **Bio (Biopython)** (Parses the .gbff file to build dictionaries of locus tag, old locus tag, gene name, synonyms, and product)
- **GEOparse** (Produces the organized per-series expression/metadata tables that gffMatch filters against the GenBank dictionaries)
- **pandas** (Handles the tab-delimited series tables, column construction, and joins during matching)

## Examples

```
git clone https://github.com/GLBRC/contort.git; python contort.py  # supply the organism .gbff when prompted; gffMatch writes GeneOrf_match_output/
```

## Evaluation signals

- Output file is tab-delimited and the first five column headers are exactly Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, and Product
- Row count equals the number of annotated genes matched in the GenBank file (the gene/CDS count in the GBFF)
- Locus_Tag fields are populated for annotated genes; 'N/A' appears only where the annotation is absent from the GenBank file
- Only expression rows matching a GenBank annotation are retained — unmatched genes are dropped
- Blanks are retained in the annotation fields for consistency with downstream joins

## Limitations

- COnTORT can only download and combine normalized and processed data imported into NCBI GEO
- Annotations not present in the user-provided GenBank file are listed as 'N/A', so completeness of matching depends entirely on the provided .gbff
- The README refers to matching 'gene IDs from the GFF' in the directory description while gffMatch actually uses the GenBank file — terminology inconsistency to be aware of when auditing outputs

## Evidence

- [abstract] The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product): "The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product)"
- [readme] Using dictionaries created from the GenBank file for the orgainsm, this script will search for matches to the gene annotation in the GenBank file and retain only those data with matches.: "Using dictionaries created from the GenBank file for the orgainsm, this script will search for matches to the gene annotation in the GenBank file and retain only those data with matches."
- [readme] Note that annotations no present in the GenBank file are listed with "N/A".: "Note that annotations no present in the GenBank file are listed with "N/A"."
- [abstract] COnTORT requires only a GenBank gene annotation file for the organism of choice: "COnTORT requires only a GenBank gene annotation file for the organism of choice"
