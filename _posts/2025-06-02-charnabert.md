---
layout: post
title: "ChaRNABERT: A character-level RNA foundation model"
subtitle: "Accepted at ICLR 2025 — Learning Meaningful Representations of Life Workshop"
background: '/img/posts/charnabert/charnabert_background.jpg'
---

Protein language models like ESM have reshaped computational biology. RNA has been harder: the same four letters can fold into very different structures, and most existing RNA models rely on hand-picked tokenization schemes—single nucleotides, codons, or fixed k-mers—that bake in assumptions about what matters in a sequence.

In [**Character-level Tokenizations as Powerful Inductive Biases for RNA Foundational Models**](https://arxiv.org/abs/2411.11808), we introduce **ChaRNABERT**, a suite of sample- and parameter-efficient RNA foundation models built around a simple idea: let the model *learn* how to tokenize RNA, starting from individual nucleotides.

The work was presented as a poster at the [**Learning Meaningful Representations of Life (LMRL) Workshop @ ICLR 2025**](https://iclr.cc/virtual/2025/35942).

## The problem

RNA sits at the center of gene regulation, catalysis, and an expanding class of therapeutics—from mRNA vaccines to aptamers and CRISPR guides. Yet compared to proteins, general-purpose RNA models still lag behind: many are task-specific, trained on narrow sequence types, or require bespoke preprocessing for each downstream application.

A recurring bottleneck is **tokenization**. Choosing nucleotides, codons, or k-mers forces the model to see RNA through a fixed lens. Motifs that look similar at one granularity can carry completely different biological meaning at another.

## Our approach

ChaRNABERT combines two components:

1. **Gradient-Based Subsequence Tokenization (GBST)** — a learnable, character-level tokenizer that dynamically selects the most informative subsequence block at each nucleotide position, with sliding offsets analogous to open reading frames. Unlike the original GBST formulation, we remove downsampling to preserve single-nucleotide resolution, which matters for many downstream tasks.

2. **A modern BERT encoder** — bidirectional self-attention with SwiGLU activations, rotary positional embeddings (RoPE), query-key normalization, and Flash Attention 2, supporting context windows up to 8,190 nucleotides.

Models are pretrained on ~31M non-coding sequences from RNAcentral and, for the extended variant, an additional ~31M coding sequences from RefSeq. We use UL2-style masking (short-span, extreme-span, and retrieval-augmented denoising) alongside standard masked language modeling. Training runs on EuroHPC resources (Leonardo Booster, MareNostrum 5) using DeepSpeed and ZeRO.

![ChaRNABERT architecture: GBST tokenization feeding a BERT encoder, trained on RNAcentral and RefSeq](/assets/2025-06-02-charnabert/architecture.png){: width="700" }

We trained a scaling suite from **8M to 650M parameters**, studying how model size, data volume, and tokenization strategy interact— including RNA-specific scaling laws that suggest optimal compute allocation differs from what we see in natural language.

## Results

We evaluated ChaRNABERT on the [**BEACON benchmark**](https://arxiv.org/abs/2406.10391) (13 tasks spanning structure, function, and engineering) and on two additional tasks we designed: **RNA–RNA-binding protein interaction** prediction (from eCLIP data) and **aptamer–protein interaction** prediction (from the University of Texas Aptamer Database).

The headline: **ChaRNABERT-8M—a model with roughly 80× fewer parameters than RiNALMo-650M—matches or beats strong baselines on most BEACON tasks**, and clearly outperforms RiNALMo on RNA–RBP classification and aptamer–protein binding prediction.

![ChaRNABERT performance relative to BEACON baselines across downstream tasks](/assets/2025-06-02-charnabert/beacon_results.png){: width="700" }

Some highlights:

- **Structural tasks**: CRB-50M leads on distance map prediction; competitive on contact maps and structural score imputation.
- **Functional tasks**: CRB-8M tops alternative polyadenylation isoform prediction; strong on splice-adjacent and ncRNA tasks.
- **Engineering tasks**: Best-in-class on RNA vaccine degradation prediction; strong CRISPR on-target performance.
- **Aptamer–protein binding**: CRB-50M reaches F1 ≈ 0.79, surpassing RiNALMo (0.74) and classical CNN/LSTM baselines.

![Aptamer–protein interaction prediction across ChaRNABERT model sizes](/assets/2025-06-02-charnabert/aptamer_results.png){: width="700" }

The learnable tokenizer is the key. GBST consistently outperforms plain nucleotide embeddings at the same parameter count, and the gains hold across context lengths and dataset sizes. For many tasks, scaling from 8M to 50M parameters yields modest improvements—suggesting that *how* you represent the sequence matters as much as *how large* the model is.

## Links

- **Paper**: [arXiv:2411.11808](https://arxiv.org/abs/2411.11808)
- **OpenReview**: [ICLR 2025 AI4NA submission](https://openreview.net/forum?id=7MU9Rl5xZE)
- **Workshop page**: [LMRL @ ICLR 2025](https://iclr.cc/virtual/2025/35942)

Weights and inference code for **ChaRNABERT-8M** will be released for academic use; larger checkpoints are available on request.

## Team

Adrian Morales-Pastor, Raquel Vázquez-Reza, Miłosz Wieczór, Clàudia Valverde, Manel Gil-Sorribes, Bertran Miquel-Oliver, Álvaro Ciudad, and Alexis Molina.
