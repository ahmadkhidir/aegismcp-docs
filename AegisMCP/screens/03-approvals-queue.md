## Screen 3: Approvals Queue

**Route:** `/approvals`
**Platform:** Web
**Presentation:** Page
**Purpose:** Provide an actionable human-in-the-loop review inbox where administrators can inspect, authorize, or reject suspended high-risk agent tool calls before execution.
**Visual Focus:** Pending approval item cards displaying countdown timers and parameter risk summaries.
**Action Hierarchy:** Primary: Approve Request; Secondary: Reject Request; Tertiary: Batch Approve (with multi-select).
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

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Tab bar | Tabs | Web | Switches between pending queue and historical reviews | Client state |
| Approval card | Card | Web | Displays details, parameter diffs, and action buttons | GET `/api/v1/approvals/pending` |
| Countdown timer | Badge | Web | Visualizes remaining TTL before automatic expiration | Client timer based on `expires_at` |
| Parameter inspector | Table | Web | Shows formatted input arguments and risk annotations | Item payload |
| Rejection dialog | Modal | Web | Captures mandatory human rejection reason | User input |

### Overlays

| Overlay | Pattern | Trigger | Behavior | States |
|---|---|---|---|---|
| Rejection Reason Dialog | Dialog | "Reject" button on an approval card | Modal prompt requiring the reviewer to select or type a rejection reason for the agent | Default, Submitting, Error |

### Data Requirements

**Data Needed:**
- List of pending approval tickets: ticket ID, agent identity, tool name, argument payload, triggering policy rule ID, creation timestamp, expiration timestamp (`expires_at`), agent prompt context.

**Data Flow:**
1. Screen mounts → fetch all active pending approvals (`GET /api/v1/approvals/pending`).
2. Subscribe to approval event updates via SSE to update list immediately when an action is approved via Slack or expired.
3. User clicks "Approve" → send `POST /api/v1/approvals/{id}/approve`; on success, remove card with slide-out animation.
4. User clicks "Reject" → open Rejection Reason Dialog; on submit, send `POST /api/v1/approvals/{id}/reject`.

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Loading** | Fetching pending queue | Skeleton card placeholders with pulsing animated lines |
| **Loaded** | One or more requests pending | List of approval cards ordered by urgency (least TTL remaining first) |
| **Empty** | Zero items in pending queue | Clean illustration + "All clear — no pending agent actions requiring human approval" |
| **Expired** | Timer reaches 00:00 while card is on screen | Card automatically fades out and transitions to Expired tab with notice |
| **Error** | API request failure | Error banner with "Retry Loading Queue" action |

**Edge Cases:**
- Two administrators review the same ticket concurrently and one approves before the other → Second administrator's action returns "Ticket already resolved by [User]" and refreshes row.
- Agent cancels session while human is reviewing card → Card displays "Agent session cancelled by client" and disables action buttons.
