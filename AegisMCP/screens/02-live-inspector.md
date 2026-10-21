## Screen 2: Live Traffic Inspector

**Route:** `/inspector`
**Platform:** Web
**Presentation:** Page
**Purpose:** Provide real-time streaming inspection of all JSON-RPC 2.0 tool calls, parameter schemas, and security evaluation decisions passing through the gateway.
**Visual Focus:** Live-updating streaming event table with real-time status badges (Allowed, Blocked, Pending HITL).
**Action Hierarchy:** Primary: Pause / Resume Stream; Secondary: Filter by Agent / Decision; Tertiary: Clear Inspection Buffer.
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

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Stream controls | Toolbar | Web | Controls SSE streaming buffer and auto-scroll | Client state |
| Filter bar | Form | Web | Filters live event stream by status, agent, and tool | Client-side filter |
| Traffic stream table | Table | Web | Displays real-time JSON-RPC tool invocations | GET `/api/v1/stream/traffic` (SSE) |
| Event detail drawer | Panel | Web | Shows formatted JSON payloads, rule traces, and redactions | Selected event payload |
| Redaction pill | Badge | Web | Highlights masked secrets or PII tokens in payload | Parsed DLP metadata |

### Data Requirements

**Data Needed:**
- Real-time JSON-RPC event frames streamed over SSE: event ID, timestamp, agent identity, target MCP tool, request arguments, policy decision, latency, outbound redaction summary.
- Active agent identity list for filter autocomplete.

**Data Flow:**
1. Screen mounts → connect to Server-Sent Events stream (`/api/v1/stream/traffic`).
2. Buffer incoming event frames (up to 1,000 items in memory).
3. Append new rows to table; auto-scroll if enabled.
4. User clicks row → populate right detail drawer with formatted syntax-highlighted JSON payload and matched rule metadata.

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Streaming** | Active SSE connection receiving events | Table live-updating; green "Live" indicator pulsing in header |
| **Paused** | Administrator clicked "Pause" | Stream paused indicator; new events buffered in background counter |
| **Empty** | No tool calls during inspection window | Empty-state message: "Waiting for agent tool activity..." with network pulse indicator |
| **Reconnecting** | SSE stream disconnected | Amber banner: "Reconnecting to live traffic stream..." with retry spinner |
| **Filtered Empty** | Active filters match zero current events | "No events match the selected filters" with "Reset Filters" button |

**Edge Cases:**
- High traffic flood (> 5,000 events/second) → Automatically engage visual sampling mode (render summary aggregates, buffer raw events in memory) to prevent browser DOM freezing.
- Selected event payload contains massive 10MB data dump → Render truncated preview with "Download Raw JSON" action instead of crashing syntax highlighter.
