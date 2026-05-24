# GenAI Document Retriever (RAG) – Complete Interview Preparation Guide
### FAANG / MAANG | Easy → Medium → Hard → Scenario-Based

---

> **How to use this guide:**  
> Read each question, understand the concept behind it, then study the answer. The difficulty levels are:
> - 🟢 **Easy** – Fundamentals, definitions, and basic concepts
> - 🟡 **Medium** – Design decisions, trade-offs, and comparisons
> - 🔴 **Hard** – Deep-dive internals, system design, and advanced optimizations
> - 🔵 **Scenario-Based** – Real-world problem-solving and debugging

---

## Project Summary (Open Every Interview With This)

> "We built a RAG-based GenAI document retriever for enterprise management use. The system allows non-technical users to ask questions in plain English and receive grounded answers from internal documents — PDFs, Word files, reports, policies, and contracts. We used LangChain for orchestration, an embedding model for semantic understanding, a vector database to store and retrieve chunks, and Google Gemini to generate the final answer. The core focus was on chunking strategy, retrieval precision, hallucination control, source citations, and production scalability."

---

## Core Flow (Memorize This)

```
Documents
   ↓
Parse (LangChain Loaders)
   ↓
Chunk (RecursiveCharacterTextSplitter)
   ↓
Embed (Embedding Model)
   ↓
Store (Vector DB + Metadata)
   ↓ ← ← ← ← ← ← ← ← ← ← ← ← ← ← User Query
Retrieve (Top-K Semantic Search + Filters)
   ↓
Build Prompt (Context + Query)
   ↓
LLM (Google Gemini)
   ↓
Answer + Citations
```

---

# 🟢 EASY – Fundamental Questions

---

### E1. What is RAG and why do we need it?

**Answer:**

RAG stands for **Retrieval Augmented Generation**. It is an architecture where, instead of asking an LLM to answer from its training data alone, we first **retrieve** relevant information from an external knowledge source (like a document database) and then **augment** the LLM's prompt with that retrieved content before asking it to **generate** an answer.

We need it because:

- LLMs have a knowledge cutoff and do not know about internal company documents.
- Fine-tuning a model on private data is expensive, slow, and hard to update.
- RAG allows the model to answer from fresh, domain-specific, and private data without any retraining.
- It naturally supports source citations, which builds trust.

**Interview Line:**
> "RAG bridges the gap between a general-purpose LLM and private enterprise knowledge by injecting retrieved evidence into the prompt at query time."

---

### E2. What is an embedding?

**Answer:**

An embedding is a **numerical representation of text in a high-dimensional vector space**. Words or sentences that are semantically similar are placed close together in this space.

For example:

- "revenue target" and "sales goal" would have vectors close to each other.
- "HR policy" and "quarterly revenue report" would be far apart.

Embeddings allow us to search by **meaning** rather than exact keywords, which is the foundation of semantic search in a RAG system.

