## Screen 3: Approvals Queue

**Route:** `/approvals`
**Device Classes:** All
**Purpose:** Provide an actionable human-in-the-loop review inbox where administrators can inspect, authorize, or reject suspended high-risk agent tool calls before execution.
**Visual Focus:** Pending approval item cards displaying countdown timers and parameter risk summaries.
**Entry Points:** [Security Dashboard](01-dashboard.md), [Live Traffic Inspector](02-live-inspector.md), or deep link from Slack notification.
**Exit Points:** [Security Dashboard](01-dashboard.md), [Live Traffic Inspector](02-live-inspector.md), [Audit Explorer](05-audit-explorer.md).

### Layout

**Web layout:**
- Header with pending items counter, batch-selection toolbar, and auto-refresh toggle.
- Status tab bar: Pending (count badge), Approved History, Rejected / Expired History.
- Vertical list of Approval Cards. Each card features:
  - Header: Requesting Agent Identity badge, Target Tool Name, Submission Timestamp, and Remaining TTL Countdown Timer (e.g. `03:45 remaining`).
  - Body: Reason for Review (e.g. "Triggered rule: High-value database mutation"), Parameter Diff View (highlighting sensitive arguments), and Agent Intent Description.
  - Action Footer: "Reject" button (outlined red), "Approve" button (solid green), and "View Session Context" link.

### Components

| Component | Type | Purpose | Available On | Priority | Data Source |
|---|---|---|---|---|---|
| Tab bar | Tabs | Switches between the pending queue and the approved and rejected-or-expired history | Mobile, Tablet, Desktop | — | READ[Screen state(activeTab)] |
| Approval card | Card | Displays details, parameter diffs, agent intent, and the approve or reject actions | Mobile, Tablet, Desktop | Primary | READ[ApprovalTicket(id, agentId, toolName, argumentsPayload, agentIntent, triggerPolicyId, createdAt, expiresAt)] |
| Countdown timer | Badge | Visualizes remaining TTL before automatic expiration | Mobile, Tablet, Desktop | — | READ[ApprovalTicket(expiresAt)] |
| Parameter inspector | Table | Shows formatted input arguments and risk annotations | Mobile, Tablet, Desktop | — | READ[ApprovalTicket(argumentsPayload)] |
| Rejection dialog | Modal | Captures mandatory human rejection reason | Mobile, Tablet, Desktop | — | WRITE[ApprovalTicket(status, reviewedBy, rejectionReason, processedAt)] |
| History list | Table | Lists resolved tickets for the selected history tab with resolution time, reviewer, and rejection reason | Mobile, Tablet, Desktop | — | READ[ApprovalTicket(id, agentId, toolName, status, reviewedBy, rejectionReason, processedAt)] |

### Overlays

| Overlay | Pattern | Trigger | Behavior | States |
|---|---|---|---|---|
| Rejection Reason Dialog | Dialog | "Reject" button on an approval card | Modal prompt requiring the reviewer to select or type a rejection reason for the agent | Default, Submitting, Error |

### Data Requirements

**Data Needed:**
- Pending approval tickets: ticket ID, agent identity, tool name, argument payload, triggering policy rule ID, agent intent (null when the call carried none), creation timestamp, expiration timestamp (`expires_at`).
- Resolved tickets for the selected history tab, newest resolution first: ticket ID, agent identity, tool name, argument payload, status, triggering policy rule ID, creation and expiry timestamps, resolving reviewer, rejection reason, and resolution timestamp.

**Data Flow:**
1. Screen mounts → fetch all active pending approvals (`GET /api/v1/approvals/pending`).
2. Subscribe to approval event updates via SSE to update list immediately when an action is approved via Slack or expired.
3. User switches to a history tab → fetch that tab's resolved tickets for its status (approved, rejected, or expired).
4. User clicks "Approve" → send `POST /api/v1/approvals/{id}/approve`; on success, remove card with slide-out animation.
5. User clicks "Reject" → open Rejection Reason Dialog; on submit, send `POST /api/v1/approvals/{id}/reject`.

### States

| Scope | State | Condition | What the User Sees |
|---|---|---|---|
| Screen | **Loading** | Fetching pending queue | Skeleton card placeholders with pulsing animated lines |
| Screen | **Loaded** | One or more requests pending | List of approval cards ordered by urgency (least TTL remaining first) |
| Screen | **Empty** | Zero items in pending queue | Clean illustration + "All clear — no pending agent actions requiring human approval" |
| Approval card | **Expired** | Timer reaches 00:00 while card is on screen | Card automatically fades out and transitions to Expired tab with notice |
| History list | **History Loading** | Fetching the selected history tab | Skeleton rows beneath the active history tab |
| History list | **History Empty** | Selected history tab has no resolved tickets | "No resolved requests in this status yet" with the active tab named |
| Screen | **Error** | API request failure | Error banner with "Retry Loading Queue" action |

**Edge Cases:**
- Two administrators review the same ticket concurrently and one approves before the other → Second administrator's action returns "Ticket already resolved by [User]" and refreshes row.
- Agent cancels session while human is reviewing card → Card displays "Agent session cancelled by client" and disables action buttons.
