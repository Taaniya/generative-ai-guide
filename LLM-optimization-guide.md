### What is KV cache? How does it help in optimizing latency during LLM inference?
* It is used to speed up the autoregressive decoding phase of an LLM for text generation by caching internally computed matrices in its attention layers to reuse them later for predicting subsequent tokens.
* When the model receives an input prompt, during prefill phase, each of its attention layers compute their Key and value matrices internally, and cache them in GPU's high-speed memory.
* During autoregression, when a new token is generated and fed back as part of the input sequence to the model, the Key and Value matrics of previous prompt tokens are reused from KV cache instead of being recomputed again, and only the key and value vectors only for the new token are computed and appended to the existing cache for generating subsequent ones and so on.
* **Performance improvement -** This makes each subsequent step a constant-time operation rather than re-processing the entire sequence history, which is essential for handling large contexts
    * Note that the initial "prefill" stage (processing the prompt) is slower because the cache must be built from scratch, while subsequent token generation is faster

* **Memory usage impact -**
    * It trades off memory for latency and computation.
    * For large models, the cache can take up significantly more memory than the model weights themselves 
    * Because the cache grows with the sequence length and batch size, it often consumes the majority of GPU memory during inference. The memory usage can be calculated as:

`Memory = 2 * precision * layers * dimension * sequence length  * batch size`

* where, 2 =  K and V matrices
* precision = bytes per parameter (E.g., 2 for fp16 inference)
* layers = No. of layers in the model
* dimension = dimension of embeddings
* sequence length = length we want to generate at the end, including the prompt tokens

Reference -
* [KV cache: Memory usage in transformers](https://youtu.be/80bIUggRJf4?si=94HRT3CFXODeaIta)
* [What is a KV cache, and why does it make LLM inference faster? - Sebastian Raschka](https://sebastianraschka.com/faq/docs/kv-cache.html)



### Quantization 
**Related courses -**
* [Quantization in depth with Pytorch, by DeepLearning.ai (Huggingface)](https://learn.deeplearning.ai/courses/quantization-in-depth/information)

Documentation & Tutorials -
* [Quantization - Pytorch](https://docs.pytorch.org/docs/stable/quantization.html)
* [Git repo - Torchao: PyTorch-Native Training-to-Serving Model Optimization](https://github.com/pytorch/ao)



