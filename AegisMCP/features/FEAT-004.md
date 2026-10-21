## FEAT-004: Behavioral Tool-Chaining Anomaly Detector

**Priority:** P0
**Effort:** L
**Depends On:** FEAT-001, FEAT-002

**User Story:**
As Elena Rostova, I want the gateway to track multi-step tool execution sequences across an agent session and block dangerous sequences (such as reading internal secrets followed by external network requests), so that indirect prompt injection cannot exfiltrate corporate data.

**Description:**
A stateful sequence inspection engine that maintains sliding-window context across an agent's active session. It compares consecutive tool invocations against an anomaly state machine (e.g. `read_resource` or `filesystem:read` followed within 60 seconds by `fetch:post` or `curl:exec`). When a prohibited sequence pattern is matched, execution is halted immediately.

**Acceptance Criteria:**
- **Given** an active agent session executing a series of permitted tools
  **When** the agent invokes a tool classified as a private data read followed by a tool classified as an egress network action
  **Then** the second tool call is blocked, the session is placed in a quarantined state, and an urgent security alert is dispatched.
- **Given** a normal development workflow invoking permitted consecutive tools
  **When** the tool sequence conforms to standard allowed operational patterns
  **Then** the calls execute without interruption.
- **Given** a session exceeding the maximum configured idle timeout
  **When** the agent resumes tool activity
  **Then** session sequence memory is reset cleanly to prevent false-positive sequence correlations across disparate tasks.

**Edge Cases & Failure States:**
- Rapid concurrent tool calls dispatched in parallel from an asynchronous agent → Atomically update session state machine using concurrent thread-safe state without race conditions.
- Memory pressure from excessive concurrent agent sessions → Evict oldest terminated session state windows using an LRU policy while preserving active session state.
