### What is KV cache? How does it help in optimizing latency during LLM inference?
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



### Quantization - The Core Compression Technique

**What Is Quantization?**

- Quantization reduces the number of bits used to store model weights.
- Analogy: Instead of storing pi as `3.14159...` (many bits), store it as `3.14` (fewer bits).
- Most LLMs today are released in **BF16** (Brain Float 16 — 16 bits per number).
- Quantization converts numbers into lower bit formats: **FP8, INT8, or INT4**.

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



