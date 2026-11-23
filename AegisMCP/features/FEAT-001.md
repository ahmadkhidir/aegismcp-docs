## FEAT-001: Zero-Trust JSON-RPC Streaming Reverse Proxy

**Priority:** P0
**Effort:** L
**Depends On:** None

**User Story:**
As Marcus Vance, I want AegisMCP to transparently intercept JSON-RPC 2.0 streaming traffic over HTTP/SSE, WebSocket, and Stdio between agents and MCP servers, so that all tool interactions pass through an enforceable security boundary with sub-5ms latency.

**Description:**
The core proxy engine sits inline between AI agent clients (such as Claude Desktop, Cursor, or autonomous orchestrators) and backend Model Context Protocol tool servers. It accepts client connections, parses streaming JSON-RPC 2.0 messages in real time, evaluates security policies, and forwards authorized requests to target tool servers while streaming responses back to the client.

Agents reach the gateway over one of three remote transports on its TLS listener, and the local wrapper carries the fourth. Each remote transport binds the verified agent identity at connection or session establishment, the wrapper applies the local policy to its local agent identity, and all of them apply the same policy, chaining, DLP, approval, and audit behavior to each request:
- **HTTPS request/response.** `POST /mcp/v1/rpc` accepts one JSON-RPC 2.0 request per call and answers with that request's response.
- **Server-Sent Events.** `GET /mcp/v1/sse` opens a session stream whose first event is `endpoint`, carrying the absolute message URL for the session (`/mcp/v1/messages?sessionId=<session-id>`). The agent posts one JSON-RPC request per call to that URL and receives `202 Accepted` once the request is accepted for processing; the response or error object, and every server notification for the session, arrives on the stream as an `event: message` frame whose data is the JSON-RPC object. The stream declares `retry: 3000` and carries a `: keepalive` comment every 15 seconds. The message URL requires the same verified identity as the stream. Closing the stream ends the session: queued and in-flight calls are abandoned, a pending approval ticket is cancelled, and the message URL answers `404` from then on.
- **WebSocket.** `GET /mcp/v1/ws` upgrades the connection after the credential is verified, and a failed verification answers `401` without upgrading. Every text frame carries exactly one complete JSON-RPC message in either direction, requests may be pipelined, and each response repeats its request's `id` regardless of the order responses arrive in. A binary frame closes the connection with `1003`. The gateway pings every 30 seconds and closes with `1001` once 45 seconds pass without a pong.
- **Stdio.** `aegis-cli wrap` owns the agent-facing standard I/O of a local child server ([FEAT-003](FEAT-003.md)). Messages are UTF-8 JSON-RPC objects, one per line, and a message never contains an embedded newline. A line that does not parse as a JSON-RPC object is answered with `-32700` on the same channel and is never forwarded. Standard output carries protocol messages only; inspection output goes to standard error. End of input on the agent channel closes the child's input, and the session ends after the in-flight calls complete.

Each upstream MCP server is configured with one of the same three transports. The gateway opens or reuses that upstream connection, forwards the request, and matches each response to its request by JSON-RPC `id`.

The same listener serves two unauthenticated probe endpoints that expose no tenant, credential, host, path, or exception detail. `GET /health` answers `200` with `{"status":"UP"}` for as long as the process serves requests, and is the liveness signal. `GET /ready` answers `200` with `{"status":"UP","checks":{"configuration":{"status":"UP"},"policy":{"status":"UP"},"auditLedger":{"status":"UP"}},"timestamp":"<ISO-8601>"}` once startup has completed. It answers `503` with `"status":"DOWN"`, a top-level `reason` from `STARTING`, `CONFIGURATION_INVALID`, `POLICY_INVALID`, `LEDGER_UNWRITABLE`, or `SHUTTING_DOWN`, and every failed check marked `"DOWN"`, whenever the node must not receive agent traffic.

**Acceptance Criteria:**
- **Given** an agent connected to AegisMCP via HTTP/SSE or WebSocket
  **When** the agent sends a valid `tools/call` JSON-RPC 2.0 request
  **Then** AegisMCP parses the payload, evaluates inline policy, and forwards the request to the designated MCP tool server within 5ms P99 overhead.
- **Given** an authenticated agent
  **When** it opens `GET /mcp/v1/sse`
  **Then** the stream answers `200`, its first event is `endpoint` naming that session's message URL, and a `: keepalive` line reaches the agent within 15 seconds.
- **Given** an open SSE session
  **When** the agent posts one JSON-RPC request to the session's message URL
  **Then** the gateway answers `202` and the response arrives on the stream as an `event: message` frame with the request `id` unchanged.
- **Given** an SSE session whose stream has closed
  **When** a request is posted to its message URL
  **Then** the gateway answers `404` and forwards nothing upstream.
- **Given** an authenticated agent
  **When** it upgrades `GET /mcp/v1/ws`
  **Then** each text frame carries exactly one JSON-RPC message, pipelined requests are answered independently with their own `id`, and a binary frame closes the connection with `1003`.
- **Given** a WebSocket connection that stops answering pings
  **When** 45 seconds pass without a pong
  **Then** the gateway closes it with `1001` and discards the session.
- **Given** a local developer setup using the Stdio transport
  **When** an agent client spawns `aegis-cli wrap` around an MCP server process
  **Then** bidirectional standard I/O streams are intercepted and inspected without buffering delays.
- **Given** a stdio line that is not a valid JSON-RPC object
  **When** it reaches the wrapper
  **Then** a `-32700` error is written to the agent channel, nothing is forwarded upstream, and no inspection text appears on standard output.
- **Given** an authorized tool execution response from the upstream MCP server
  **When** the response streams back through the proxy
  **Then** AegisMCP delivers the result to the agent client preserving exact JSON-RPC message identifiers.
- **Given** an agent with a missing, expired, or untrusted credential
  **When** it opens an SSE stream, posts to a message URL, or requests a WebSocket upgrade
  **Then** the gateway answers `401` and establishes no session.
- **Given** a running gateway whose configuration, policy state, and audit ledger are healthy
  **When** `GET /ready` is called without a credential
  **Then** it answers `200` with `"status":"UP"` and each check `"UP"`.
- **Given** a gateway still starting, holding a configured policy document that failed to compile, unable to append to the audit ledger, or shutting down
  **When** `GET /ready` is called
  **Then** it answers `503` with `"status":"DOWN"`, a `reason` naming that condition, the failed check marked `"DOWN"`, and no host, path, credential, or exception text.
- **Given** a gateway that fails its readiness check
  **When** `GET /health` is called
  **Then** it still answers `200` with `{"status":"UP"}` while the process serves requests.

**Edge Cases & Failure States:**
- Malformed JSON or non-spec JSON-RPC message → Return standard JSON-RPC error response (`-32700 Parse error` or `-32600 Invalid Request`) and abort upstream forwarding.
- Upstream MCP server crash or unhandled timeout → Return JSON-RPC internal error (`-32603`) to the agent, terminate the upstream socket, and log an upstream connection failure event.
- SSE stream closes while a tool call is in flight or awaiting approval → Abandon the call, cancel the approval ticket, refuse the session's message URL, and record the cancellation in the audit ledger and Web Console approval history ([FEAT-006](FEAT-006.md)).
- Upstream response carries an `id` that matches no in-flight request in the session → Discard the response and never deliver it to another session.
- Two requests on one streaming connection carry the same `id` → Reject the second with `-32600` and forward only the first.
- WebSocket ping is answered after the 45-second window → The connection is already closed; the late pong is ignored and no session is restored.
