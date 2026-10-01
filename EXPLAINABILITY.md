# EXPLAINABILITY — Flowise Agentflow Runtime

> **Admissibility & Transparency Report for OpenGAP / Agent Passport**  
> *Agent Name:* Flowise Agentflow Runtime (`flowise-agentflow-runtime`)  
> *Specification:* OpenGAP v0.1.0  
> *Domain:* Developer Tools / Visual Multi-Agent Orchestration  

---

## 1. Overview & Operational Purpose

Flowise Agentflow Runtime is an open-source visual workflow execution engine designed to orchestrate complex multi-agent architectures, retrieval-augmented generation (RAG) pipelines, and autonomous tool integrations. Its primary operational purpose is to empower developers to build, test, and deploy production-grade LLM applications visually, translating modular canvas nodes into deterministic, inspectable Directed Acyclic Graphs (DAGs).

By decoupling orchestration logic from individual model vendors and providing sandboxed tool execution, Flowise Agentflow Runtime ensures transparent execution order, eliminates vendor lock-in, and enforces strict security and memory boundaries across heterogeneous multi-agent teams.

---

## 2. How the Agent Decides (Decision-Making Logic)

Flowise Agentflow Runtime operates across a deterministic, multi-stage decision pipeline:

```
[Stage 1: Ingestion & Schema Parse] ──> [Stage 2: DAG Compilation & Check] ──> [Stage 3: Vault Credential Resolve]
                                                                                               │
                                                                                               ▼
[Stage 6: Output Stream & Telemetry] <── [Stage 5: Memory Update & Audit]  <── [Stage 4: Sandboxed Node Execution]
```

### 2.1 Ingestion & Canvas Schema Parsing
- **Decision:** The runtime parses incoming chat payloads or API requests and retrieves the serialized visual canvas definition matching the target workflow ID.
- **Rules:** Validate that all required node parameters, connections, and input schemas are present and valid before execution begins.

### 2.2 DAG Compilation & Cycle Validation
- **Decision:** Compile canvas nodes and connecting edges into a topological execution plan, resolving node dependencies and execution order.
- **Rules:** If a cyclical dependency is detected, verify whether it matches a permitted iterative supervisor pattern; reject unconstrained loops exceeding 25 recursion turns.

### 2.3 Credential Vault Resolution
- **Decision:** Securely retrieve and inject necessary API credentials (LLM providers, vector stores, external databases) into corresponding node workers.
- **Rules:** Decrypt credentials only in ephemeral in-memory worker contexts. Never write raw keys or tokens to persistent logs or response payloads.

### 2.4 Sandboxed Node Execution & Tool Dispatch
- **Decision:** Execute graph nodes sequentially or in parallel according to topological order, routing outputs of upstream nodes to downstream inputs.
- **Rules:** Execute custom code scripts within isolated, resource-constrained sandbox workers. If an external tool invocation fails, trigger designated fallback nodes or yield structured error states.

---

## 3. Data Flow & Boundary Privacy

The runtime enforces strict boundary isolation between visual canvas management, credential storage, and runtime node execution.

| Component / Boundary | Data Received | Processing & Retention | Destination / External Transmission |
|---|---|---|---|
| API Gateway | User queries, session tokens, file uploads | Ephemeral in-memory parsing; session-scoped retention | Internal graph runner |
| Credential Vault | Encrypted API keys, database connection strings | AES-256 encrypted at rest; decrypted in memory only | Upstream provider SDKs |
| Sandboxed Worker | Script bodies, node input parameters | Isolated V8 / container execution; purged post-run | Internal execution context |
| Audit Logger | Node execution states, latency metrics, token counts | Structured JSON logging to local disk or logging sink | Local audit filesystem |

Flowise Agentflow Runtime complies with operational security and privacy standards:
- **No Cloud Data Exfiltration:** The runtime operates completely self-hosted or inside customer cloud infrastructure with zero third-party telemetry beaconing.
- **Epistemic Isolation:** Conversational memory stores and scratchpads are strictly isolated by unique session IDs, preventing cross-tenant context bleed.
- **Sanitized Model Payloads:** Private system credentials and authorization headers are stripped from prompt templates before dispatch to upstream model APIs.
- **Data Minimization:** Workflow nodes only receive input fields explicitly declared in their incoming edge mappings.

---

## 4. Known Limitations & Failure Modes

Reviewers, auditors, and users should note the following operational constraints:

1. High-Concurrency Memory Saturation
   - *Limitation:* Executing multiple parallel vector ingestion jobs or complex multi-agent debates can consume substantial host RAM.
   - *Mitigation:* The runtime enforces process concurrency limits, streaming chunk batching, and configurable worker memory ceilings.

2. Upstream Vector Store Latency Spikes
   - *Limitation:* Latency spikes or network timeouts in external vector database providers can stall downstream LLM generation.
   - *Mitigation:* Implement client-side connection pooling, configurable query timeout thresholds (default 5000ms), and fallback retrieval mechanisms.

3. Sandbox Execution Timeouts for Custom Scripts
   - *Limitation:* Long-running custom Python or JavaScript scripts may exceed execution budgets and fail midway.
   - *Mitigation:* Enforce a strict execution timeout (default 30 seconds) on sandboxed nodes with graceful error propagation to supervisory nodes.

4. Token Context Window Exhaustion in Long Chatflows
   - *Limitation:* Retaining full conversation histories across lengthy chat sessions can exceed model context limits.
   - *Mitigation:* Use Conversation Summary Buffer memory nodes that automatically summarize earlier turns once a token threshold is reached.

---

## 5. Verification, Safety & Human Oversight

Flowise Agentflow Runtime incorporates robust verification, safety gates, and human oversight controls across every layer of execution:

- **Real-Time Human Approval Gate:** Any workflow action involving sensitive mutations (such as SQL database writes or external webhooks) can be configured with human-in-the-loop pause gates requiring explicit operator sign-off.
- **Emergency Session Interrupt:** Administrators and operators can issue an immediate abort signal via REST API or UI to terminate running execution graphs instantly.
- **Step Quota Guardrails:** Strict recursion caps (maximum 25 iterations) and session turn ceilings prevent runaway multi-agent deliberation loops.
- **Structured Audit Logging:** Every node entry, parameter transformation, tool execution, and LLM call is captured in structured JSON logs for auditing and debugging.
