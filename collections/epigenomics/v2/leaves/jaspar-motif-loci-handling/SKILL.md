---
name: jaspar-motif-loci-handling
description: Use when strand asymmetry analysis needs strand-annotated transcription
  factor binding site loci, e.g. obtaining CTCF (MA0139.1) motif occurrences on hg38
  as BED records with '+'/'-' strand signs to orient ChIP-seq peaks or mutations.
when_to_use_negative:
- The features of interest already carry strand annotation — Asymmetron's orientation
  step and motif-derived strand references are only needed for unassigned features.
- The analysis targets palindromic motifs, which cannot be oriented relative to one
  another since their two orientations are indistinguishable.
- The goal is de novo motif discovery rather than strand assignment of features using
  known PWM loci.
edam_operation: http://edamontology.org/operation_0445
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0160
tools:
- name: Asymmetron
  role: Consumes the strand-annotated JASPAR/Unibind motif loci and the unannotated
    features; its orientation function performs the strand assignment based on overlap
    and closest center distance
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Implements Asymmetron's core BED-formatted file comparison functions used
    for interval overlap between motif loci and unannotated features
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: FIMO
  role: Scans JASPAR PWMs against the hg38 genome to produce the motif occurrence
    loci, using human-genome nucleotide frequencies as the background model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:a6a39a1fc0f9c70eac191eea3a5477f456ff5af22e311ecc4d0416ff53f6b629
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - FIMO
  - Pybedtools
  - Python
  edam_operation: http://edamontology.org/operation_0445
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# jaspar-motif-loci-handling

## Summary

Handling of JASPAR-derived transcription factor motif loci: scanning PWMs against a genome with FIMO, filtering matches by P-value, and importing strand-annotated motif occurrence BED files so their strand signs can be transferred to unannotated features via Asymmetron's orientation function.

## When to use

Use when strand asymmetry analysis needs strand-annotated transcription factor binding site loci, e.g. obtaining CTCF (MA0139.1) motif occurrences on hg38 as BED records with '+'/'-' strand signs to orient ChIP-seq peaks or mutations.

## When NOT to use

- The features of interest already carry strand annotation — Asymmetron's orientation step and motif-derived strand references are only needed for unassigned features.
- The analysis targets palindromic motifs, which cannot be oriented relative to one another since their two orientations are indistinguishable.
- The goal is de novo motif discovery rather than strand assignment of features using known PWM loci.

## Inputs

- JASPAR 2020 CORE vertebrates non-redundant PWM collection (position frequency matrices)
- Reference genome assembly (hg38) FASTA for FIMO motif scanning
- Unibind PWM motif maps (2019 release) as precomputed strand-annotated motif loci
- Asymmetron orientation function input: BED files of unannotated features (e.g. CTCF ChIP-seq peaks) paired with strand-annotated motif occurrence BED files (e.g. MA0139.1 CTCF motifs)

## Outputs

- Strand-annotated motif occurrence BED records (chromosome, start, end, PWM match score, strand sign '+'/'-')
- Per-PWM filtered match sets meeting the P-value < 10^-6 threshold
- Oriented feature BED files (e.g. ctcf_peaks_oriented.bed) with strand signs transferred from the nearest-center overlapping motif

## How to apply

Derive position frequency matrices (PWMs) from the JASPAR 2020 CORE vertebrates non-redundant collection, then scan them against the genome with FIMO using as background model the nucleotide frequencies across the human genome (hg38). Filter the FIMO output by requiring a minimum P-value <10^-6, and drop PWMs for which no matches meet this threshold (the paper retained 551 PWMs this way). Alternatively, use precomputed strand-annotated motif maps such as the Unibind PWM motif maps (2019 release). The resulting motif loci carry a strand sign ('+' or '-') and, formatted as BED intervals, serve as the strand reference input for Asymmetron's orientation function, which assigns each overlapping unannotated feature the sign of the motif instance with the closest center-to-center distance.

## Related tools

- **Asymmetron** (Consumes the strand-annotated JASPAR/Unibind motif loci and the unannotated features; its orientation function performs the strand assignment based on overlap and closest center distance) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Implements Asymmetron's core BED-formatted file comparison functions used for interval overlap between motif loci and unannotated features) — https://github.com/Ahituv-lab/Asymmetron
- **FIMO** (Scans JASPAR PWMs against the hg38 genome to produce the motif occurrence loci, using human-genome nucleotide frequencies as the background model)

## Examples

```
fimo --thresh 1e-6 MA0139.1.meme hg38.fa > ctcf_motifs_fimo.tsv
```

## Evaluation signals

- Every retained motif match satisfies the P-value < 10^-6 FIMO threshold; PWMs with no passing matches are excluded (paper kept 551 of the JASPAR PWMs).
- BED output is 6-column with a valid '+'/'-' strand field for every motif locus, and coordinates are on the hg38 build.
- For the orientation step, each assigned feature's strand equals the strand sign of the overlapping motif whose center |(s_i+e_i)/2 - (s'_i+e'_i)/2| is minimal among all overlapping instances.
- Reported counts of peaks assigned vs. left unassigned are consistent: features with no overlapping annotated motif remain unassigned.
- Pipeline reproducibility: commands and files are available on the Asymmetron GitHub page, so outputs can be diffed against the published workflow.

## Limitations

- Strand assignment only succeeds for unannotated features that overlap an annotated motif; features with no overlap are left unassigned.
- The paper's threshold of P < 10^-6 for FIMO matches is stringent and can exclude weaker but biologically real motif instances.
- Asymmetron requires DNA features to be provided with strand annotation for its asymmetry functions; orientation of third-party features depends entirely on the quality and completeness of the reference motif loci.
- The README conda setup pins an older environment (pybedtools, python=3.5, seaborn, numpy), which may require adaptation on modern systems.

## Evidence

- [methods] Position frequency matrices (PMWs) of transcription factors were derived from JASPAR (release 2020) for the non-redundant CORE collection: "Position frequency matrices (PMWs) of transcription factors were derived from JASPAR (release 2020) for the non-redundant CORE collection"
- [methods] FIMO scan with hg38 background and P-value threshold: "motif scanning was performed with FIMO using as background model the nucleotide frequencies across the human genome"
- [methods] P-value cutoff for retained matches: "requiring a minimum P-value <10−6 (29)."
- [methods] Orientation of unannotated features from overlapping annotated ones: "A fourth function performs the strand assignment of an unassigned feature based on another overlapping feature"
- [readme] Asymmetron can orient third-party features via other genomic elements: [evidence span withheld — anchor status: external_doc]
