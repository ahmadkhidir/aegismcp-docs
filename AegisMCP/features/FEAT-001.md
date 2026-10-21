## FEAT-001: Zero-Trust JSON-RPC Streaming Reverse Proxy

**Priority:** P0
**Effort:** L
**Depends On:** None

**User Story:**
As Marcus Vance, I want AegisMCP to transparently intercept JSON-RPC 2.0 streaming traffic over HTTP/SSE, WebSocket, and Stdio between agents and MCP servers, so that all tool interactions pass through an enforceable security boundary with sub-5ms latency.

**Description:**
The core proxy engine sits inline between AI agent clients (such as Claude Desktop, Cursor, or autonomous orchestrators) and backend Model Context Protocol tool servers. It accepts client connections, parses streaming JSON-RPC 2.0 messages in real time, evaluates security policies, and forwards authorized requests to target tool servers while streaming responses back to the client.

**Acceptance Criteria:**
- **Given** an agent connected to AegisMCP via HTTP/SSE or WebSocket
  **When** the agent sends a valid `tools/call` JSON-RPC 2.0 request
  **Then** AegisMCP parses the payload, evaluates inline policy, and forwards the request to the designated MCP tool server within 5ms P99 overhead.
- **Given** a local developer setup using Stdio transport
  **When** an agent client spawns `aegis-proxy` wrapping an MCP server process
  **Then** bidirectional standard I/O streams are intercepted and inspected without buffering delays.
- **Given** an authorized tool execution response from the upstream MCP server
  **When** the response streams back through the proxy
  **Then** AegisMCP delivers the result to the agent client preserving exact JSON-RPC message identifiers.

**Edge Cases & Failure States:**
- Malformed JSON or non-spec JSON-RPC message → Return standard JSON-RPC error response (`-32700 Parse error` or `-32600 Invalid Request`) and abort upstream forwarding.
- Upstream MCP server crash or unhandled timeout → Return JSON-RPC internal error (`-32603`) to the agent, terminate the upstream socket, and log an upstream connection failure event.
