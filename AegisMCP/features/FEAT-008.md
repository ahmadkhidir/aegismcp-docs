## FEAT-008: Web Admin Security Console & Live Traffic Inspector

**Priority:** P1
**Effort:** L
**Depends On:** FEAT-001, FEAT-007

**User Story:**
As Elena Rostova, I want a web dashboard displaying real-time streaming tool call telemetry, blocked violation alerts, and pending Human-in-the-Loop requests, so that security teams have complete operational visibility over autonomous agents.

**Description:**
A browser-based management console that provides visual monitoring and administration for AegisMCP deployments. It features a live streaming inspector for JSON-RPC traffic, an interactive approval queue for pending Human-in-the-Loop tool requests, policy configuration viewers, and security analytics charts.

**Acceptance Criteria:**
- **Given** an authenticated administrator logged into the web console
  **When** navigating to the Live Traffic Inspector
  **Then** incoming tool calls stream in real time via Server-Sent Events, displaying agent ID, target tool, execution status (Allowed/Blocked/Pending), and latency.
- **Given** one or more tool calls waiting for human authorization
  **When** an administrator opens the Approvals view
  **Then** the console displays tool name, requesting agent, parameter diffs, and risk justification, with one-click "Approve" and "Reject" buttons.
- **Given** an auditor reviewing historical security violations
  **When** filtering the Audit Explorer by date range, agent identity, or violation type
  **Then** matching tamper-evident audit records are rendered with export capability (JSON/CSV).

**Edge Cases & Failure States:**
- Browser loses network connection to console SSE stream → Display reconnection banner, automatically retry with exponential backoff, and re-sync pending approval list upon reconnect.
- Unauthorized user attempting to access admin console without proper RBAC role → Deny access, redirect to SSO login, and log an unauthorized access attempt.
