## RAG
* [RAG solution design and evaluation guide - Microsoft](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/rag/rag-solution-design-and-evaluation-guide)

### Chunking strategies
Semantic chunking:
* Semantic chunking divides text based on changes in meaning rather than fixed character or token counts.
* It is the most reliable choice for continuous prose, essays, and conversational transcripts.
* **How it works:** It measures the embedding distance or cosine similarity between consecutive sentences or small windows of text. When the similarity score drops below a specific threshold (indicating a shift in topic), a new chunk boundary is created.
* **Pros:**
   * Produces highly coherent, contextually unified chunks;
   * prevents mid-sentence or mid-thought cutoffs; naturally aligns with distinct topics.
* **Cons:**
   * Computationally expensive and slower because it requires generating embeddings for sentences during the ingestion phase;
   * sensitive to threshold hyperparameter tuning.
* **Why it's reliable:**
   * It ensures that a single, complete thought is never sliced in half.
   * It dynamically adapts to the author's natural transition of ideas.

* **Risk mitigated:**
   * It eliminates the risk of fixed-size boundaries cutting off a crucial sentence or formula mid-way.

Hierarchical (Parent-Child) Chunking:
* Hierarchical chunking creates multiple nested layers of text chunks at different levels of granularity or resolution.
* It is the most robust option for technical manuals, legal documents, and financial reports.
* **How it works:** A large parent chunk (e.g., an entire section or 1024 tokens) is subdivided into smaller child chunks (e.g., 128–256 tokens). The vector database indexes and searches the smaller child chunks for maximum precision, but returns or feeds the larger parent context to the LLM during generation.
* **Pros:**
   * Solves the core trade-off between retrieval precision (small chunks) and contextual richness (large chunks);
   * highly interpretable multi-resolution retrieval.
* **Cons:**
   * Increases storage overhead because both parent and child representations must be saved;
   * more complex pipeline architecture to manage relationships
* **Why it's reliable:**
   * It decouples search from generation. By indexing small "child" chunks, the vector search easily finds exact matches.
   * By feeding the large "parent" chunk to the LLM, the model never loses the broader context (like table headers or overarching themes)
* **Risk mitigated:**
   * It eliminates the risk of an LLM hallucinating due to a tiny, isolated snippet lacking context.

Recursive Character Text Chunking:
* (Most Reliable Baseline)If you want something that "just works" out of the box with zero configuration or high computational overhead, Recursive Character Chunking (using a hierarchy of separators like `\n\n`, `\n`,` `, `""`) is the industry baseline.
* **Why it's reliable:**
   * It respects structural boundaries (paragraphs first, then sentences, then words) without needing expensive machine learning models to calculate semantic shifts.
* **Risk mitigated:**
   * It acts as a highly predictable, low-cost safety net for mixed-format data.

   
Other chunking strategies:
 * Fixed size
 * Adaptive
 * Context-enriched chunking
 * AI-driven dynamic chunking
 * Evaluating chunking approaches with quantitative metrics
 * Best practices & implementation guidelines
 * Advanced techniques

  References:
   * [Databricks - Ultimate guide to chunking strategies for RAG](https://community.databricks.com/t5/technical-blog/the-ultimate-guide-to-chunking-strategies-for-rag-applications/ba-p/113089)
  * [Pinecone - chunking strategies](https://www.pinecone.io/learn/chunking-strategies/)
  * [Microsoft - Chunking approaches](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/rag/rag-chunking-phase#chunking-approaches)
  * [Implement RAG chunking strategies with LangChain and watsonx.ai - IBM](https://www.ibm.com/think/tutorials/chunking-strategies-for-rag-with-langchain-watsonx-ai)
  * [Text Chunking strategies -Qdrant](https://qdrant.tech/course/essentials/day-1/chunking-strategies/)
  * [Chunking Strategies to Improve LLM RAG Pipeline Performance - Weaviate, Sept 2025](https://weaviate.io/blog/chunking-strategies-for-rag)
 
### What is Query translation in RAG?
*  Query translation rewrites or expands user questions using a language model before searching a database to improve retrieval accuracy

#### Why is it needed?
* **Mismatch in text types:** A basic RAG flow embeds both the user question and the target documents to find similarities. However, questions are often short, sparse, potentially vague, or ill-worded, whereas documents are typically dense, structured and informative text chunks.
* **Bridging the gap:** By translating the query, the system maps the question into a space that is more closely aligned with the language, tone, or structure of the documents stored in the index, leading to more accurate search results.

#### Different types of query translation?
1. **Multi-Query:** Generates several differently worded variations of the original question from multiple perspectives to capture a wider range of relevant documents. 
2. **RAG-Fusion:** Expands a query into multiple variations like multi-query, but adds a ranking step using Reciprocal Rank Fusion to merge and sort the retrieved document lists.
3. **Query Decomposition:** Breaks a complex or multi-part question down into smaller, sequential, or independent sub-questions to address each part separately.
4. **Step-Back Prompting:** Abstracts a specific or detailed question into a higher-level, conceptual question to pull broader background context.
5. **HyDE (Hypothetical Document Embeddings):**
    * It works by having an LLM generate a hypothetical document that would answer the user's question, and then using embeddings of the generated document to perform the retrieval against the index.
    * Because the hypothetical document is generated in the same style and format as the real documents in your index, it sits much closer to them in the high-dimensional embedding space.
    * This helps the retriever find more accurate and relevant results than it could with the raw question alone.

Reference -
1. [Tutorial - RAG from scratch, Langchain](https://youtube.com/playlist?list=PLfaIDFEXuae2LXbO1_PKyVJiQ23ZztA0x&si=GD_iDRvZkp-tnJwX)
