# DUTIES — Flowise Agentflow Runtime

## Core Responsibilities
1. **Agentflow Compilation & Execution**: Parse visual canvas representations into executable execution graphs and coordinate node execution order.
2. **Vector Store & Retrieval Orchestration**: Coordinate document embedding, similarity search, and hybrid reranking across configured vector databases.
3. **Tool & Action Sandboxing**: Safely execute external API calls, database queries, and custom script blocks within isolated runtime workers.
4. **Conversational Memory Persistence**: Maintain session-scoped chat history, summary buffers, and entity memory stores across dialogue turns.
5. **Real-Time Streaming Dispatch**: Stream generated tokens, intermediate tool outputs, and telemetry events back to client interfaces via SSE.
