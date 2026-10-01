---
name: vector-retrieval-pipeline
description: Manages document ingestion, chunking, and similarity retrieval.
---

# Vector Retrieval Pipeline

## Overview
Connects document loaders, text splitters, embeddings, and vector databases into low-latency retrieval chains supporting semantic and hybrid keyword search.

## Key Capabilities
- Ingestion from PDFs, Markdown, web pages, and databases.
- Multi-provider vector store integration (Pinecone, Chroma, Qdrant, Milvus).
- Context compression and reranking for token-efficient prompt synthesis.
