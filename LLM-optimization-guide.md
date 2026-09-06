* [What is KV cache?](#what-is-kv-cache)
* [What is PagedAttention (for KV Cache)](#what-is-pagedattention-for-kv-cache)
* [What is continuous batching?](#what-is-continuous-batching)
* [Quantization - The Core Compression Technique](#quantization---the-core-compression-technique)
* [Speculative decoding](#speculative-decoding)
* [What are the different hardware and infrastructure optimizations for LLM inference?](#what-are-the-different-hardware-and-infrastructure-optimizations-for-llm-inference)

### How does LLM inference work?
* LLM inference is the process of using a trained AI model to generate text or answers based on an input prompt.
* It works in two main stages:
   * **The Prefill Phase:** The model reads your entire input prompt all at once. It processes all input tokens (word pieces) in parallel and builds a memory bank called the KV cache (Key-Value cache) to remember the context.

   * **The Decoding Phase:** The model generates the output text autoregressively—meaning it creates one token at a time. Each new token depends on the previous ones. Because it must load the entire model weights from memory for every single word generated, this phase is slow and memory-heavy.

### What are the techniques to optimize inference in production?
* To lower latency (speed up responses) and reduce cost (save computing power), production systems use several key strategies.
* To run LLMs in production efficiently, optimization strategies target either reducing memory footprint or maximizing hardware utilization.


1. Serving and Architectural Optimizations
* PagedAttention:
   * Manages GPU memory for the KV cache like an operating system manages computer RAM.
   * Eliminates memory fragmentation and allows many more concurrent users without running out of space.

2. Continuous Batching (Dynamic Batching):
   * Groups incoming requests from different users together on the fly.
   * Replaces idle GPU time with active computing, maximizing hardware efficiency.
   
3. Prompt Caching:
   * Saves the KV cache of frequently used text (like long system instructions or codebases).
   * Stops the GPU from re-reading the same text for every new user request.

4. Speculative Decoding:
   * Pairs a small, fast "draft" model with a large, slow "target" model.
   * The small model guesses the next few words quickly, and the big model checks them all in one single step.

5. KV Cache Quantization:
   * Downsamples the stored Key-Value vectors from FP16 to INT8 or INT4.
   * This allows long-context applications to fit into GPU memory without triggering Out-Of-Memory (OOM) errors.

6. Optimizing the attention mechanism:
   * Multi-query attention (2019):
      * Multi-query attention (MQA) uses many query heads and a single key-value head; i.e., key and value vectors are shared among the multiple attention heads, while the query vector is still projected multiple times as before, as in Multi-head attention (MHA).
      * While the amount of computation done in MQA is identical to MHA, the amount of data (keys, values) read from memory is a fraction of before.
      * When bound by memory bandwidth, this enables better compute utilization.
      * It also reduces the size of the KV-cache in memory, allowing space for larger batch sizes.
      * **Limitation -** The reduction in key-value heads comes with a potential accuracy drop
   * Grouped-query attention (GQA,2023):
      * GQA strikes a balance between MHA and MQA by projecting key and values to a few groups of query heads. Within each of the groups, it behaves like multi-query attention.
      * This is a balance between memory requirements and model quality
      * E.g., Llama 2 70B uses GQA
   * Flash attention (2013):
     * An algorithm that optimizes the attention mechanism at the hardware level.
     * It reduces the number of memory reads/writes between the GPU's slow HBM and fast SRAM, speeding up the prefill phase and handling long contexts gracefully.
     * It is an IO-aware exact attention algorithm that splits inputs into blocks fitting into fast GPU on-chip SRAM to mitigate memory bandwidth bottlenecks
7. Model optimization:
   * Quantization:
     * Quantization is the process of reducing the precision of a model’s weights and activations.
     * Most models are trained with 32 or 16 bits of precision, where each parameter and activation element takes up 32 or 16 bits of memory—a single-precision floating point. However, most deep learning models can be effectively represented with eight or even fewer bits per value.  
     * Reduces memory usage drastically and speeds up math calculations with little loss in accuracy.
   * Pruning and Sparsity:
     * Removes redundant or less important weights from the network, reducing the overall parameter count that needs to be calculated.
     * It’s been shown that many deep learning models are robust to pruning, or replacing certain values that are close to 0 with 0 itself.
     * Sparse matrices are matrices where many of the elements are 0. These can be expressed in a condensed form that takes up less space than a full, dense matrix.

   * Knowledge distillation:
      * Another approach to shrinking the size of a model is to transfer its knowledge to a smaller model through a process called distillation. This process involves training a smaller model (called a student) to mimic the behavior of a larger model (a teacher).

References and further readings -
* [Mastering LLM Techniques: Inference Optimization, Nvidia (Nov, 2023)](https://developer.nvidia.com/blog/mastering-llm-techniques-inference-optimization/)
* [(Multi-Query Attention) Fast Transformer Decoding: One Write-Head is All You Need, 2019](https://arxiv.org/pdf/1911.02150)
* [Grouped Query Attention: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints](https://arxiv.org/pdf/2305.13245v2)
* [Optimizing Inference for Long Context and Large Batch Sizes with NVFP4 KV Cache](https://developer.nvidia.com/blog/optimizing-inference-for-long-context-and-large-batch-sizes-with-nvfp4-kv-cache/)
* [A Visual Guide to Attention Variants in Modern LLMs](https://magazine.sebastianraschka.com/p/visual-attention-variants)
* [Distilling knowledge in neural network - 2015 (Research paper summary)](https://github.com/Taaniya/research-paper-summaries/blob/main/architecture_advancements/architecture_advancements_papers.md#distilling-knowledge-in-neural-network--2015)


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