The embedding model (such as `text-embedding-ada-002` by OpenAI or Google's embedding models) takes a piece of text and returns a fixed-length vector, e.g., 768 or 1536 dimensions.

---

### E3. What is a vector database and why do we use it?

**Answer:**

A vector database is a **specialized database designed to store, index, and search high-dimensional vectors efficiently**.

When we embed all document chunks, we store those vectors in a vector database. When a user asks a question, we embed that query too and search for vectors that are nearest to it.

Popular vector databases include:

- **FAISS** – Facebook's open-source library, great for local and small-scale use
- **Pinecone** – Managed cloud vector DB, production-grade
- **Weaviate** – Open-source, supports hybrid search
- **Chroma** – Lightweight, easy to use for prototyping
- **Qdrant** – Open-source with strong filtering support

We use vector DBs instead of relational DBs because SQL cannot do semantic similarity search efficiently.

---

### E4. What is chunking and why is it important?

**Answer:**

Chunking is the process of **splitting large documents into smaller, manageable text units** before embedding them.

We chunk because:

- Embedding models have token limits (e.g., 512 or 8192 tokens).
- Embedding a 300-page document as one unit produces a very broad, imprecise vector.
- Smaller chunks lead to more targeted, accurate retrieval.
- The LLM also has a context window limit — passing only relevant chunks stays within that limit.

**Common chunking parameters:**
- Chunk size: 500–1000 tokens
- Overlap: 80–150 tokens (overlap prevents context loss at boundaries)

---

### E5. What is top-k in retrieval?

**Answer:**

Top-k is the **number of most similar document chunks returned** from the vector database for a given query.

If top-k = 5, the system returns the 5 chunks whose embeddings are most similar to the query embedding.

Choosing the right k involves a trade-off:

- **Too small (k=2):** May miss the relevant chunk if it is ranked slightly lower.
- **Too large (k=10):** Introduces noise, increases cost, and may confuse the LLM with irrelevant context.

A typical starting point is k=4 or k=5, tuned based on evaluation.

---

### E6. What is cosine similarity and how is it used in retrieval?

**Answer:**

Cosine similarity is a **mathematical measure of the angle between two vectors**. It ranges from -1 to 1, where:

- 1 means identical direction (most similar)
- 0 means perpendicular (unrelated)
- -1 means opposite

In a vector database, when a user submits a query, the system embeds it and finds document chunks whose vectors have the **highest cosine similarity** to the query vector.

**Why cosine and not Euclidean distance?**
Cosine similarity focuses on direction (meaning), not magnitude (length). This makes it robust to differences in text length.

---

### E7. What is LangChain and what role does it play here?

**Answer:**

LangChain is a **Python framework that orchestrates all the moving parts of an LLM-based pipeline**.

In our RAG system, LangChain is responsible for:

- Loading documents using `DocumentLoaders` (PDF, DOCX, CSV, etc.)
- Splitting them using `RecursiveCharacterTextSplitter`
- Generating embeddings through integration with models
- Connecting to vector stores (FAISS, Pinecone, Chroma)
- Building `RetrievalQA` or custom chains that combine retrieval + LLM generation
- Managing prompt templates

Without LangChain, we would have to write all of these integrations manually. LangChain provides composable, reusable components.

---

### E8. What is the difference between semantic search and keyword search?

**Answer:**

| Aspect | Keyword Search | Semantic Search |
|---|---|---|
| Method | Exact word match | Meaning-based vector similarity |
| Example | "revenue target" matches only "revenue target" | "revenue target" also matches "sales goal", "income objective" |
| Technology | Inverted index (like Elasticsearch) | Embedding model + vector DB |
| Failure case | Misses synonyms and paraphrases | May match loosely related content |
| Best for | Exact lookups, structured queries | Natural language questions, diverse vocabulary |

In enterprise documents, different users phrase things differently, so semantic search is far more reliable.

---

### E9. What is a prompt template and why is it used in RAG?

**Answer:**

A prompt template is a **structured blueprint for the input sent to the LLM**. In a RAG system, the prompt typically includes:

- A system instruction telling the LLM to answer only from the provided context
- The retrieved document chunks as context
- The user's question

**Example template:**

```
You are an enterprise document assistant.
Answer the user's question using ONLY the context below.
If the answer is not in the context, say "Information not found in the provided documents."

Context:
{retrieved_chunks}

Question:
{user_question}

Answer:
```

Using a well-crafted prompt template reduces hallucination and keeps the model grounded.

---

### E10. What types of documents can the system handle?

**Answer:**

LangChain supports a wide variety of document loaders:

- **PDFs** – `PyPDFLoader`, `PDFMinerLoader`
- **Word documents** – `Docx2txtLoader`
- **CSV / Excel** – `CSVLoader`, `UnstructuredExcelLoader`
- **Plain text** – `TextLoader`
- **HTML** – `BSHTMLLoader`
- **Confluence / Notion / Slack** – via integration loaders
- **Scanned PDFs** – Requires OCR (e.g., Tesseract) before loading

Each loader extracts text from the file, which then goes through the chunking and embedding pipeline.

---

### E11. What is metadata and how is it used in RAG?

**Answer:**

Metadata is **additional information stored alongside each document chunk** in the vector database.

Examples of metadata:

- `file_name`: "Q3_Finance_Report.pdf"
- `page_number`: 12
- `department`: "Finance"
- `document_type`: "Quarterly Report"
- `upload_date`: "2024-09-01"
- `sensitivity_level`: "Confidential"

Metadata is used for:

1. **Filtering** – Retrieve only finance documents when the user asks a finance question
2. **Citations** – Show "Source: Q3_Finance_Report.pdf, Page 12"
3. **Access control** – Restrict retrieval based on user role
4. **Audit logs** – Track which documents were used

---

### E12. What is hallucination in LLMs and how does RAG reduce it?

**Answer:**

Hallucination is when an LLM **generates confident-sounding but factually incorrect or unsupported content**. It happens because LLMs predict the next token based on patterns, not because they "know" facts.

RAG reduces hallucination by:

1. **Grounding the prompt** with real retrieved content
2. **Instructing the model** to answer only from the provided context
3. **Adding a fallback** – if relevant content is not found, the model says "Information not found"
4. **Showing citations** – so answers can be verified against source documents

RAG does not eliminate hallucination entirely, but it significantly reduces it for domain-specific queries.

---

# 🟡 MEDIUM – Design and Trade-off Questions

---

### M1. How do you choose the right chunk size for your documents?

**Answer:**

Chunk size is one of the most important hyperparameters in RAG. There is no single correct answer — it depends on the document type, the nature of queries, and the embedding model.

**Key considerations:**

**Too small (< 200 tokens):**
- Chunks may lose context (e.g., a revenue figure without the region it belongs to)
- More chunks to search, higher storage and retrieval cost
- Individual chunks may not contain enough meaning to be useful

**Too large (> 1500 tokens):**
- Embedding becomes too broad, captures mixed topics
- Retrieval becomes imprecise
- Larger context may dilute the LLM's focus

**Practical guidelines by document type:**

| Document Type | Recommended Chunk Size | Overlap |
|---|---|---|
| Dense financial reports | 600–800 tokens | 100–150 tokens |
| HR policy documents | 400–600 tokens | 80–120 tokens |
| FAQ-style documents | 200–400 tokens | 50–80 tokens |
| Technical manuals | 700–1000 tokens | 100–150 tokens |

**How to decide:**
- Run retrieval experiments with different chunk sizes
- Evaluate whether the top-k results contain the answer
- Use evaluation datasets with known query-document pairs

---

### M2. Why is chunk overlap important and how do you set it?

**Answer:**

Chunk overlap ensures that **sentences or ideas that span a chunk boundary are not lost**.

**Without overlap:** Consider a sentence like "The North region revenue target of 12% was set in Q3." If this sentence starts at the end of chunk 4 and ends at the beginning of chunk 5, then neither chunk contains the complete sentence. Retrieval may miss this critical fact.

**With overlap:** Chunk 4 ends at token 800, chunk 5 starts at token 700. Both chunks contain the full sentence, so retrieval is more likely to surface it.

**Setting overlap:**
- A general rule is overlap = 10–20% of chunk size
- For a 600-token chunk, use 80–120 tokens of overlap
- Too much overlap increases storage cost and introduces redundancy

**Trade-off:**
More overlap → better context continuity, but more storage and computation.

---

### M3. RAG vs Fine-tuning — when would you choose each?

**Answer:**

This is a very common FAANG interview question. The answer depends on the use case.

**Choose RAG when:**
- Documents change frequently (e.g., monthly reports, policy updates)
- You need source citations and explainability
- You need to add new knowledge without retraining
- The dataset is large and diverse
- Budget and time are limited
- You want grounded, verifiable answers

**Choose Fine-tuning when:**
- You need the model to adopt a specific tone, persona, or output format
- You have a fixed, stable knowledge base that does not change often
- The task requires style transfer (e.g., always respond in legal language)
- Domain-specific terminology needs to be deeply embedded

**In practice, many enterprise systems combine both:**
- Fine-tune for style and format
- RAG for fresh, domain-specific knowledge retrieval

---

### M4. How do you evaluate the quality of a RAG system?

**Answer:**

Evaluation has two dimensions: **retrieval quality** and **generation quality**.

**Retrieval Metrics:**

| Metric | What it Measures |
|---|---|
| Hit Rate (Recall@k) | Is the correct chunk in the top-k results? |
| Mean Reciprocal Rank (MRR) | How highly is the correct chunk ranked? |
| Precision@k | How many of the top-k chunks are actually relevant? |
| NDCG | Quality of ranking, gives higher weight to top positions |

**Generation Metrics:**

| Metric | What it Measures |
|---|---|
| Faithfulness | Is the answer grounded in the retrieved context? |
| Answer Relevancy | Does the answer address the question? |
| Context Precision | Are the retrieved chunks relevant to the question? |
| Context Recall | Does the retrieved context contain all the needed information? |

**Frameworks:**
- **RAGAS** – Popular open-source framework for RAG evaluation
- **TruLens** – Another evaluation framework
- Custom LLM-as-judge pipelines

**Interview Line:**
> "I evaluate retrieval and generation separately. Retrieval is measured by recall@k and MRR. Generation is measured by faithfulness and answer relevancy using RAGAS."

---

### M5. What is reranking and when should you use it?

**Answer:**

Reranking is a **second-pass scoring step** that takes the initial top-k results from vector search and re-scores them using a more powerful but slower model.

**Why it is needed:**
- Vector search uses approximate nearest neighbors, which is fast but not always precise.
- The initial ranking may not perfectly reflect relevance to the specific query.
- A reranker (often a cross-encoder model) compares the query and each chunk together, producing more accurate relevance scores.

**How it works:**

1. Vector DB returns top-20 chunks
2. A reranker model scores all 20 against the query
3. Top-5 highest-scoring chunks are sent to the LLM

**Popular rerankers:**
- Cohere Rerank API
- BGE Reranker (open-source)
- Cross-encoder models from sentence-transformers

**Trade-off:**
Reranking improves precision but adds latency and cost. Use it when retrieval quality matters more than speed.

---

### M6. What is hybrid search and when is it beneficial?

**Answer:**

Hybrid search **combines vector (semantic) search with keyword (BM25) search** to get the benefits of both.

**How it works:**
1. Run keyword search (BM25) — good for exact terms, codes, product names
2. Run vector search — good for semantic meaning
3. Merge and re-score results using a technique called Reciprocal Rank Fusion (RRF)

**When to use hybrid search:**

- Documents contain **product codes, IDs, names** that must be matched exactly (e.g., "Policy ID: HR-2024-007")
- Documents mix structured data and free-form text
- Users may include exact technical terms in their query

**Example:**
If a user searches "Policy HR-2024-007 leave entitlement", vector search may return semantically similar leave policies but miss the exact policy ID. Hybrid search catches both.

---

### M7. How do you handle documents in multiple languages?

**Answer:**

Multi-language support requires careful choices at each layer:

**Embedding Layer:**
- Use a multilingual embedding model such as `multilingual-e5` or `paraphrase-multilingual-mpnet-base-v2`
- These models map semantically similar content across languages to nearby vectors

**Chunking Layer:**
- Language-aware chunkers handle sentence boundaries correctly
- Avoid splitting mid-sentence in languages with different grammar structures

**LLM Layer:**
- Gemini and many modern LLMs support multilingual generation natively
- Instruct the model on which language to respond in

**Metadata Layer:**
- Store language as a metadata field
- Optionally filter by language if users prefer responses in one language

**Optional Approach – Translate and Unify:**
- Translate all documents to a single language (e.g., English) before embedding
- Simpler architecture, but loses nuance and adds translation overhead

---

### M8. What are the limitations of RAG?

**Answer:**

RAG is powerful but has known limitations:

**1. Retrieval bottleneck:**
If the relevant chunk is not retrieved, the LLM cannot answer correctly, regardless of how smart it is. The system is only as good as its retrieval.

**2. Multi-hop reasoning:**
If an answer requires combining information from multiple documents (e.g., "Compare the Q2 and Q3 reports"), simple RAG may not retrieve and connect both correctly.

**3. Long-range dependencies:**
If an answer requires context from many scattered sections of a large document, chunking may split that context across too many chunks.

**4. Embedding model quality:**
Poor embeddings mean poor semantic search. Domain-specific vocabulary (medical, legal, financial) may not be well-represented by general embedding models.

**5. Context window constraints:**
Even after retrieval, the LLM still has a context window. If chunks are large and numerous, they may not all fit.

**6. Latency:**
The retrieval step adds network and compute latency compared to a direct LLM call.

**7. Stale index:**
If the ingestion pipeline lags, newly updated documents may not reflect in answers.

---

### M9. What is the difference between FAISS, Pinecone, and Weaviate?

**Answer:**

| Feature | FAISS | Pinecone | Weaviate |
|---|---|---|---|
| Type | Local library | Managed cloud service | Open-source / cloud |
| Best for | Prototyping, local dev | Production at scale | Hybrid search, knowledge graphs |
| Managed | No (self-hosted) | Fully managed | Self-hosted or cloud |
| Metadata filtering | Limited | Strong | Strong |
| Horizontal scaling | Manual | Built-in | Built-in |
| Cost | Free (compute only) | Paid (usage-based) | Free (self-hosted) |
| Persistence | Manual (save to disk) | Always-on cloud | Built-in persistence |
| ANN algorithm | HNSW, IVF, etc. | Internal (optimized) | HNSW |

**When to use what:**
- Prototype or local testing → FAISS
- Enterprise production, no infrastructure management → Pinecone
- Self-hosted with hybrid search → Weaviate or Qdrant

---

### M10. How do you handle table data in PDFs during ingestion?

**Answer:**

Tables are one of the trickiest parts of document ingestion. Standard text extraction often destroys table structure.

**Challenges:**
- Tables are rendered as visual grids in PDFs, not natural text
- Row-column relationships are lost when extracted linearly
- Numbers and labels may get mixed up

**Solutions:**

1. **Markdown table extraction** – Tools like `unstructured.io` can detect table regions and convert them to Markdown format (e.g., `| Column A | Column B |`)

2. **CSV-based storage** – Extract tables as structured CSV, store separately with a text description, embed the description for retrieval, and return the raw table as context

3. **Camelot / pdfplumber** – Python libraries that extract table data with row-column awareness from PDFs

4. **Multimodal approach** – Use vision-capable models to read the table image and describe its contents

5. **Pre-processing pipeline** – Add a table detection stage before chunking to handle tables as a special document type

---

### M11. How do you handle document updates and versioning?

**Answer:**

In enterprise systems, documents are frequently revised. Handling updates properly avoids stale answers.

**Strategy 1: Version-aware metadata**
- Store `version_id`, `is_active`, and `updated_at` with every chunk
- When a document is updated, mark old chunks as `is_active = False` and insert new chunks with the new version
- Retrieval filters on `is_active = True` only

**Strategy 2: Full delete and re-insert**
- When a document is updated, delete all chunks with that `document_id` from the vector DB
- Re-run the full ingestion pipeline for the new version
- Simpler but creates a brief window where the document is unavailable

**Strategy 3: Soft delete with audit trail**
- Keep old versions for audit and compliance purposes
- Default query retrieves only the latest version
- Support a "show older version" mode for compliance teams

**De-duplication consideration:**
If the same document is uploaded twice, detect it using a hash of the file content before running ingestion.

---

# 🔴 HARD – Deep-Dive and Advanced Questions

---

### H1. How does HNSW indexing work and why is it used in vector databases?

**Answer:**

HNSW stands for **Hierarchical Navigable Small World**. It is the most widely used Approximate Nearest Neighbor (ANN) algorithm in modern vector databases.

**Why ANN?**
Exact nearest neighbor search compares a query vector against every stored vector — this is O(n) and becomes extremely slow at millions of vectors.

**How HNSW works:**
HNSW builds a multi-layer graph:

- **Bottom layer** – All vectors are nodes, connected to their nearest neighbors
- **Higher layers** – Progressively fewer nodes, acting like a "highway" for fast navigation
- **Search** – Start from the top layer (fewest nodes), navigate toward the target area, drop to the next layer, repeat until reaching the bottom layer with precise neighbors

**Why HNSW is good:**
- O(log n) search time
- High recall (close to exact search)
- Supports dynamic insertions (no need to rebuild the index)

**Trade-off:**
- High memory usage (stores graph structure alongside vectors)
- Index construction is slower than flat indexes

---

### H2. Explain multi-stage retrieval architecture in detail.

**Answer:**

Multi-stage retrieval is a production pattern where retrieval is done in **multiple passes of increasing precision but decreasing scope**.

**Stage 1 – Pre-filtering (Metadata):**
Apply hard filters based on known attributes before any vector search.

Example: User is in Finance → filter to `department = Finance` only.

This reduces the search space from 1 million chunks to maybe 50,000.

**Stage 2 – Approximate Vector Search (ANN):**
Run semantic similarity search on the filtered subset.

Return top-50 candidates.

**Stage 3 – Reranking:**
Pass top-50 candidates through a cross-encoder reranker.

Score each candidate precisely against the query.

Return top-5.

**Stage 4 – Context Assembly:**
Merge top-5 chunks intelligently.

- Deduplicate overlapping content
- Order by relevance or document structure
- Trim to fit within LLM context window

**Stage 5 – LLM Generation:**
Pass assembled context to Gemini with grounding instructions.

**Why this matters:**
Each stage is a quality gate. Stage 1 reduces cost. Stage 2 provides recall. Stage 3 improves precision. Stage 4 optimizes context. Stage 5 generates the answer.

---

### H3. How would you detect and measure hallucination in generated answers?

**Answer:**

Hallucination detection is an open research problem, but several practical approaches exist.

**Approach 1 – Faithfulness scoring (RAGAS):**
RAGAS measures whether each claim in the answer can be attributed to the retrieved context. It uses an LLM to compare the answer against the chunks and returns a faithfulness score between 0 and 1.

**Approach 2 – NLI-based entailment:**
Use a Natural Language Inference (NLI) model to check whether the context "entails" the answer. If the answer contains a claim not entailed by any context chunk, it is flagged as potentially hallucinated.

**Approach 3 – Self-consistency check:**
Ask the LLM the same question multiple times with slight prompt variation. If answers are inconsistent, hallucination risk is high.

**Approach 4 – Citation grounding:**
Require the model to cite the specific chunk that supports each claim. Then validate that the cited chunk actually contains the claim.

**Approach 5 – Answer absence detection:**
If similarity scores of retrieved chunks are below a threshold, instruct the system to respond "Information not found" instead of generating an answer.

**Production monitoring:**
- Log all queries, retrieved chunks, and generated answers
- Periodically sample and run hallucination evaluation
- Maintain a labeled evaluation dataset and track score over time

---

### H4. How would you build a self-correcting RAG pipeline?

**Answer:**

A self-correcting RAG pipeline, sometimes called **Corrective RAG (CRAG)**, adds verification and fallback steps to improve answer quality.

**Architecture:**

1. **Retrieve** – Get top-k chunks from vector DB
2. **Evaluate Retrieval Quality** – Use an LLM or classifier to score whether retrieved chunks are relevant to the query
   - If confident: proceed to generation
   - If uncertain: run a web search or expand retrieval
   - If irrelevant: reject and return "not found"

3. **Generate** – Build prompt and call LLM
4. **Verify Answer** – Cross-check answer against retrieved chunks using NLI or faithfulness model
5. **Refine if needed** – If verification fails, re-retrieve with a reformulated query or fallback to a safer response

**LangGraph integration:**
LangGraph (LangChain's agentic framework) allows building this as a stateful graph where nodes are retrieval, evaluation, and generation steps, and edges represent conditional transitions.

**When to use self-correcting RAG:**
- High-stakes domains: legal, compliance, medical
- When answer quality is more important than latency
- When queries are complex and may require iterative refinement

---

### H5. How do you optimize embedding generation at scale?

**Answer:**

At millions of documents, embedding generation becomes a bottleneck.

**Optimization strategies:**

**1. Batch processing:**
Do not embed one chunk at a time. Batch 64 or 128 chunks per API call.

Most embedding APIs support batch inputs, which amortizes the network overhead.

**2. Worker parallelism:**
Run multiple embedding workers concurrently using a message queue (e.g., Kafka, RabbitMQ, or AWS SQS).

Each worker picks up a batch of chunks, generates embeddings, and writes to the vector DB.

**3. Caching embeddings:**
If the same document chunk appears multiple times (e.g., a boilerplate legal disclaimer that is in every contract), cache its embedding to avoid redundant computation.

**4. Asynchronous ingestion:**
Ingestion should be non-blocking. When a document is uploaded, publish an event to a queue. Workers process asynchronously. The user interface shows "indexing in progress."

**5. Hardware acceleration:**
Embedding models run faster on GPU. For self-hosted models, use GPU-enabled inference servers (e.g., TorchServe, Triton Inference Server).

**6. Smaller embedding model for low-priority content:**
Use a lighter, cheaper model for non-critical documents and a higher-quality model for sensitive or high-frequency documents.

---

### H6. How would you implement access control at the retrieval layer?

**Answer:**

Access control in a RAG system must happen **before** chunks are sent to the LLM. Filtering after generation is too late — the LLM has already seen unauthorized content.

**Architecture:**

**Step 1 – User identity and role:**
- Authenticate the user (SSO / OAuth2)
- Fetch their role and permissions from an IAM system (e.g., LDAP, Active Directory, or custom RBAC)

**Step 2 – Metadata tagging:**
- During ingestion, tag each chunk with:
  - `owner_department`: "Legal"
  - `sensitivity_level`: "Confidential"
  - `allowed_roles`: ["Legal", "Executive"]

**Step 3 – Retrieval-time filter:**
- Before vector search, inject a metadata filter based on the user's role
- Example: If user is in "Finance," only retrieve chunks where `allowed_roles` contains "Finance" or "All"

**Step 4 – Audit logging:**
- Log every query with: user ID, role, query text, retrieved chunk IDs, and timestamp
- This supports compliance reviews

**Step 5 – Row-level security in the vector DB:**
- Some vector DBs (Weaviate, Qdrant) support tenant isolation, which provides hard separation at the data layer, not just the application layer.

**Never rely on prompt instructions alone:**
> "Ignore finance documents for this user" in the prompt is not secure. It can be bypassed. Security must be enforced in the retrieval filter, not in the prompt.

---

### H7. How do you handle multi-hop questions in RAG?

**Answer:**

A multi-hop question requires **combining information from multiple documents or sections** to arrive at the answer.

**Example:**
> "What was the revenue growth percentage in Q3 compared to the target set in the Q2 planning document?"

This requires:
1. Finding Q3 actual revenue → from Q3 Report
2. Finding Q3 target → from Q2 Planning Document
3. Computing the comparison

Simple RAG may not retrieve both chunks or connect them correctly.

**Solutions:**

**1. Query decomposition:**
Use an LLM to break the multi-hop question into simpler sub-questions.

- Sub-Q1: "What was the Q3 actual revenue?"
- Sub-Q2: "What was the Q3 revenue target set in Q2?"
- Then combine answers.

**2. Iterative retrieval:**
- First retrieve for Sub-Q1, get answer
- Use that answer as context to retrieve for Sub-Q2
- Repeat until all hops are resolved

**3. Knowledge graph augmentation:**
Build a knowledge graph over entities in the documents (companies, regions, quarters, metrics). Use graph traversal to find connected facts.

**4. Agentic RAG:**
Use an LLM agent that decides what to retrieve next based on what it knows so far. LangGraph or LangChain Agents support this pattern.

---

### H8. How would you design the observability layer for a production RAG system?

**Answer:**

Observability answers: "Is the system working correctly and where is it failing?"

**Metrics to track:**

| Category | Metric | Alert Threshold |
|---|---|---|
| Performance | Query latency (p50, p95, p99) | p99 > 5s |
| Performance | Ingestion latency per document | > 30s for standard doc |
| Retrieval | Top-k relevance score distribution | Score < 0.5 suggests poor retrieval |
| Generation | Faithfulness score (RAGAS) | < 0.7 triggers review |
| Cost | LLM tokens per query | Spike detection |
| Cost | Cache hit ratio | < 20% suggests optimization needed |
| Reliability | Error rate (retrieval + LLM failures) | > 1% triggers alert |
| Usage | Queries per second | Capacity planning |

**Logging:**
- Every query: timestamp, user_id, query_text, retrieved_chunk_ids, answer_text, latency
- Ingestion events: document_id, chunk_count, status, timestamp

**Tools:**
- LangSmith (LangChain's native observability platform)
- Arize AI / Phoenix for LLM evaluation
- Prometheus + Grafana for infrastructure metrics
- Datadog or New Relic for full-stack monitoring

**Feedback loop:**
- Add thumbs up/down in the UI
- Use negative feedback to flag bad answers for human review
- Use reviewed answers as ground truth for automated evaluation

---

### H9. What are the differences between dense retrieval, sparse retrieval, and hybrid retrieval?

**Answer:**

**Dense Retrieval:**
- Uses neural embedding models to encode text into dense vectors (e.g., 768-dim float)
- Captures semantic meaning
- Query: "sales goal" → can match "revenue target"
- Weakness: poor at exact keyword or code matching
- Examples: FAISS, Pinecone with text-embedding-ada-002

**Sparse Retrieval:**
- Uses traditional inverted index with TF-IDF or BM25 scoring
- Captures exact word matches and term frequency
- Query: "revenue target" → matches documents containing those exact words
- Weakness: misses synonyms and paraphrases
- Examples: Elasticsearch, OpenSearch, Lucene

**Hybrid Retrieval:**
- Combines both approaches
- Dense results + Sparse results are merged using Reciprocal Rank Fusion (RRF) or learned fusion
- Best of both worlds: handles both meaning and exact terms

**When to use what:**
- Dense only: general Q&A over free-form text
- Sparse only: product catalogs, exact code or ID lookup
- Hybrid: enterprise search where users mix natural language and specific terms

---

### H10. How do you handle context window overflow in large retrievals?

**Answer:**

LLMs have a fixed context window (e.g., Gemini 1.5 Pro: 1M tokens, but cost increases with length). Even so, passing everything is not optimal.

**Strategy 1 – Selective top-k:**
Only pass the top 3–5 most relevant chunks. Do not pass all retrieved content.

**Strategy 2 – Chunk compression:**
Use an LLM to summarize each retrieved chunk before including it in the prompt. This reduces token count while preserving key information.

**Strategy 3 – Map-reduce pattern:**
- For each retrieved chunk, run an independent "extract answer" LLM call
- Then combine extracted answers in a final synthesis call
- Avoids overwhelming a single prompt

**Strategy 4 – Lost in the middle awareness:**
Research shows LLMs pay more attention to content at the beginning and end of a long context. Place the most relevant chunks first or last.

**Strategy 5 – Dynamic context window:**
Measure how many tokens each chunk uses. Fill the context window greedily from highest to lowest relevance until the token budget is reached.

---

# 🔵 SCENARIO-BASED – Real-World Problem Solving

---

### S1. The system returns irrelevant answers

**Question:** Management reports that the system is returning answers that have nothing to do with the user's question. How do you debug this?

**Answer:**

When answers are irrelevant, the root cause is almost always in the retrieval layer, not the generation layer. Here is my systematic debugging approach:

**Step 1 – Inspect retrieved chunks directly:**
Log and print the actual top-k chunks returned for the failing query. This isolates whether the retrieval layer is the problem.

In many cases, irrelevant chunks are returned because the query vector is landing in the wrong part of the vector space.

**Step 2 – Check embedding quality:**
Run a quick sanity check. For the failing query, manually look at what chunks are similar. If "Q3 revenue target" retrieves chunks about "employee leave policy," the embedding model may not be well-suited for the domain.

Consider:
- Upgrading to a domain-specific embedding model
- Fine-tuning the embedding model on a small labeled dataset

**Step 3 – Review chunk boundaries:**
Check whether key facts are getting split across chunks. Poor chunk boundaries can result in semantically incomplete embeddings.

Experiment with larger chunk sizes and higher overlap.

**Step 4 – Add metadata filters:**
If the user's query clearly belongs to a domain (Finance, Legal, HR), add metadata pre-filtering to narrow the search space.

**Step 5 – Evaluate retrieval with labeled data:**
Build a small evaluation set with 50–100 queries and known correct chunks. Measure Recall@5. If it is below 0.7, systematic retrieval improvements are needed.

**Interview Line:**
> "My first step is always to inspect retrieved chunks before assuming the LLM is at fault. In most cases, the issue is chunking, embedding quality, or missing metadata filters."

---

### S2. The system hallucinates even though context is provided

**Question:** Gemini gives fluent but factually wrong answers even when the relevant chunk is in the retrieved context. How do you solve this?

**Answer:**

This is more nuanced than the retrieval problem, because the right content was retrieved but the LLM still generated wrong information.

**Root cause investigation:**

**Scenario A – Prompt is too permissive:**
If the prompt says "Answer the question based on the documents, using your knowledge if needed," the model will use its parametric knowledge and hallucinate.

Fix: Make the prompt strict. "Answer ONLY from the provided context. If not found, say 'Information not found.'"

**Scenario B – Multiple noisy chunks dilute the answer:**
If top-10 chunks are passed and most are irrelevant, the relevant evidence gets buried. The LLM may anchor on the wrong content.

Fix: Reduce top-k. Pass only the top 3–5 most relevant chunks.

**Scenario C – Relevant chunk is present but answer is implicit:**
The chunk says "growth was 12%." The user asks "what was the North region growth?" The connection requires inference. The model may hallucinate the region.

Fix: Improve chunking to ensure the entity (North region) and its fact (12%) are always in the same chunk.

**Scenario D – Model confuses similar values:**
Multiple chunks with different numbers exist. The model picks the wrong one.

Fix: Add structured metadata or headers to clarify which document, region, and period each chunk belongs to.

**Ongoing solution – Faithfulness monitoring:**
Run RAGAS faithfulness scoring on a sample of queries daily. Set an alert if faithfulness drops below 0.75.

---

### S3. Scaling from 10,000 to 100 million documents

**Question:** Your system works well for 10,000 documents. The company now has 100 million. It is slow and unstable. What do you redesign?

**Answer:**

At 100 million documents, this is a full distributed system engineering problem.

**Problem 1 – FAISS runs out of memory:**
FAISS is in-memory and single-node. At scale, migrate to Pinecone, Weaviate, or a distributed Qdrant cluster.

**Problem 2 – Ingestion pipeline is synchronous:**
Replace synchronous ingestion with an async event-driven architecture:
- Document upload triggers a message to a Kafka queue
- Ingestion workers (horizontally scalable) consume from the queue
- Each worker parses, chunks, embeds, and writes to vector DB
- Status tracked in a jobs database

**Problem 3 – Retrieval is slow:**
- Enable ANN indexing (HNSW) in the vector DB
- Apply metadata pre-filtering before vector search
- Add a caching layer (Redis) for frequent queries

**Problem 4 – LLM cost explodes:**
- Cache answers for common queries with a TTL
- Route simple factual questions to smaller, cheaper models
- Batch similar queries if running async

**Problem 5 – Embedding generation is a bottleneck:**
- Run embedding in parallel workers
- Batch embed 64–128 chunks per API call
- Cache embeddings for duplicate content

**Redesigned Architecture:**

```
Upload → Event Queue (Kafka)
       → Ingestion Workers (horizontal)
       → Object Store (raw files)
       → Vector DB (Pinecone / Weaviate)
       → Metadata DB (PostgreSQL)

Query → API Gateway → Auth → Role Filter
      → Cache Check (Redis)
      → Embedding Service
      → Vector DB (filtered ANN search)
      → Reranker
      → LLM (Gemini) → Answer + Citations
      → Observability (LangSmith / Datadog)
```

---

### S4. A new document must be searchable within 5 minutes of upload

**Question:** The business requires near-real-time indexing. How do you design an ingestion pipeline that satisfies this SLA?

**Answer:**

This is a real-time streaming ingestion problem.

**Pipeline design:**

**Step 1 – Upload triggers event:**
When a document is uploaded to object storage (S3 or GCS), an event is published to a message queue (e.g., Kafka, AWS SQS, or Google Pub/Sub).

**Step 2 – Ingestion service picks up immediately:**
A lightweight consumer service listens to the queue. When it receives a new file event, it:
- Downloads the file
- Runs OCR if needed (for scanned PDFs)
- Parses with LangChain loader
- Splits into chunks

**Step 3 – Parallel embedding:**
Chunks are sent in batches to the embedding service. For a typical 20-page document (~40 chunks), one API batch call takes < 2 seconds.

**Step 4 – Upsert to vector DB:**
Insert chunk vectors and metadata in a single batch write.

**Step 5 – Mark as active:**
Update the document status to `indexed = True` in the metadata store.

**Step 6 – Notify user:**
Send a webhook or WebSocket event to the UI: "Document is now searchable."

**SLA management:**
- Monitor ingestion latency with alerting
- Set p95 target: < 3 minutes for standard documents
- For large documents (> 100 pages), split into parallel workers per document section

---

### S5. A user asks a question that spans multiple documents

**Question:** A user asks: "Compare the Q2 and Q3 revenue performance and tell me if we are on track for the annual target." How does your system handle this?

**Answer:**

This is a multi-document, multi-hop question. Standard single-stage RAG may not handle it well.

**Why it is hard:**
- The answer requires data from at least three documents: Q2 report, Q3 report, and annual target plan.
- Single top-k retrieval may not surface all three.
- The LLM must synthesize and compare, not just extract.

**Approach 1 – Query decomposition:**
Use an LLM to decompose the original question into atomic sub-queries:
- Sub-Q1: "What was Q2 revenue performance?"
- Sub-Q2: "What was Q3 revenue performance?"
- Sub-Q3: "What is the annual revenue target?"

Run retrieval separately for each sub-query. Then combine all retrieved chunks into a synthesis prompt.

**Approach 2 – Multi-document retrieval with deduplication:**
Retrieve top-k for the full query, but force diversity in results — at least one chunk per relevant document. Some vector DBs support MMR (Maximal Marginal Relevance) retrieval for this.

**Approach 3 – Agentic RAG:**
Use a LangGraph agent:
1. Agent decides what to retrieve first
2. Retrieves Q3 data
3. Reads answer
4. Decides to retrieve Q2 for comparison
5. Reads answer
6. Retrieves annual target
7. Synthesizes final comparative answer

This is more accurate but adds latency.

**Prompt for synthesis:**
```
You are a financial analyst. Given the following data from multiple quarterly reports:

Q2 Data: {q2_context}
Q3 Data: {q3_context}
Annual Target: {target_context}

Compare Q2 and Q3 performance and assess if the annual target is likely to be met.
```

---

### S6. Management wants to know if the system can be trusted

**Question:** The CTO asks: "Before we roll this out company-wide, how do I know this thing won't give wrong answers that people will act on?" What do you say and what do you build?

**Answer:**

This is a trust and governance question, not just a technical one.

**What I would communicate:**

> "The system is designed to give grounded answers — meaning it only answers from retrieved documents, not from its imagination. Every answer shows the source document and page number so any user can verify it in 10 seconds. When the system is not confident, it says 'I could not find a reliable answer' rather than guessing."

**What I would build:**

**1. Source citations are non-negotiable:**
Every answer must display: "Source: Q3_Finance_Report.pdf, Page 12." Users can click to view the exact paragraph.

**2. Confidence thresholds:**
If the top retrieved chunk has a similarity score below a threshold (e.g., 0.6), the system responds: "I could not find a confident answer in the available documents."

**3. Human-in-the-loop for sensitive workflows:**
For critical decisions (executive reporting, legal, compliance), route answers through a human review step before distribution.

**4. Evaluation dashboard for leadership:**
Show ongoing metrics:
- Faithfulness score this week: 0.89
- Queries answered confidently: 94%
- Average source citation rate: 100%
- User thumbs up/down ratio: 87% positive

**5. Audit trail:**
Every query is logged. If someone acts on a wrong answer, the full retrieval chain is traceable.

**6. Regular red-teaming:**
Run periodic tests where domain experts ask known questions to verify answer quality.

---

### S7. LLM cost is growing faster than usage

**Question:** Your CFO says the monthly LLM bill tripled despite only a 30% increase in users. What happened and how do you fix it?

**Answer:**

This is a cost optimization problem rooted in inefficient usage patterns.

**Diagnosing the problem:**

**Check 1 – Token usage per query:**
Log input token count and output token count per query. If average input tokens grew from 1,000 to 3,000, the context being passed to the LLM grew.

This could mean top-k was increased without cost analysis, or chunks became larger.

**Check 2 – Repeated queries not being cached:**
If the same policy question is asked 5,000 times a day and every query hits the LLM, that is expensive. Check cache hit ratio.

**Check 3 – Model tier mismatch:**
Are all queries going to the largest, most expensive model even for simple FAQ questions?

**Fixes:**

**Fix 1 – Aggressive caching:**
- Cache at query embedding level: if a new query is very similar to a cached query (cosine similarity > 0.95), return the cached answer
- Cache at exact query level: hash the query string, store answer with TTL
- Estimated impact: 30–50% cost reduction for high-repeat query patterns

**Fix 2 – Model routing:**
- Classify queries by complexity before sending to LLM
- Simple FAQ-style questions → Gemini Flash (cheaper, faster)
- Complex reasoning or synthesis questions → Gemini Pro (more capable)

**Fix 3 – Reduce top-k:**
If top-k was set to 10, reduce to 4–5. Fewer chunks mean fewer input tokens.

**Fix 4 – Chunk compression:**
Before passing chunks to LLM, summarize each chunk to extract only the relevant sentence. Pass 3 compressed sentences instead of 3 full 500-token chunks.

**Fix 5 – Output token limits:**
Set `max_tokens` appropriately. If the system is generating long verbose answers when short answers suffice, the output token cost adds up.

---

### S8. A confidential document is being surfaced to unauthorized users

**Question:** An employee from the HR team accessed a salary band document that should only be visible to HR leadership. How did this happen and how do you prevent it?

**Answer:**

This is a critical security incident. The retrieval layer was not properly enforcing access control.

**Root cause analysis:**

**Root cause 1 – Access metadata not tagged during ingestion:**
If the document was ingested without `allowed_roles` metadata, it was treated as accessible to everyone.

**Root cause 2 – Metadata filter not applied at retrieval time:**
If the retrieval code was not injecting a filter based on the user's role, all documents were searched regardless of access.

**Root cause 3 – Role lookup failure:**
A bug in the IAM integration caused the user's role to fall back to a default that bypassed filters.

**Immediate remediation:**

1. Audit all documents in the vector DB to verify they have correct access metadata
2. Re-run ingestion for documents with missing or incorrect metadata
3. Patch the retrieval layer to enforce mandatory metadata filters
4. Add an automated test: a low-privilege user account must never retrieve a high-privilege document, verified in CI/CD

**Architectural fix:**

```python
def retrieve(query, user_role, top_k=5):
    # This filter is MANDATORY, not optional
    access_filter = {"allowed_roles": {"$in": [user_role, "All"]}}
    
    return vector_db.search(
        query_embedding=embed(query),
        filter=access_filter,   # Applied before vector search
        top_k=top_k
    )
```

**Monitoring:**
- Alert on any retrieval where `allowed_roles` filter was skipped
- Quarterly access control audit: compare document metadata with IAM policy

---

### S9. Users are asking questions the documents do not answer

**Question:** Many users are frustrated because they ask questions that have no answer in the documents. The system says "Information not found" but users think it is broken. What do you do?

**Answer:**

This is a user experience and expectation management problem as much as a technical one.

**Technical improvements:**

**1. Improve the "not found" message:**
Instead of: "Information not found."

Say: "I could not find an answer to your question in the available documents. The documents I searched include Finance reports, HR policies, and Operations manuals. If this topic should be covered, you may want to check with [relevant department] or request the document be added."

**2. Show what was searched:**
Tell the user which documents were searched. This helps them realize the answer may exist in a document that has not been indexed yet.

**3. Suggest related questions:**
If the exact answer is not found, show 2–3 related questions that the system can answer. This demonstrates value and redirects the user.

**4. Identify coverage gaps:**
Log all "not found" queries. Cluster them by topic. Present a report to the document owners:

> "In the last 30 days, 250 users asked about travel reimbursement policies, but no relevant document was found. Recommend adding the reimbursement policy document."

**5. Feedback button:**
Add a "Was this helpful?" button. When users report "not helpful," add those queries to an improvement backlog.

**Process improvement:**

Work with document owners to index missing documents. A RAG system is only as good as its document coverage.

---

### S10. The system is asked to answer from a very recent document not yet indexed

**Question:** A quarterly report was just published and a user asks about it 10 minutes after upload. The system says "not found." The index has not finished. How do you handle this?

**Answer:**

This is a real-time freshness problem.

**Short-term solution – Show indexing status:**
When a user searches and gets no result, check if there are documents with status `indexing_in_progress` that match the query topic.

Show a message: "A new document is currently being indexed. Results should be available in approximately 3 minutes."

**Medium-term solution – Prioritize fresh documents:**
In the ingestion queue, assign high priority to certain document types (e.g., quarterly reports). These jump the queue and get indexed within 2–3 minutes instead of the normal batch window.

**Long-term solution – Two-tier retrieval:**
- Tier 1: Recently uploaded documents (last 24 hours) stored in a fast "hot" index with smaller scope
- Tier 2: Full historical index

When a user queries, search both tiers. Merge results. This ensures fresh documents are searchable quickly even before full indexing completes.

**Architectural consideration – Streaming ingestion:**
Build a streaming pipeline where chunks are inserted into the vector DB as soon as they are processed, rather than waiting for the full document to finish. This means partial search results become available faster.

---

# Summary Reference Card

## Core Technologies

| Component | Technology |
|---|---|
| Orchestration | LangChain |
| LLM | Google Gemini |
| Embedding | text-embedding models (Google, OpenAI, or open-source) |
| Vector DB | FAISS (dev), Pinecone / Weaviate / Qdrant (prod) |
| Document Loaders | LangChain loaders (PDF, DOCX, CSV, HTML) |
| Text Splitter | RecursiveCharacterTextSplitter |
| Evaluation | RAGAS, LangSmith |
| Observability | LangSmith, Datadog, Prometheus + Grafana |

## Difficulty Summary

| Level | Count | Focus |
|---|---|---|
| 🟢 Easy | 12 | Definitions, basic concepts, core components |
| 🟡 Medium | 11 | Design decisions, trade-offs, comparisons |
| 🔴 Hard | 10 | Deep-dive internals, advanced architecture |
| 🔵 Scenario | 10 | Real-world debugging and problem-solving |

## Closing Statement for Interviews

> "The strength of this system is not simply using an LLM. The real engineering value lies in the chunking strategy, embedding quality, metadata-driven filtering, multi-stage retrieval, strict prompt grounding, source citations, access control, and production observability. I describe it not as a chatbot, but as a production-grade RAG platform for enterprise document intelligence — one where every design decision was made with accuracy, trust, scale, and cost in mind."

---

*End of Guide*
