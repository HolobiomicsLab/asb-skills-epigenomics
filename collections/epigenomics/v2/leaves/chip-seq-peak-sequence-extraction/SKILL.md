---
name: chip-seq-peak-sequence-extraction
description: Use when preparing summit-centered, score-annotated FASTA inputs for
  MEPP positional motif profiling from MACS2 narrowpeak calls (e.g. ENCODE K562 GATA1
  ChIP-seq or mESC Nanog ChIP-seq/ChIP-nexus peaks).
when_to_use_negative:
- Input is a whole peak interval or a BED file without summit (narrowPeak) coordinates
  — MEPP positional profiles require a well-defined anchor point such as the MACS2
  summit, not arbitrary region boundaries
- Sequences lack a score and would have to be binarized — MEPP's design avoids arbitrary
  threshold selection and needs a score-based continuum of biological interest
- De novo motif discovery is the goal — MEPP 'currently functions with a fixed motif
  library' and its motif-learning module, not interval extraction, would be the wrong
  frame here
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0160
tools:
- name: MEPP
  role: Consumes the summit-centered scored FASTA and computes the position-dependent
    motif enrichment profile relative to the summit anchor
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installation channel for MEPP (pip install mepp or pip install git+https://github.com/npdeloss/mepp@main)
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:70e07ef47b1aead6ef3518f04f2b2d4ba30622211fb4e363592b9106e8f8d8cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  - pip
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# chip-seq-peak-sequence-extraction

## Summary

Extracts ±200 bp genomic sequences centered on MACS2 peak summits from ChIP-seq or ChIP-nexus narrowpeak calls and attaches each peak's MACS2 log2 fold-change as a sequence-level score, producing the scored FASTA continuum that MEPP requires for positional motif enrichment profiling.

## When to use

Use when preparing summit-centered, score-annotated FASTA inputs for MEPP positional motif profiling from MACS2 narrowpeak calls (e.g. ENCODE K562 GATA1 ChIP-seq or mESC Nanog ChIP-seq/ChIP-nexus peaks).

## When NOT to use

- Input is a whole peak interval or a BED file without summit (narrowPeak) coordinates — MEPP positional profiles require a well-defined anchor point such as the MACS2 summit, not arbitrary region boundaries
- Sequences lack a score and would have to be binarized — MEPP's design avoids arbitrary threshold selection and needs a score-based continuum of biological interest
- De novo motif discovery is the goal — MEPP 'currently functions with a fixed motif library' and its motif-learning module, not interval extraction, would be the wrong frame here

## Inputs

- MACS2 narrowpeak peak calls (ChIP-seq or ChIP-nexus), e.g. ENCODE K562 GATA1 ChIP-seq peaks (~28,000 for Nanog ChIP-seq, ~39,000 for Nanog ChIP-nexus)
- Reference genome assembly for sequence retrieval
- Motif position weight matrix file in JASPAR format (or HOMER-format motifs bundled in the mepp repository data/ directory)
- Peak-level MACS2 log2 fold-change scores (or other narrowpeak signal scores) as the biological-continuum score

## Outputs

- Scored FASTA of summit-centered ±200 bp sequences with headers of the form ">sequence_name sequence_score"
- MEPP positional enrichment profile plots (PNG/SVG) relative to the peak-summit anchor point
- Clustering table of positional correlation p-values (multiple-testing adjusted)

## How to apply

Retrieve peak calls (e.g. ENCODE K562 GATA1 ChIP-seq and control MACS2 narrowpeaks) and extract ±200 bp of genomic sequence symmetric around each MACS2 peak summit, using the summit — not the whole peak interval — as the positional anchor. Write each sequence to a FASTA whose header encodes the parent peak's MACS2 log2 fold change in the form ">sequence_name sequence_score", since MEPP consumes the score as a continuum of biological interest rather than requiring an arbitrary threshold. For ChIP-seq-derived sequences, plan to scan both motif orientations (+/-) because ChIP-seq does not resolve motif strand; strand-resolved ChIP-nexus data can be analyzed in a single orientation. Feed the scored FASTA plus a motif matrix file (e.g. GATA1 or Nanog PWMs in JASPAR format) to MEPP, optionally tuning --dgt to reject repeat-laden sequences and --pval/--bg for MOODS match thresholds. Validate by inspecting the resulting positional profile: a correct run shows motif enrichment centered on the summit anchor.

## Related tools

- **MEPP** (Consumes the summit-centered scored FASTA and computes the position-dependent motif enrichment profile relative to the summit anchor) — https://github.com/npdeloss/mepp
- **pip** (Installation channel for MEPP (pip install mepp or pip install git+https://github.com/npdeloss/mepp@main)) — https://github.com/npdeloss/mepp

## Examples

```
mepp --fa k562_gata1_summit_seqs.scored.fa --motifs jaspar_motifs.txt --out mepp_out --orientations +/-
```

## Evaluation signals

- FASTA headers conform exactly to MEPP's expected format ">sequence_name sequence_score" so MACS2 log2FC is parsed as the score
- Each extracted sequence is symmetric ±200 bp around the MACS2 peak summit, matching the source-card extraction window
- For GATA1 K562 reproduction, gata1_mepp_profile.png exists and shows centrally positioned GATA1 motif enrichment around the summit, qualitatively matching published Figure 2 (less sharp than TSS-centered profiles)
- For Nanog comparison, the ChIP-nexus profile shows a sharp central peak with flanking Oct4-Sox2-TCF-Nanog twin peaks, while the ChIP-seq profile is broader and lower-amplitude (Figure 5)
- Low-complexity/repetitive sequences are filtered via the --dgt degenerate-base percentage before profiling

## Limitations

- ChIP-seq does not resolve motif strand, so both orientations (+/-) must be scanned and profiles are broader and lower-amplitude than strand-resolved ChIP-nexus profiles
- Positional profiles derived from ChIP-seq peak summits are intrinsically less sharp than single-nucleotide-resolution TSS-centered analyses
- MEPP currently functions only with a fixed motif library; no de novo motif recognition/assembly
- GPU memory can be exhausted by permutation testing (--perms); --gjobs must be set carefully or --nogpu used to avoid jobs crowding each other out of GPU memory

## Evidence

- [readme] MEPP requires scored FASTA input with score embedded in the sequence header: [evidence span withheld — anchor status: external_doc]
- [abstract] MEPP produces positional enrichment profiles relative to anchor points, motivating summit-centered extraction: "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs)"
- [intro] Position of motifs relative to biologically meaningful features is the core quantity being profiled: "MEPP identifies sequence motifs enriched at positions relative to biologically meaningful features"
- [intro] Score-based design avoids arbitrary thresholds, so peak log2FC should be carried through extraction: "The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias."
