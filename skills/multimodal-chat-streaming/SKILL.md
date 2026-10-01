---
name: multimodal-chat-streaming
description: Streams tokens, handles file uploads, and manages chat memory.
---

# Multimodal Chat Streaming

## Overview
Powers real-time user chat interactions with token-by-token Server-Sent Events (SSE) streaming, multimodal image and audio attachment processing, and sliding-window memory buffers.

## Key Capabilities
- Sub-100ms first-token streaming response latency.
- File attachment parsing and inline vision model dispatch.
- Multi-session chat history persistence with automatic summarization.
