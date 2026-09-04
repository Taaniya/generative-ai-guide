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


### What is Query routing?
* Query routing in Retrieval-Augmented Generation (RAG) is performed after query translation. This is an advanced decision-making step that directs a user's input to the most appropriate data source, tool, or processing strategy before searching.
* Instead of sending every question through a single, naive retrieval pipeline, a router acts like an intelligent traffic cop. It analyzes user intent and forwards the request to the right destination—such as a vector database for unstructured text, a relational database (Text-to-SQL) for structured data, a knowledge graph, or even skipping retrieval entirely for general knowledge. 

#### Core Types of Routing
* **Logical Routing:** Uses a language model equipped with descriptions of available data sources or tools. The LLM reasons about the query and outputs a structured choice (like a function call or classification tag) to pick the best path.
* **Semantic Routing:** Embeds the incoming user query and measures its vector similarity against preset prompt descriptions or category anchors. The route with the highest similarity score wins. This method is fast and avoids LLM reasoning overhead.

#### Why Routing Matters
* **Reduces Latency and Cost:** Simple queries or chit-chat can bypass heavy vector searches or web lookups, saving tokens and speeding up response times.
* **Improves Accuracy:** Complex queries that need precise numbers can go straight to SQL or graph databases, while conceptual questions go to vector search.
* **Prevents Bad Retrieval:** Sending an irrelevant query to a specialized retriever injects noisy context that can degrade the final answer quality.


Further readings:
  * [Research paper RAGRouter: Learning to Route Queries to Multiple Retrieval-Augmented Language Models, Zhang et al., NeurIPS 2025](https://openreview.net/pdf?id=4VKVUmE1I8)


### How does semantic routing work?
* Semantic routing works by turning the routing choice into a vector similarity problem instead of a natural language reasoning problem.
* Instead of asking an LLM to read descriptions and pick a path, semantic routing matches the mathematical "meaning" (embedding) of a user's query against pre-defined route templates.

**1. Defining the Routes:** 
* You start by defining your destinations (routes) and providing a handful of example phrases for each.
* These examples act as anchors for what that route represents.
    * Route A (Technical Support): "How do I reset my password?", "My account is locked", "Where do I update my API key?"
    * Route B (Chit-Chat): "Hello", "How are you doing?", "Good morning bot."
    * Route C (Product Recommendations): "What is your best running shoe?", "Recommend a laptop for gaming", "Which plan should I buy?"
 
**2. Creating Vector Anchors:**
* Before handling any live traffic, your system passes all of these example phrases through an embedding model (like OpenAI's text-embedding-3-small or Hugging Face models). This converts the phrases into a set of coordinates (vectors) that capture their semantic meaning, which are stored locally in memory.

**3. Processing the Incoming Query:**
* When a user submits a live query (e.g., "I forgot my login info"), the system immediately generates an embedding for **only that query** using the exact same embedding model.

**4. Mathematical Matching:**
* The system runs a fast mathematical comparison—typically using Cosine Similarity—between the new query vector and all your pre-calculated example vectors.

**5. Executing the Selected Route:**
* The route associated with the closest matching example vector is selected.
* For "I forgot my login info", the system will find high similarity with Route A's examples.
* The query is then instantly forwarded to the specialized technical support RAG pipeline or vector database index.

Semantic vs. Logical Routing: Quick Comparison
|Feature | Semantic Routing | Logical (LLM) Routing|
|----|-----|----|
| Mechanism | Vector similarity scoring | LLM text reasoning / Function calling | 
| Speed | Extremely Fast (milliseconds) | Slower (waiting for LLM generation) | 
| Cost | Very Cheap (only requires one embedding) | Higher (requires input/output tokens) | 
| Complexity | Best for clear, distinct intent categories | Best for nuanced logic or dynamic variables|


### What is Reciprocal Rank Fusion?
* Reciprocal Rank Fusion (RRF) merges multiple ranked search lists into one unified list by calculating a new score for each document based only on its position in each list, without comparing raw score numbers. It is widely used in hybrid search systems to combine keyword searches and vector searches.

#### How the RRF Formula Works
* Instead of dealing with different score scales from different search tools, RRF looks strictly at where a document placed (its rank).
* The Score Equation: For each document, the system calculates a score using the formula:

  $Score = \sum \frac{1}{k+\text{rank}}$
  
* The Constant (k): The letter k is a small smoothing constant, usually set to 60. This stops top-ranked items from completely overpowering the final score.
* The Summation: If a document appears in more than one list, its scores from each list are added together. 
