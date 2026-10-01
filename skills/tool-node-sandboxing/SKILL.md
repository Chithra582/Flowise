---
name: tool-node-sandboxing
description: Safely runs custom JavaScript and Python tool functions in sandboxes.
---

# Tool Node Sandboxing

## Overview
Provides a secure isolation environment for executing user-defined JavaScript and Python function nodes, preventing host filesystem tampering or environment variable leakage.

## Key Capabilities
- Ephemeral worker execution with CPU and memory limits.
- Network access restriction and whitelist filtering.
- Input/output parameter validation against JSON schema definitions.
