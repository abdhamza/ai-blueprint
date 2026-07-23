# RAG Interview Questions & Answers

> A practical interview guide for Retrieval-Augmented Generation (RAG), FAISS, embeddings, and vector search.
>
> These answers are designed for junior-to-mid level AI/ML Engineer interviews and are based on implementing a simple RAG pipeline using FAISS.

---

# Table of Contents

1. Walk me through your RAG pipeline.
2. Why do we split documents into chunks?
3. Why not embed the whole PDF?
4. What is an embedding?
5. Why did you use FAISS?
6. Is FAISS a database?
7. How does FAISS find similar vectors?
8. Why retrieve only Top-K chunks?
9. What happens if K is too small?
10. What happens if K is too large?
11. Why did you choose your chunk size?
12. Why use chunk overlap?
13. What similarity metric did you use?
14. What metadata did you store?
15. What happens when new documents arrive?
16. Why separate ingestion and querying?
17. Why use RAG instead of fine-tuning?
18. What are the limitations of your RAG system?
19. If your dataset grows to millions of documents, would you still use FAISS?
20. How do you evaluate a RAG system?
21. What trade-offs did you encounter while building your RAG pipeline?

---

## 1. Walk me through your RAG pipeline.

### Answer

> Our RAG pipeline consisted of two phases: **ingestion** and **retrieval**.
>
> During ingestion, we loaded documents from PDFs, split them into smaller chunks using a RecursiveCharacterTextSplitter, generated embeddings for each chunk using an embedding model, and stored those embeddings along with their metadata in a FAISS index.
>
> During retrieval, when a user asked a question, we generated an embedding for the query, searched the FAISS index to retrieve the Top-K most relevant chunks, and passed those chunks along with the user's question to the LLM. The LLM then generated the final answer using the retrieved context.

---

## 2. Why do we split documents into chunks?

### Answer

> Embedding an entire document creates one vector representing many different topics. This reduces retrieval accuracy because one embedding cannot accurately represent every concept.
>
> Splitting documents into smaller chunks allows each chunk to represent one focused idea. During retrieval, FAISS can return only the relevant chunks instead of an entire document.
>
> Chunking also helps because LLMs have limited context windows.

---

## 3. Why not embed the whole PDF?

### Answer

> A single embedding for an entire PDF mixes together all the topics contained in the document.
>
> For example, if a PDF discusses AI, databases, networking, and cloud computing, and the user asks only about FAISS, the embedding still represents all topics together.
>
> Smaller chunks create more meaningful embeddings, improving retrieval accuracy.

---

## 4. What is an embedding?

### Answer

> An embedding is a dense numerical vector that represents the semantic meaning of text.
>
> Instead of comparing words directly, vector databases compare embeddings.
>
> Sentences with similar meanings produce embeddings that are close together in vector space.

Example:

```
"What is Artificial Intelligence?"

↓

[0.24, 0.81, 0.11, ...]

```

```
"Explain AI"

↓

[0.23, 0.79, 0.14, ...]

```

These vectors are close because both sentences have similar meanings.

---

## 5. Why did you use FAISS?

### Answer

> FAISS is an open-source vector similarity search library developed by Meta.
>
> It efficiently stores embeddings and performs nearest-neighbor searches much faster than comparing every vector manually.
>
> It is lightweight, fast, free, and ideal for local development or medium-sized datasets.

---

## 6. Is FAISS a database?

### Answer

> Not exactly.
>
> FAISS is primarily a vector similarity search library.
>
> It stores vectors and retrieves similar vectors efficiently but does not provide production database features such as:
>
> - Authentication
> - Replication
> - Distributed storage
> - Metadata filtering
> - Automatic scaling
>
> Those capabilities are available in vector databases like Pinecone, Milvus, Qdrant, and Weaviate.

---

## 7. How does FAISS find similar vectors?

### Answer

> When a user asks a question:
>
> 1. Generate an embedding for the query.
> 2. Compare that embedding with stored document embeddings.
> 3. Compute similarity using a distance metric.
> 4. Return the nearest vectors (Top-K).
>
> Those vectors correspond to the most relevant document chunks.

---

## 8. Why retrieve only Top-K chunks?

### Answer

> The LLM does not need the entire knowledge base.
>
> Retrieving only the most relevant chunks:
>
> - Reduces token usage
> - Lowers API cost
> - Reduces latency
> - Removes irrelevant information
>
> Typical values are **k = 3** or **k = 5**.

---

## 9. What happens if K is too small?

### Answer

> Important information may be missing.
>
> If the answer spans multiple chunks but only one chunk is retrieved, the LLM may generate an incomplete or incorrect answer.

---

## 10. What happens if K is too large?

### Answer

> Too many chunks increase prompt size, token cost, and latency.
>
> The LLM also has to process more irrelevant information, which can reduce answer quality.

---

## 11. Why did you choose your chunk size?

