---
name: gencode-gene-annotation-filtering
description: Use when a strand-asymmetry or motif-orientation analysis needs strand-annotated,
  protein-coding gene regions (e.g. GENCODE v33) formatted as BED input for Asymmetron's
  contained_asymmetries.
when_to_use_negative:
- The analysis does not involve strand orientation or motifs relative to genes (e.g.
  plain gene counting or expression quantification), where a full unfiltered GTF suffices
- Working with a non-human genome or an annotation release other than GENCODE (the
  article used GENCODE v33 on hg38), requiring a different source annotation
- Asymmetry is to be measured between consecutive occurrences of a single feature
  (Asymmetron's consecutive_patterns) rather than motifs contained within strand-assigned
  regions
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0780
tools:
- name: Asymmetron
  role: Provides the contained_asymmetries function that consumes the filtered, strand-annotated
    protein-coding gene BED to quantify same vs opposite strand motif occurrence
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Underlies the BED-formatted interval overlap counting used by contained_asymmetries
    on the filtered gene regions
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: scipy
  role: Two-tailed binomial test (p = 0.5) on the asymmetry counts derived from the
    filtered gene regions
- name: pandas
  role: Statistical analysis of the count table produced from the filtered gene-region
    overlaps
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:7ab6b5e8116a8ba452c3d5b14ec9361d9ac49e1430d97012afd7b114b776dddc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# gencode-gene-annotation-filtering

## Summary

Extract strand-annotated protein-coding gene regions from the GENCODE v33 annotation (hg38) as a BED file, so that strand-asymmetric occurrence of motifs within genes can be measured with Asymmetron's contained_asymmetries function.

## When to use

Use when a strand-asymmetry or motif-orientation analysis needs strand-annotated, protein-coding gene regions (e.g. GENCODE v33) formatted as BED input for Asymmetron's contained_asymmetries.

## When NOT to use

- The analysis does not involve strand orientation or motifs relative to genes (e.g. plain gene counting or expression quantification), where a full unfiltered GTF suffices
- Working with a non-human genome or an annotation release other than GENCODE (the article used GENCODE v33 on hg38), requiring a different source annotation
- Asymmetry is to be measured between consecutive occurrences of a single feature (Asymmetron's consecutive_patterns) rather than motifs contained within strand-assigned regions

## Inputs

- GENCODE v33 GTF/GFF annotation file for human genome hg38 (from gencodegenes.org)
- Optional: strand-annotated motifs BED file (e.g. Alu SINE coordinates from UCSC hg38 rmsk.txt.gz)

## Outputs

- Strand-annotated BED file of protein-coding gene regions (chromosome, start, end, strand)
- Same/opposite strand asymmetry ratios and Bonferroni-corrected two-tailed binomial test P-values when paired with contained_asymmetries

## How to apply

Download the GENCODE v33 GTF annotation from gencodegenes.org and filter it to include only protein-coding genes, as the Asymmetron study did for its hg38-based analyses. Convert the filtered gene records into a strand-annotated regions BED file retaining chromosome, start, end and strand (+/-) columns, since Asymmetron requires strand assignment for regions and motifs. Use this regions BED together with a motifs BED (e.g. Alu SINE coordinates from the UCSC hg38 rmsk table, 11 March 2019 version) as input to contained_asymmetries, which counts same- versus opposite-strand motif-region pairs via Pybedtools interval overlap. Judge success by computing the same/(same+opposite) asymmetry ratio and confirming significance with a two-tailed binomial test (p = 0.5) and Bonferroni correction, e.g. the SINE/Alu ratio of 0.471 (template-strand preference, corrected P < 0.001).

## Related tools

- **Asymmetron** (Provides the contained_asymmetries function that consumes the filtered, strand-annotated protein-coding gene BED to quantify same vs opposite strand motif occurrence) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Underlies the BED-formatted interval overlap counting used by contained_asymmetries on the filtered gene regions) — https://github.com/Ahituv-lab/Asymmetron
- **scipy** (Two-tailed binomial test (p = 0.5) on the asymmetry counts derived from the filtered gene regions)
- **pandas** (Statistical analysis of the count table produced from the filtered gene-region overlaps)

## Evaluation signals

- Filtered BED contains only records with feature type gene and gene_type 'protein_coding' from the GENCODE v33 GTF
- Every BED record carries a strand value ('+' or '-') and coordinates consistent with hg38
- contained_asymmetries on Alu motifs vs the filtered gene BED reproduces a SINE/Alu template-strand asymmetry ratio < 0.5 (reported 0.471)
- Binomial test P-values (p = 0.5, two-tailed) remain significant after Bonferroni correction (corrected P < 0.001) for Alu repeats, while MIR SINEs should not show strong asymmetry
- Ratio is defined as same-strand occurrences divided by same plus opposite-strand occurrences

## Limitations

- The asymmetry signal is strongest near the TSS and decreases with distance from it, so asymmetry estimates depend on the genomic distance window used
- Results are specific to the hg38 genome build and the GENCODE v33 release; other builds/releases require revalidation
- Multiple-testing burden requires Bonferroni correction when many motif classes are tested concurrently

## Evidence

- [methods] Gene annotation from GENCODE was used (v33); the file was derived from (https://www.gencodegenes.org/): "Gene annotation from GENCODE was used (v33); the file was derived from (https://www.gencodegenes.org/)"
- [methods] filtered to include only protein-coding genes: "and filtered to include only protein-coding genes (25)."
- [methods] strand asymmetry estimation of motifs within regions: "(ii) strand asymmetry estimation of strand-assigned motifs within strand-assigned regions"
- [results] LINEs, SINEs and LTRs template strand preference with reported ratios: "We found a preference for LINEs, SINEs and LTRs to be at the template strand (ratios 0.392, 0.471, 0.316, respectively, binomial test, Bonferroni corrected P-values < 0.001)"
- [abstract] Asymmetron input requirements: strand-annotated DNA features: "Asymmetron takes as input DNA features provided with strand annotation and outputs strand asymmetries for consecutive occurrences of a single DNA feature or between pairs of"
