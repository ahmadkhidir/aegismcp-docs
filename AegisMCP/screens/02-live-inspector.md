## Screen 2: Live Traffic Inspector

**Route:** `/inspector`
**Device Classes:** All
**Purpose:** Provide real-time streaming inspection of all JSON-RPC 2.0 tool calls, parameter schemas, and security evaluation decisions passing through the gateway.
**Visual Focus:** Live-updating streaming event table with real-time status badges (Allowed, Blocked, Pending HITL).
**Entry Points:** [Security Dashboard](01-dashboard.md), or left navigation rail.
**Exit Points:** [Security Dashboard](01-dashboard.md), [Approvals Queue](03-approvals-queue.md), [Policy Manager](04-policy-manager.md).

### Layout

**Web layout:**
- Persistent header with stream control buttons (Pause/Play, Auto-scroll toggle, Buffer counter) and quick-search input.
- Filter toolbar: Multi-select dropdowns for Agent Identity (SPIFFE/OIDC), Decision Status (Allowed / Blocked / Pending), Tool Name, and Latency range.
- Master-detail split view:
  - Left panel (60% width): Live streaming table displaying Timestamp, Status Badge, Agent ID, Target Tool, Latency, and Action trigger.
  - Right panel (40% width): Selected event inspection drawer displaying formatted JSON-RPC request and response AST, parameter diffs, matched policy rule IDs, and DLP redaction markers.

### Components

| Component | Type | Purpose | Available On | Priority | Data Source |
|---|---|---|---|---|---|
| Stream controls | Toolbar | Pauses and resumes the stream, toggles auto-scroll, and clears the inspection buffer | Mobile, Tablet, Desktop | Primary | READ[Screen state(streamPaused, autoScroll, bufferCount)]; WRITE[Screen state(streamPaused, autoScroll)] |
| Filter bar | Form | Filters live event stream by status, agent, and tool | Mobile, Tablet, Desktop | Secondary | READ[AgentIdentity(id, name)]; WRITE[Screen state(activeFilters)] |
| Traffic stream table | Table | Displays real-time JSON-RPC tool invocations | Mobile, Tablet, Desktop | — | READ[Screen state(trafficBuffer)] |
| Event detail drawer | Panel | Shows formatted JSON payloads, rule traces, and redactions | Mobile, Tablet, Desktop | — | READ[Screen state(selectedEvent)] |
| Redaction pill | Badge | Highlights masked secrets or PII tokens in payload | Mobile, Tablet, Desktop | — | READ[Screen state(redactionSummary)] |

### Data Requirements

**Data Needed:**
- Real-time event frames from one Server-Sent Events connection at `/api/v1/stream/traffic`. The stream carries three named event types:
  - `traffic`: one frame per intercepted tool call, carrying event ID, timestamp, agent identity, target MCP tool, request arguments, policy decision (`ALLOWED`, `BLOCKED`, `PENDING_APPROVAL`), latency, matched policy rule IDs, and outbound redaction summary.
  - `approval`: one frame per approval ticket created, approved, rejected, or expired, carrying ticket ID, status, and expiry.
  - `system`: one frame whenever the fleet threat level or the pending approval count changes.
  The inspector renders `traffic` frames; `approval` and `system` frames keep the dashboard's live indicators current.
- Event IDs increase monotonically for the life of the gateway node. A `: keepalive` comment arrives every 15 seconds while the stream is idle.
- The console subscribes with the operator's OIDC access token, presented in the `Authorization` header with the `Bearer` scheme on an authenticated fetch-based event reader. Any of the four roles may subscribe, and the stream is read-only.
- Active agent identity list for filter autocomplete.

**Data Flow:**
1. Screen mounts → the console opens an authenticated fetch reader against the stream with the operator's OIDC access token in the `Authorization` header, and the server's first frame sets a 3-second retry directive.
2. Buffer incoming `traffic` frames (up to 1,000 items in memory).
3. Append new rows to the table; auto-scroll if enabled. Apply `approval` and `system` frames to the live indicators.
4. On disconnect, reconnect with exponential backoff from 1 second to 30 seconds without reloading the page. Events produced during the gap are not replayed, so the pending approval count is re-read from the approvals queue after reconnection.
5. User clicks row → populate right detail drawer with formatted syntax-highlighted JSON payload and matched rule metadata.

### States

| Scope | State | Condition | What the User Sees |
|---|---|---|---|
| Traffic stream table | **Streaming** | Active SSE connection receiving events | Table live-updating; green "Live" indicator pulsing in header |
| Screen | **Paused** | Administrator clicked "Pause" | Stream paused indicator; new events buffered in background counter |
| Traffic stream table | **Empty** | No tool calls during inspection window | Empty-state message: "Waiting for agent tool activity..." with network pulse indicator |
| Screen | **Reconnecting** | SSE stream disconnected | Amber banner: "Reconnecting to live traffic stream..." with retry spinner; the client retries with exponential backoff from 1 second to 30 seconds |
| Traffic stream table | **Filtered Empty** | Active filters match zero current events | "No events match the selected filters" with "Reset Filters" button |

**Edge Cases:**
- High traffic flood (> 5,000 events/second) → Automatically engage visual sampling mode (render summary aggregates, buffer raw events in memory) to prevent browser DOM freezing.
- Selected event payload contains massive 10MB data dump → Render truncated preview with "Download Raw JSON" action instead of crashing syntax highlighter.
