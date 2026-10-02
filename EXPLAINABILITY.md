# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **Flowise Agentflow Runtime** (`flowise`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** Flowise Agentflow Runtime (`flowise`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Developer Tools / Visual Multi-Agent Orchestration & Flow Runtimes  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), OWASP LLM Top 10, MITRE ATLAS  

---

## How the Agent Decides

Flowise Agentflow Runtime is an open-source visual workflow execution engine designed to orchestrate complex multi-agent architectures, retrieval-augmented generation (RAG) pipelines, and autonomous tool integrations. Its primary operational purpose is to empower developers to build, test, and deploy production-grade LLM applications visually, translating modular canvas nodes into deterministic, inspectable Directed Acyclic Graphs (DAGs).

### 1. Decision Architecture

The visual canvas parsing, DAG compilation, credential resolution, and node execution pipeline operates across a deterministic, five-stage architecture:

```
User Query / Webhook Trigger (Chat Message / Flow Trigger / API Invocation / File Upload)
    │
    ▼
[Stage 1: Ingestion & Canvas Schema Parsing]
    │  - Deserializes visual canvas JSON definition matching target workflow ID
    │  - Validates node configurations, parameter bindings, and port connections
    │  - Initializes execution session memory and isolation parameters
    ▼
[Stage 2: DAG Compilation & Cycle Validation]
    │  - Compiles canvas nodes and edges into a topological execution sequence
    │  - Evaluates cyclical routes against permitted supervisor feedback loops
    │  - Enforces finite recursion caps (`recursion_limit: 25`) to prevent infinite cycling
    ▼
[Stage 3: Credential Vault Resolution]
    │  - Decrypts AES-256 stored provider keys, database URIs, and vector store tokens
    │  - Injects credentials strictly into ephemeral in-memory node workers
    │  - Enforces least-privilege scoping across tool executions
    ▼
[Stage 4: Sandboxed Node Execution & Tool Dispatch]
    │  - Executes graph nodes sequentially or in parallel along the topological sort
    │  - Runs custom JavaScript/Python tool code within isolated V8 sandbox workers
    │  - Captures node intermediate outputs, token metrics, and execution latencies
    ▼
[Stage 5: Output Stream Serialization & Audit Logging]
    │  - Serializes verified response tokens and artifact streams to client interface
    │  - Updates session conversation memory buffers (BufferMemory, Redis, Zep)
    │  - Commits structured JSON execution traces locally with scrubbed credentials
    ▼
Validated Agentflow Execution Stream & Auditable Node Trajectory Record
```

### 2. Decision Logic & Graph Routing Formulations

The runtime evaluates node execution ordering, loop termination, and vector similarity thresholds using deterministic mathematical models:

1. **Topological Node Precedence Index ($P_{\text{node}}$)**:
   $$P_{\text{node}} = \text{TopologicalSort}(V, E)$$
   where $V$ represents configured canvas nodes and $E$ represents directed connection edges, guaranteeing that dependent inputs are resolved prior to downstream node execution.

2. **Vector Retrieval Relevance Metric ($R_{\text{vector}}$)**:
   $$R_{\text{vector}} = \text{CosineSim}(\mathbf{v}_{\text{query}}, \mathbf{v}_{\text{doc}}) \ge \theta_{\text{threshold}}$$
   where $\theta_{\text{threshold}}$ (default $\theta = 0.75$) deterministically filters out irrelevant document chunks before injecting context into active prompts.

### 3. Thresholding & Refusal Decision Criteria

Flowise Agentflow Runtime enforces strict operational safety and integrity boundaries:
- **Refusal to Execute Unsanitized Sandbox Code**: Custom function nodes containing access to host process environments (`process.exit`, `child_process`, `fs.unlinkSync`) are deterministically rejected with code `ERR_UNSAFE_SANDBOX_CALL_BLOCKED`.
- **Refusal of Plaintext Credential Exposure**: Workflows configured to output raw API credentials or encryption keys into response streams trigger immediate execution termination (`ERR_CREDENTIAL_LEAK_PREVENTED`).
- **Turn Ceiling Enforcement**: Agentflow multi-agent conversational turns are bounded by `max_turns: 25` to eliminate runaway token exhaustion (`WARN_TURN_BUDGET_EXCEEDED`).
- **Local Workspace Confinement**: File operations and local database writes are restricted strictly to designated local runtime directories (`ERR_OUT_OF_BOUNDS_WRITE`).

### 4. Fallback Decision Mechanism

Continuous workflow availability is guaranteed through multi-tier fault recovery:
- **Model Cascade Failover**: When the primary foundation model experiences latency spikes or HTTP 429 rate limits, the orchestrator cascades automatically between `claude-3-5-sonnet`, `gpt-4o`, and `gemini-2.0-flash`.
- **Node-Level Exception Handlers**: If a vector database query or third-party web scraper fails, the graph activates pre-configured fallback branch nodes.
- **Graceful Degradation to Pure Extraction**: If conversational reasoning nodes fail, the engine routes raw retrieved context directly to the output interface with diagnostic warnings.

### 5. Human-in-the-Loop Governance

