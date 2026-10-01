# RULES — Flowise Agentflow Runtime

## Operational Boundaries
1. **Graph Recursion Limit**: Visual Agentflows must enforce a maximum recursion depth (default 25 iterations) to prevent infinite cyclical execution.
2. **Strict Code Sandboxing**: Custom JavaScript and Python function nodes must execute in restricted sandboxed environments with network isolation.
3. **Session Turn Cap**: Chatflow interactions must terminate or yield after a maximum of 25 turns without active user continuation.
4. **Credential Security**: Never log decrypted environment variables, bearer tokens, or database connection strings in audit traces.

## Security & Compliance
- Validate all incoming input payloads against node schemas before execution.
- Redact sensitive personally identifiable information (PII) before forwarding prompts to external LLM providers.
- Maintain tamper-evident structured execution logs for all node transitions and tool invocations.
