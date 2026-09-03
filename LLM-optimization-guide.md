* [What is KV cache?](#what-is-kv-cache)
* [What is PagedAttention (for KV Cache)](#what-is-pagedattention-for-kv-cache)
* [What is continuous batching?](#what-is-continuous-batching)
* [Quantization - The Core Compression Technique](#quantization---the-core-compression-technique)
* [Speculative decoding](#speculative-decoding)
* [What are the different hardware and infrastructure optimizations for LLM inference?](#what-are-the-different-hardware-and-infrastructure-optimizations-for-llm-inference)

### What is KV cache?
* It is used to speed up the autoregressive decoding phase of an LLM for text generation by caching internally computed matrices in its attention layers to reuse them later for predicting subsequent tokens.
* When the model receives an input prompt, during prefill phase, each of its attention layers compute their Key and value matrices internally, and cache them in GPU's high-speed memory.
* During autoregression, when a new token is generated and fed back as part of the input sequence to the model, the Key and Value matrics of previous prompt tokens are reused from KV cache instead of being recomputed again, and only the key and value vectors only for the new token are computed and appended to the existing cache for generating subsequent ones and so on.
* **Performance improvement -** This makes each subsequent step a constant-time operation rather than re-processing the entire sequence history, which is essential for handling large contexts
    * Note that the initial "prefill" stage (processing the prompt) is slower because the cache must be built from scratch, while subsequent token generation is faster

* **Memory usage impact -**
    * It trades off memory for latency and computation.
    * Because the cache grows with the sequence length and batch size, it often consumes the majority of GPU memory during inference. The memory usage can be calculated as:

`Memory = 2 * precision * layers * dimension * sequence length  * batch size`

* where, 2 =  K and V matrices
* precision = bytes per parameter (E.g., 2 for fp16 inference)
* layers = No. of layers in the model
* dimension = dimension of embeddings
* sequence length = length we want to generate at the end, including the prompt tokens

* For large models, the cache can take up significantly more memory than the model weights themselves. E.g., For model OPT 30B,
  * precision = 2 (FP16)
  * layers = 48
  * dimensions = 7168
  * sequence length = 1024 (Suppose we set max sequence length as 1024)
  * batch size = 128
  * KV cache memory usage = 180GB
  * whereas, model memory usage = 2 * 30B = 60GB (assuming FP16 precision)
  
  
Reference -
* [KV cache: Memory usage in transformers](https://youtu.be/80bIUggRJf4?si=94HRT3CFXODeaIta)
* [What is a KV cache, and why does it make LLM inference faster? - Sebastian Raschka](https://sebastianraschka.com/faq/docs/kv-cache.html)


### What is PagedAttention (for KV Cache)
* **Problem with old approach:** Previous methods reserved one large, contiguous memory block per request, sized for the maximum possible context length — resulting in 60–80% wasted memory, since most requests don't use their full context.
* PagedAttention's solution: Splits the KV cache into small, fixed-size blocks that can be placed non-contiguously anywhere in GPU memory (similar to virtual memory paging in OS design).
**Result:** Significantly more concurrent requests can be served on the same GPU hardware.

* Find more details on how it works [here](https://github.com/Taaniya/deeplearning-ai-course-fast-and-efficient-llm-inference-with-vllms/blob/main/L5_Serving_LLMs_Efficiently_with_vLLM_Part_1.md#5-pagedattention)


### What is continuous batching?
* Dynamically batching incoming requests as they arrive/finish across different users, rather than static batching
* Replaces idle GPU time with active computing, maximizing hardware efficiency.
* With batching, the GPU reads the model's weights once and use them for many users simultaneously.
* Unlike static batching, where the entire batch has to wait for the longest request to finish, with dynamic batching, once a request finishes, a new request immediately takes the slot in the batch.

* Find more details on how it works [here](https://github.com/Taaniya/deeplearning-ai-course-fast-and-efficient-llm-inference-with-vllms/blob/main/L5_Serving_LLMs_Efficiently_with_vLLM_Part_1.md#3-continuous-batching)
* [Continuous batching - HuggingFace (Nov, 2025)](https://huggingface.co/blog/continuous_batching)


### Quantization - The Core Compression Technique

**What Is Quantization?**

- Quantization reduces the number of bits used to store model weights.
- Analogy: Instead of storing pi as `3.14159...` (many bits), store it as `3.14` (fewer bits).
- Most LLMs today are released in **BF16** (Brain Float 16 — 16 bits per number).
- Quantization converts numbers into lower bit formats: **FP8, INT8, or INT4**.

**Motivation: The gap in current situation and real problems with LLMs -** 
* Model sizes of LLMs have grown since past few years - from the original Transformer in 2017 at 50 million parameters to reaching hundreds of billions of parameters which GPU memory cannot yet handle completely.
* This limitation leads to following problems -
   * **GPU & Infrastructure Cost:**	Bigger models require more hardware accelerators, often spread across multiple nodes — expensive to operate.
   * **User Experience Tradeoffs:**	More parameters can mean slower responses, lower throughput, and less room for long context in the KV cache.
   * **Energy & Carbon Footprint:**	Every extra GPU draws power; at scale, this adds up to a real environmental cost.
   * **Risk of Model Obsolescence:**	Risk of investing in infrastructure for a model that gets superseded quickly.



**Numeric Formats Explained**

| Format | Full Name | Notes |
|---|---|---|
| **FP32** | Floating-Point 32 | Huge range, fine-grained precision |
| **BF16** | Brain Floating-Point 16 | Same range as FP32 but less precision; developed by Google; more stable for large models |
| **FP16** | Floating-Point 16 | Narrower range than BF16 |
| **INT8** | Integer 8 | Whole numbers, smaller range, larger gaps between values |
| **INT4** | Integer 4 | Most aggressive; largest gaps between representable values |

- As you move from FP32 → BF16 → FP16 → INT8 → INT4: **range shrinks, gaps between values grow** — you trade precision for size.

**What Gets Quantized?**

Quantization specifically targets the **linear layers** inside transformer blocks, because:
- Most forward pass time is spent inside linear layers.
- The main matrix multiplications happen there.
- The bulk of the model's weights live there.

**Note:** The embedding layer and LM head are typically excluded from quantization to preserve accuracy.

Two things inside a linear layer can be quantized:
1. **Weights** — the model's learned parameters.
2. **Input Activations** — intermediate tensors computed during a forward pass that flow through the linear layers (e.g., the tensor multiplied by weights to produce Q, K, V, or the weighted sum that flows into the O projection).

**What are linear layers in transformer model?**

Both the Self-Attention block and FFN are built from **linear layers**.

- A **linear layer** is a matrix multiplication: takes an input vector → multiplies by a weight matrix → produces an output vector.
- Linear layers are where **almost all model parameters live** and where **almost all computation happens**.


**Self-Attention Block — 4 Linear Layers**

| Projection | Name | Meaning |
|------------|------|---------|
| Q | Query | "What do I want to know from the context?" |
| K | Key | "Here's my label and the kind of information I contain" |
| V | Value | "If my label matches, here is my actual content" |
| O | Output | Final output projection of the attention block |

**Feed-Forward Network — 3 Linear Layers**

- **Gate projection**
- **Up projection**
- **Down projection**

> These weight matrices are **learned once during training** and remain fixed during inference.

   
**Related courses -**
* [Quantization in depth with Pytorch, by DeepLearning.ai (Huggingface)](https://learn.deeplearning.ai/courses/quantization-in-depth/information)
* [Fast and efficient LLM inference with vLLM, by DeepLearning.ai](https://www.deeplearning.ai/courses/fast-and-efficient-llm-inference-with-vllm)
   * [Course notes](https://github.com/Taaniya/deeplearning-ai-course-fast-and-efficient-llm-inference-with-vllms)

Documentation & Tutorials -
* [Quantization - Pytorch](https://docs.pytorch.org/docs/stable/quantization.html)
* [Git repo - Torchao: PyTorch-Native Training-to-Serving Model Optimization](https://github.com/pytorch/ao)

### Speculative decoding
In speculative decoding, the logits for the small (draft) and large (target) models come from **separate forward passes** of each respective model over the current context sequence.

**Small (Draft) Model Logits:** 
* Generated sequentially (or via a parallel drafting head/sub-network) step-by-step as the small model proposes K candidate tokens.
* At each draft step t, the small model runs a fast forward pass on the existing prefix, outputs raw pre-softmax scores (logits) for that position, samples a token $x_{t}$, and feeds it back in to predict the next guess.
* The corresponding draft probabilities $p_{draft}$ are calculated from these saved logits.

**Target Model Logits:**
   * Generated in a single, parallel forward pass after the draft phase.
   * The entire sequence (the original context plus all K speculated draft tokens) is fed into the large target model all at once.
   * Utilizing causal masking, the target model evaluates all positions simultaneously and outputs a full set of logits corresponding to every token position in the speculated block in one go.

The rejection sampling step then converts both sets of logits into probability distributions $p_{target}$ and $p_{draft}$ via softmax (and temperature scaling if used) to compare them position-by-position.

References -
* https://developer.nvidia.com/blog/an-introduction-to-speculative-decoding-for-reducing-latency-in-ai-inference/
* [Watch - Speculative decoding: When 2 LLMs are faster than 1](https://youtu.be/S-8yr_RibJ4?si=-TpUdM6S_qdYBvk4)
* https://aryagm.com/blog/speculative-decoding-the-art-of-being-good-enough/

### What are the different hardware and infrastructure optimizations for LLM inference?

* **Tensor Parallelism (TP):** Splits individual matrix multiplications across multiple GPUs (e.g., Megatron-LM). Essential for models too large to fit on a single GPU's VRAM.
* **Pipeline Parallelism (PP):** Splits different layers of the model sequentially across multiple GPUs.
* **Managed API Routers:** Using LLM gateways to route simpler queries to smaller, cheaper models (like GPT-4o-mini or Llama-3-8B) and reserving complex queries for flagship models.


Other references and reading sources:
* [Mastering LLM Techniques: Inference Optimization - Nvidia (Nov 2023)](https://developer.nvidia.com/blog/mastering-llm-techniques-inference-optimization/)
