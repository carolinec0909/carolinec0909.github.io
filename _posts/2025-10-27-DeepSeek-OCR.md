---
title: "Beyond Text: How Optical Compression Unlocks 10× Larger Contexts for LLMs"
date: 2025-10-27
---

## 1. The Context Window Bottleneck

Large language models (LLMs) have become incredibly capable, but their biggest limitation hasn’t changed: **the context window**.  
Even with cutting-edge models boasting hundreds of thousands of tokens, we still hit hard limits on memory, efficiency, and cost.  

Scaling up the context window linearly is *brutal*. Each doubling of tokens leads to quadratic growth in attention computation and memory usage.  
Beyond a few hundred thousand tokens, even A100-class GPUs start sweating.

But what if we could **scale the context window 10×** without actually increasing sequence length?

---

## 2. The Breakthrough: Optical Compression via DeepEncoder
![Deep Encoder Architecture]({{ site.baseurl }}/assets/images/2025-10-27-DeepSeek-OCR-images/model-architecture.png)

DeepSeek-OCR (2025) proposes a radical rethink of how we represent input.  
Instead of processing text tokens directly, it **renders text as an image**, then compresses it into a small number of vision tokens, achieving 10× to 20× effective context extension through compression.

### How It Works

1. **Render text → image**  
   Each page or document is rendered into an image, preserving formatting, layout, and embedded visuals.

2. **Encode with DeepEncoder**  
   DeepEncoder combines:
   - **SAM** for local perception (fine-grained segmentation)
   - **16× convolutional compressor** for optical downsampling
   - **CLIP-based encoder** for global semantic alignment

3. **Output:** 64 – 400 vision tokens per page (vs. thousands of text tokens)

That **16× convolutional downsampling** (e.g., 4,096 image patches → 256 compressed tokens) is the core enabler.  
It allows a model to “see” far more information in the same context length, turning a 10k-token document into just a few hundred high-information tokens.

---

## 3. The Paradigm Shift: Why LLM Inputs Should Be Images

Andrej Karpathy summarized this shift neatly:

> “Maybe it makes more sense that all inputs to LLMs should only ever be images.”  
> (See his tweet [here](https://x.com/karpathy/status/1980397031542989305).)

It’s a bold statement, but increasingly, it makes sense.

### Why Images Win

- **Compression Efficiency:** Visual compression can pack 10× more information than tokenized text at similar fidelity.  
- **Multimodal by default:** Captures formatting, color, icons, equations, and diagrams all lost in plain text.  
- **Bidirectional attention becomes natural:** Unlike text sequences, images are *fixed holistic inputs*. We can use full bidirectional attention without causal masking.  
- **No tokenizer.** The tokenizer is a relic. It is full of Unicode quirks, byte encodings, and brittle mappings that make semantically identical text look totally different to the model.  
- **End-to-end trainable:** Vision-first models can process mixed modalities natively (text, layout, figures) without artificial boundaries.

In this view, **OCR isn’t a hack but the future**.  
Text → image → vision tokens could be the universal input pathway, even for “pure text” tasks.

---

## 4. Deep Dive: Bidirectional vs. Autoregressive Attention

Karpathy’s commentary on the DeepSeek-OCR paper highlights a deep implication of this approach:

> “Input can now be processed with bidirectional attention easily and as default, not autoregressive attention—a lot more powerful.”

Let’s unpack that.

### Autoregressive (Causal) Attention

- Used in GPT-style decoder-only LLMs  
- Each token only attends to *previous* tokens (future ones are masked)  
- Works well for **generation**, but poorly for **understanding**  
- Early tokens can’t access clarifying info from later tokens → shallower comprehension

### Bidirectional Attention

- Every token attends to every other token  
- Great for **understanding and reasoning**, since it captures the full context symmetrically  
- Used in encoders like BERT, but expensive (`O(n²)` memory)

### Why DeepSeek-OCR Enables Bidirectional by Default

- **Compression:** Shrinks sequences 10–20× (e.g., from 4k tokens to 256 vision tokens).  
  → Bidirectional attention now affordable even on large documents.  
- **Fixed Inputs:** Since images aren’t generated step-by-step, the model can process the entire compressed input *at once* with no need for causal masking.  
- **Hybrid Pipeline:**  
  *DeepEncoder → Vision Tokens → LLM*  
  The LLM can apply bidirectional attention for understanding, then switch to autoregressive for generation.  
- **No tokenizer barriers:** The model receives a dense, continuous input space (pixels → embeddings), enabling smoother gradient flow and more general learning.

---

## 5. Why It’s “A Lot More Powerful”

### Better Understanding

Bidirectional attention over compressed vision tokens yields deeper, more holistic comprehension.  
DeepSeek-OCR achieves **97% accuracy** on OCR benchmarks with **7–20× fewer tokens**, outperforming baselines like GOT-OCR2.0.

### Extreme Context Scaling

By compressing entire documents into a few hundred vision tokens, models can process **massive contexts** without exploding memory requirements.

### Multimodal Natively

Layout, tables, charts, and emphasis (bold, color, structure) are preserved.  
No need for handcrafted markup.

### Practical Gains

- Faster inference (~2500 tokens/s on A100 with vLLM)  
- Efficient fine-tuning on user-specific document styles  
- Compatible with existing LLM architectures

---

## 6. The Bigger Picture

This optical approach hints at a new paradigm for LLMs:

- **Inputs → pixels**  
- **Compression → vision embeddings**  
- **Understanding → bidirectional**  
- **Generation → autoregressive**

It unifies text and vision under a single framework, replacing messy tokenizers with clean, learned representations.

As Karpathy put it: maybe the tokenizer really *must go.*

---

## 7. Summary

| Challenge           | Traditional LLMs             | DeepSeek-OCR / DeepEncoder             |
|---------------------|------------------------------|----------------------------------------|
| **Context window**  | Hard to scale, quadratic cost| 10×+ expansion via optical compression|
| **Input modality**  | Text-only, tokenized         | Rendered image (text + visuals)       |
| **Attention**       | Autoregressive (limited)     | Bidirectional (full context)          |
| **Compression**     | None                         | 16× convolutional downsampling        |
| **Tokenizer**       | Required, brittle            | Eliminated                            |
| **Efficiency**      | Limited by token length      | Compresses to 64–400 vision tokens/page|

---

## 8. Closing Thoughts

DeepSeek-OCR and its DeepEncoder pipeline point toward a post-tokenizer era,   
where **all inputs are visual**, **context is near-infinite**, and **bidirectional attention** becomes the default superpower of understanding.

In this world, LLMs don’t just read text. They *see* it.

---

**📄 Source:** [DeepSeek-OCR: Scaling Vision-Language Models for Document Understanding (arXiv 2510.18234v1)](https://arxiv.org/pdf/2510.18234v1)  
**🐦 Karpathy Tweet:** [https://x.com/karpathy/status/1980397031542989305](https://x.com/karpathy/status/1980397031542989305)