### Answer

> I used approximately **500 characters** with **100 characters of overlap**.
>
> This size was large enough to preserve context while remaining small enough to generate meaningful embeddings.
>
> The optimal chunk size depends on the document type and requires experimentation.

---

## 12. Why use chunk overlap?

### Answer

> Without overlap, sentences may be split across chunk boundaries.
>
> Overlap duplicates a small amount of text between neighboring chunks, preserving context and improving retrieval quality.

---

## 13. What similarity metric did you use?

### Answer

> FAISS supports several similarity metrics.
>
> In many LangChain implementations, cosine similarity is achieved by normalizing embeddings and using an inner-product index. Alternatively, FAISS can use L2 (Euclidean) distance.
>
> The main goal is to retrieve vectors that are closest in semantic space.

---

## 14. What metadata did you store?

### Answer

> Along with each embedding, I stored metadata such as:
>
> - Source document
> - Page number
> - Chunk ID
>
> Metadata makes it easier to display citations and trace where retrieved information came from.

---

## 15. What happens when new documents arrive?

### Answer

> New documents pass through the same ingestion pipeline:
>
> 1. Load document
> 2. Split into chunks
> 3. Generate embeddings
> 4. Add embeddings to the FAISS index
>
> If the dataset changes significantly, rebuilding the index from scratch may be preferable.

---

## 16. Why separate ingestion and querying?

### Answer

> Generating embeddings is computationally expensive, while querying happens frequently.
>
> By separating these phases, embeddings are generated only once.
>
> During querying, the application simply loads the existing FAISS index, making responses much faster.

---

## 17. Why use RAG instead of fine-tuning?

### Answer

> RAG is better when knowledge changes frequently because updating the knowledge base only requires adding or re-indexing documents.
>
> Fine-tuning modifies the model's parameters, which is expensive and time-consuming.
>
> Fine-tuning is better for changing model behavior, while RAG is better for providing up-to-date knowledge.

---

## 18. What are the limitations of your RAG system?

### Answer

Some limitations include:

- Retrieval quality depends on chunking strategy.
- Poor embeddings reduce retrieval accuracy.
- Choosing an inappropriate Top-K value affects answer quality.
- FAISS is not distributed and is less suitable for very large production systems.
- If relevant chunks are not retrieved, the LLM may generate incomplete or incorrect answers.

---

## 19. If your dataset grows to millions of documents, would you still use FAISS?

### Answer

> For local development and medium-sized datasets, FAISS is an excellent choice because it is lightweight and extremely fast.
>
> For production systems with millions of documents, I would consider vector databases such as Pinecone, Milvus, Qdrant, or Weaviate because they provide distributed storage, metadata filtering, replication, persistence, and automatic scaling.

---

## 20. How do you evaluate a RAG system?

### Answer

I evaluate both retrieval and generation.

### Retrieval Metrics

- Recall@K
- Precision@K
- MRR (Mean Reciprocal Rank)
- Hit Rate

### Generation Metrics

- Groundedness
- Faithfulness
- Answer Accuracy
- Completeness

### Operational Metrics

- Latency
- Token Cost
- Retrieval Time
- User Satisfaction

---

## 21. What trade-offs did you encounter while building your RAG pipeline?

### Answer

> The biggest trade-offs were chunk size, chunk overlap, and the number of retrieved chunks.
>
> Smaller chunks improve retrieval precision but may lose context.
>
> Larger chunks preserve context but reduce retrieval accuracy.
>
> Increasing Top-K provides more information to the LLM but also increases token cost and latency.
>
> Choosing the right balance depends on the application's requirements and usually requires experimentation.

---

## Interview Tips

## Explain your design decisions.

Interviewers are often more interested in *why* you made a decision than *what* library you used.

For example:

- Why FAISS instead of Pinecone?
- Why chunk size of 500?
- Why overlap of 100?
- Why Top-K = 3?

Being able to justify these choices demonstrates practical understanding.

---

## Mention trade-offs.

Whenever possible, discuss trade-offs instead of presenting one solution as universally correct.

Example:

> "FAISS is excellent for local development because it's lightweight and fast, but for large-scale production systems I'd choose a distributed vector database such as Qdrant or Pinecone."

---

## Know the complete RAG flow.

```
User Question
      │
      ▼
Generate Query Embedding
      │
      ▼
Similarity Search (FAISS)
      │
      ▼
Retrieve Top-K Chunks
      │
      ▼
Prompt + Retrieved Context
      │
      ▼
Large Language Model
      │
      ▼
Generated Answer
```

---

# Final Advice

Building a simple RAG project yourself is one of the best ways to prepare for AI interviews.

If you can confidently explain:

- Why chunking is necessary
- How embeddings work
- Why FAISS was chosen
- How similarity search retrieves context
- Why Top-K matters
- The trade-offs in chunking and retrieval

you'll be well-prepared for many RAG-related interview questions.
