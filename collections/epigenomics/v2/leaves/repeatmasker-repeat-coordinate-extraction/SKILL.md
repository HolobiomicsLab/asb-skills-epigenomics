---
name: repeatmasker-repeat-coordinate-extraction
description: Use when an analysis requires strand-annotated genomic coordinates of
  transposable elements (LINE, SINE, LTR subclasses) from UCSC hg38 RepeatMasker as
  motif input for interval-overlap or asymmetry tooling.
when_to_use_negative:
- The analysis needs nucleotide-level repeat sequences or consensus motifs rather
  than genomic interval coordinates
- The target genome build is not hg38 — the extracted coordinates are genome-build-specific
  and would require liftOver
- A feature table with strand annotation already exists (e.g. pre-formatted BED files),
  making re-extraction from rmsk redundant
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Asymmetron
  role: Downstream consumer of the extracted strand-annotated repeat BED files; its
    contained_asymmetries function counts same versus opposite strand motif occurrences
    within regions
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Used by Asymmetron for the BED-formatted interval-overlap comparisons against
    the extracted repeat coordinates
- name: Python
  role: Pandas/Python-based parsing of the rmsk.txt.gz table and filtering to repeat
    subclasses (LINE, SINE, LTR; Alu, MIR, L1, L2)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:1a177d9094674c65620c4ac088bcf659b42b9cba01f033f2fb8e1906646888b6
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# RepeatMasker repeat coordinate extraction

## Summary

Extracts chromosome, start, end and strand coordinates for selected repeat classes (e.g. Alu SINEs, MIRs, L1/L2 LINEs) from the UCSC hg38 RepeatMasker table (rmsk.txt.gz, 11 March 2019 version) into strand-annotated BED files suitable for strand asymmetry analysis with Asymmetron.

## When to use

Use when an analysis requires strand-annotated genomic coordinates of transposable elements (LINE, SINE, LTR subclasses) from UCSC hg38 RepeatMasker as motif input for interval-overlap or asymmetry tooling.

## When NOT to use

- The analysis needs nucleotide-level repeat sequences or consensus motifs rather than genomic interval coordinates
- The target genome build is not hg38 — the extracted coordinates are genome-build-specific and would require liftOver
- A feature table with strand annotation already exists (e.g. pre-formatted BED files), making re-extraction from rmsk redundant

## Inputs

- UCSC hg38 rmsk.txt.gz (RepeatMasker table, version from 11 March 2019) downloaded from hgdownload.soe.ucsc.edu
- Repeat class/subclass selection (e.g. LINE, SINE, LTR; Alu, MIR, L1, L2)

## Outputs

- Strand-annotated BED file(s) of repeat coordinates with chromosome, start, end and strand ('+'/'-') columns, one per repeat subclass
- Subclass-specific motif files (e.g. Alu BED, MIR BED, L1 BED, L2 BED) ready for Asymmetron contained_asymmetries

## How to apply

Download rmsk.txt.gz from hgdownload.soe.ucsc.edu (version from 11 March 2019) and parse the RepeatMasker table to retain chromosome, start, end and strand columns. Filter the table to the repeat class of interest — the Asymmetron workflow filtered to LINE, SINE and LTR transposable elements, and further subdivided SINEs into Alu and MIR and LINEs into L1 and L2 to test subclass-specific asymmetries. Write each subset as a strand-annotated BED file with '+' or '-' in the strand column, since Asymmetron requires DNA features with strand annotation. These BED files then serve as the motif input to Asymmetron's contained_asymmetries function against strand-assigned region files (e.g. GENCODE v33 protein-coding genes) to count same- versus opposite-strand occurrences.

## Related tools

- **Asymmetron** (Downstream consumer of the extracted strand-annotated repeat BED files; its contained_asymmetries function counts same versus opposite strand motif occurrences within regions) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Used by Asymmetron for the BED-formatted interval-overlap comparisons against the extracted repeat coordinates)
- **Python** (Pandas/Python-based parsing of the rmsk.txt.gz table and filtering to repeat subclasses (LINE, SINE, LTR; Alu, MIR, L1, L2))

## Examples

```
zcat rmsk.txt.gz | awk 'NR>1 && $12=="SINE" && $13~/Alu/ {print $6"\t"$7"\t"$8"\t"$10}' > alu_sine.bed
```

## Evaluation signals

- The output BED file contains exactly four usable columns (chromosome, start, end, strand) with strand values restricted to '+' or '-'
- Coordinate count per subclass is consistent with expected RepeatMasker copy numbers in hg38 (Alu is the dominant SINE subclass)
- Subclass filtering reproduces the paper's structure: SINEs split into Alu and MIR, LINEs split into L1 and L2
- Downstream contained_asymmetries run on the extracted Alu BED yields a transcriptional strand asymmetry ratio of 0.471 for SINEs with Bonferroni-corrected P < 0.001 (preference for the template strand, ratio < 0.5)
- Genome build is confirmed as hg38 throughout, matching the paper's human genome build

## Limitations

- Coordinates are specific to the 11 March 2019 rmsk.txt.gz version and hg38 build; other versions or builds will shift intervals
- Only genomic intervals and strand are extracted — repeat sequence content, divergence and RepeatMasker score fields are not retained unless additionally parsed
- Strand asymmetry interpretation requires pairing with strand-annotated region files (e.g. GENCODE v33 protein-coding genes); repeat BEDs alone cannot yield asymmetry statistics

## Evidence

- [methods] Coordinates of transposable elements were derived for the human genome (hg38) from the http://hgdownload.soe.ucsc.edu/goldenPath/hg38/database/rmsk.txt.gz (version from 11 March 2019): "Coordinates of transposable elements were derived for the human genome (hg38) from the http://hgdownload.soe.ucsc.edu/goldenPath/hg38/database/rmsk.txt.gz (version from 11 March 2019)"
- [methods] and were filtered to include LINE, SINE and LTR transposable elements: "and were filtered to include LINE, SINE and LTR transposable elements."
- [results] We found a preference for LINEs, SINEs and LTRs to be at the template strand (ratios 0.392, 0.471, 0.316, respectively, binomial test, Bonferroni corrected P-values < 0.001): "We found a preference for LINEs, SINEs and LTRs to be at the template strand (ratios 0.392, 0.471, 0.316, respectively, binomial test, Bonferroni corrected P-values < 0.001)"
- [abstract] Asymmetron takes as input DNA features provided with strand annotation: "Asymmetron takes as input DNA features provided with strand annotation and outputs strand asymmetries for consecutive occurrences of a single DNA feature or between pairs of features."
- [methods] its core BED-formatted file comparison functions use the package Pybedtools: "its core BED-formatted file comparison functions use the package Pybedtools"
