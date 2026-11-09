---
name: variable-length-sequence-adaptive-pooling
description: Use when converting enhancer/DNA FASTA sequences of differing lengths
  into fixed-dimensional dense numeric features for a deep learning model, using pre-trained
  dna2vec k-mer vectors plus adaptive pooling.
when_to_use_negative:
- The input is already a numeric feature table or fixed-length one-hot encoding —
  no dna2vec embedding or pooling is needed.
- The goal is motif-based (TFBS/PWM) feature extraction from HOCOMOCO v11 — use motif_find.py
  instead of dna2vec_code.py.
- Sequences are not DNA k-mer encodable (e.g. protein or RNA modification data) —
  the dna2vec k=3 lookups would be invalid.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0749
tools:
- name: dna2vec
  role: Source of the pre-trained DNA vectors providing 100-dimensional embeddings
    for each 3-mer looked up during encoding
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: Enhancer-MDLF
  role: Downstream multi-input deep learning framework that consumes the 10,000-dimensional
    pooled dna2vec features
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: numpy
  role: Array operations used to concatenate k-mer embeddings and perform adaptive
    pooling
- name: dna2vec_code.py
  role: Repository script implementing the extraction; invoked with --input_file,
    --cell_line, and --set arguments
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:282efe8783de5911384f3dbd4eb296865161e2338975874087b47f88d317ef50
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
  repo_url: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
  tools:
  - Enhancer-MDLF
  - Python
  - dna2vec
  - dna2vec_code.py
  - numpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# variable-length-sequence-adaptive-pooling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes variable-length DNA enhancer sequences into fixed-size feature vectors by concatenating pre-trained dna2vec k-mer embeddings (k=3, 100-dim per k-mer, stride 1) and applying adaptive pooling to flatten each sequence to the 10,000-dimensional input expected by Enhancer-MDLF.

## When to use

Use when converting enhancer/DNA FASTA sequences of differing lengths into fixed-dimensional dense numeric features for a deep learning model, using pre-trained dna2vec k-mer vectors plus adaptive pooling.

## When NOT to use

- The input is already a numeric feature table or fixed-length one-hot encoding — no dna2vec embedding or pooling is needed.
- The goal is motif-based (TFBS/PWM) feature extraction from HOCOMOCO v11 — use motif_find.py instead of dna2vec_code.py.
- Sequences are not DNA k-mer encodable (e.g. protein or RNA modification data) — the dna2vec k=3 lookups would be invalid.

## Inputs

- Enhancer FASTA sequences per cell line and set (e.g. 'data/train/NHLF.fasta', 'data/test/NHLF.fasta')
- Pre-trained dna2vec DNA vectors from the repository's 'dna2vec' folder (k-mer embeddings, 100-dim per 3-mer)

## Outputs

- Per-sequence fixed 10,000-dimensional dna2vec feature vectors saved in the 'feature' directory, produced separately for train and test sets

## How to apply

Load enhancer FASTA sequences (e.g. 'data/train/NHLF.fasta') following the Enhancer-MDLF data naming convention, and load the pre-trained DNA vectors shipped in the 'dna2vec' folder (vectors from Ng's dna2vec, arXiv:1701.06279). For each sequence, slide a k-mer window with k=3 and stride 1, look up each k-mer's pre-trained 100-dimensional embedding, and concatenate the embeddings along the sequence so variable-length sequences yield variable-length embedding stacks. Apply adaptive pooling (implemented with numpy array operations, as in dna2vec_code.py) over the concatenated per-k-mer embedding sequence to collapse it to a single fixed 10,000-dimensional feature vector per enhancer. Run the extraction separately for training and test sets by passing --input_file, --cell_line, and --set arguments, and save the resulting feature files in the 'feature' directory for consumption by the Enhancer-MDLF multi-input network.

## Related tools

- **dna2vec** (Source of the pre-trained DNA vectors providing 100-dimensional embeddings for each 3-mer looked up during encoding) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **Enhancer-MDLF** (Downstream multi-input deep learning framework that consumes the 10,000-dimensional pooled dna2vec features) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **numpy** (Array operations used to concatenate k-mer embeddings and perform adaptive pooling)
- **dna2vec_code.py** (Repository script implementing the extraction; invoked with --input_file, --cell_line, and --set arguments) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF

## Examples

```
python dna2vec_code.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train
```

## Evaluation signals

- Each input enhancer sequence yields exactly one output feature vector of fixed 10,000-dimensional input size, matching the dimension expected by the Enhancer-MDLF network.
- Output feature files exist in the 'feature' directory for both train and test sets after running dna2vec_code.py.
- The embedding lookup uses the pre-trained dna2vec vectors shipped in the 'dna2vec' folder with k=3 and stride 1, so per-sequence pre-pooling length equals sequence length minus k plus 1 stacked 100-dim embeddings.
- Train and test extractions are run independently with matching --cell_line and --set arguments so feature rows align with the label files (e.g. NHLF_y_train.txt / NHLF_y_test.txt).
- Features produced this way let main.py (e.g. 'python main.py --cell_line NHLF') run and yield prediction results comparable to the paper's reported Enhancer-MDLF performance.

## Limitations

- The fixed 10,000-dimensional output requires adaptive pooling, which discards positional structure of the variable-length k-mer embedding sequence; sequences longer than the pooled size lose fine-grained information.
- The method depends on the specific pre-trained dna2vec vectors distributed in the repo; other k-mer sizes or embedding dimensionalities would change the feature shape and break compatibility with the provided network.
- Environment constraints (Python 3.8, tensorflow 2.2.0, pip 23.3.1 per requirements.txt) must be respected; the README recommends a dedicated conda env ('conda create -n test python=3.8') to avoid package conflicts.

## Evidence

- [methods] The dna2vec encoding uses k=3-mer embeddings concatenated with stride 1 over each sequence, then adaptive pooling flattens the variable-length features to a fixed 10,000-dimensional input for the deep learning model.: [evidence span withheld — anchor status: not_found]