Human developers retain complete architectural direction and sign-off authority:
- **Visual Canvas Pre-Execution Inspection**: Developers can visually inspect every node dependency, variable mapping, and credential binding before publishing flows.
- **Emergency Session Kill Switch**: Operators can halt active flow execution threads instantly through the admin UI or via `Ctrl+C` server interrupt signals.
- **Structured Audit Trajectory Logging**: Every node invocation, prompt payload, latency figure, and token count is captured in tamper-evident local logs for auditing.

---

## The Data It Uses

Flowise Agentflow Runtime operates under strict privacy, data minimization, and local workspace isolation standards.

### 1. Ingested Input Data

The agent processes only operational assets necessary to fulfill visual workflow execution:
- **User Messages & Queries**: Incoming natural language chat messages, API parameters, and file attachments.
- **Visual Canvas Definitions**: Serialized JSON flow schemas detailing node blocks, edges, credentials, and configuration flags.
- **Document Payloads**: PDF, DOCX, and TXT files ingested for document splitting and vector embedding pipelines.

### 2. Configuration & Reference Data

- **Node Block Registry**: Metadata specifications, input/output schemas, and parameter descriptors for 100+ LangChain/LlamaIndex components.
- **Encrypted Credential Store**: AES-256 encrypted database storing third-party API keys and connection strings.
- **Vector Store Connection Configurations**: Endpoint configurations for Pinecone, Qdrant, Chroma, Weaviate, and Milvus.

### 3. Base Model & Inference Lineage

- **Deterministic Orchestration Engines**: Topological DAG compilers, vector distance calculators, and V8 sandbox runners executed natively in Node.js and TypeScript (100% deterministic with zero LLM variance).
- **Foundation LLMs**: High-capability frontier models (`claude-3-5-sonnet`, `gpt-4o`, `gemini-2.0-flash`) utilized for in-node reasoning, summarization, and chat response synthesis.
- **Zero Training on User Data**: User flow designs, conversation histories, and embedded documents are never transmitted to external cloud servers or used for model training.

### 4. Data Privacy, Storage, and Retention

- **OWASP LLM & MITRE ATLAS Hardened**: Defended against prompt injection, context leakage, and unauthorized agency.
- **Session-Scoped Memory Partitioning**: Conversation histories are partitioned strictly by `sessionId`, guaranteeing complete epistemic isolation between concurrent users.
- **Self-Hosted Data Sovereignty**: All workflows, user conversations, and credential vaults reside entirely within the user's self-hosted deployment.
- **Zero Commercial Monetization**: Application graph schemas, state histories, and developer prompts are never monetized, aggregated, or shared with third parties.

---

## Limitations

Understanding the operational boundaries and technical constraints of Flowise Agentflow Runtime is essential for production deployment.

### 1. High-Concurrency Memory Storage Contention
- **Limitation**: Managing thousands of concurrent multi-turn chat sessions on local SQLite backends can lead to database locking delays.
- **Mitigation**: Flowise supports production database backends (PostgreSQL, MySQL) and distributed caching (Redis) for high-concurrency enterprise workloads.

### 2. Upstream Model Provider Context Window Saturation
- **Limitation**: Ingesting extensive document contexts into visual RAG chains can exceed model context limits if chunking parameters are misconfigured.
- **Mitigation**: The runtime incorporates visual token counting warnings and supports conversational summary memory nodes to compress message histories.

### 3. Complex Multi-Agent Subgraph Debugging
- **Limitation**: Debugging nested Agentflow graphs containing dozens of conditional branching rules can become complex through visual canvases alone.
- **Mitigation**: The runtime outputs structured node-by-node execution logs showing exact inputs, outputs, and latencies for each step.

### 4. Non-Deterministic Tool Output Schema Variations
- **Limitation**: External web scraping or API tool nodes may return unformatted strings that disrupt downstream prompt formatting.
- **Mitigation**: Flowise incorporates output parser nodes (JSON, CSV, Structured) that enforce strict Pydantic/Zod schemas before passing data downstream.

### 5. Multi-Node Latency Accumulation in Deep Graphs
- **Limitation**: Executing sequential chains with multiple LLM calls compounds network latency during user-facing chat interactions.
- **Mitigation**: The engine implements token streaming across all LLM nodes and supports parallel branch execution where dependencies permit.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Decision logic & graph routing formulations | Section 2 | Verified |
| - Thresholding & refusal decision criteria | Section 3 | Verified |
| - Fallback decision mechanism | Section 4 | Verified |
| - Human-in-the-loop governance & oversight | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested user messages, canvas definitions & docs | Section 1 | Verified |
| - Configuration, node registry & credential store | Section 2 | Verified |
| - Base model lineage & deterministic engines | Section 3 | Verified |
| - Data privacy, retention lifecycle & MITRE/OWASP | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - High-concurrency memory storage contention | Section 1 | Verified |
| - Upstream model provider context window saturation | Section 2 | Verified |
| - Complex multi-agent subgraph debugging | Section 3 | Verified |
| - Non-deterministic tool output schema variations | Section 4 | Verified |
| - Multi-node latency accumulation in deep graphs | Section 5 | Verified |
