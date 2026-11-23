## Screen 1: Security Dashboard

**Route:** `/dashboard`
**Device Classes:** All
**Purpose:** Provide security and platform administrators with an immediate overview of fleet health, real-time tool invocation volume, blocked security violations, and pending approvals.
**Visual Focus:** High-contrast Fleet Security Status indicator card flanked by real-time metric counters.
**Entry Points:** Navigated directly on login or via main navigation sidebar.
**Exit Points:** [Live Traffic Inspector](02-live-inspector.md), [Approvals Queue](03-approvals-queue.md), [Policy Manager](04-policy-manager.md), [Audit Explorer](05-audit-explorer.md).

### Layout

**Web layout:**
- Top navigation bar with AegisMCP brand logo, environment badge (Production/Staging), cluster selector dropdown, and admin profile menu.
- Hero summary banner: Fleet Threat Level indicator (Normal / Elevated / Lockdown) with the action that changes the state — Emergency Kill-Switch while the fleet is Normal or Elevated, Release Lockdown while a lockdown is in force.
- Metric counter grid (4 columns): Active Agent Connections, 24h Tool Calls, Blocked Violations (24h), Pending Approvals.
- Main content grid (2 columns, 2:1 ratio):
  - Left column: Real-time Invocations vs. Violations trend graph (last 24 hours), followed by Recent Security Incident cards.
  - Right column: Urgent Approvals widget (top 3 pending requests with remaining TTL) and Agent Fleet Status list.

### Components

| Component | Type | Purpose | Available On | Priority | Data Source |
|---|---|---|---|---|---|
| Top bar | Navbar | Displays branding, cluster selection, and user session | Mobile, Tablet, Desktop | — | READ[ConsoleOperatorSession(name, roles, idp)] |
| Threat banner | Panel | Shows global security posture and the lockdown or release action | Mobile, Tablet, Desktop | Secondary | READ[Screen state(threatLevel, fleetStatus)] |
| Metric card grid | Grid | Displays key operational counters | Mobile, Tablet, Desktop | — | READ[Screen state(activeAgentCount, toolCallCount, blockedViolationCount, pendingApprovalCount)] |
| Invocations chart | Card | Visualizes throughput and violation spikes | Mobile, Tablet, Desktop | — | READ[Screen state(throughputBuckets)] |
| Approvals widget | Card | Lists actionable pending approvals with countdowns | Mobile, Tablet, Desktop | Primary | READ[ApprovalTicket(id, agentId, toolName, expiresAt)] |
| Recent incidents | Table | Lists recent blocked prompt injection / chaining violations | Mobile, Tablet, Desktop | — | READ[SecurityViolation(id, agentId, violationType, severity, createdAt)] |

### Overlays

| Overlay | Pattern | Trigger | Behavior | States |
|---|---|---|---|---|
| Emergency Lockdown & Release Confirmation | Dialog | "Emergency Kill-Switch" while the fleet is Normal or Elevated; "Release Lockdown" while a lockdown is in force | States the cluster-wide effect and requires the operator's fresh multi-factor re-confirmation; in an Enterprise production deployment it also requires a second distinct operator or admin to confirm, and a release additionally requires a written reason. Submits the global kill or release and reports a refusal without changing fleet state. | Default, Awaiting Second Approver, Submitting, Error |

### Data Requirements

**Data Needed:**
- System threat status and connected gateway node count.
- Aggregated 24h metrics: total requests, blocked violations, active agents, pending approvals.
- Throughput and violation trend as 24 one-hour buckets covering the trailing 24 hours; each bucket carries its start time, the tool calls intercepted in that hour, and the tool calls blocked in that hour. The dashboard offers no other chart interval.
- High-priority pending approvals list (max 3).
- Recent violation incidents (max 5).
- Live threat-level and pending-approval changes from the same event stream the [Live Traffic Inspector](02-live-inspector.md) subscribes to, applying its `system` and `approval` frames; that screen defines the stream's authentication, heartbeat, event identity, and reconnect bounds.

**Data Flow:**
1. Screen mounts → display skeleton loaders across metric cards and chart panels.
2. Fetch system status, metrics summary, and the 24-bucket timeseries in parallel via REST API.
3. Subscribe to the live event stream shared with the Live Traffic Inspector and apply its `system` and `approval` frames to the threat banner and the pending approval badge.
4. Render loaded cards and interactive trend chart; a dropped stream reconnects on the bounds defined by the Live Traffic Inspector.

### States

| Scope | State | Condition | What the User Sees |
|---|---|---|---|
| Screen | **Loading** | Initial metrics fetching | Skeleton rectangular placeholders across all metric cards and chart |
| Screen | **Loaded** | Metrics successfully retrieved | Full dashboard with live counters and active trend visualization |
| Screen | **Elevated Threat** | Blocked violations exceed baseline threshold | Amber warning banner on threat panel with link to filtered Live Inspector |
| Screen | **Lockdown** | Global kill-switch tripped | High-visibility crimson banner across the top naming the lockdown, carrying the Release Lockdown action for an operator holding `operator` or `admin`, and marking proxy status Quarantined |
| Screen | **Error** | API telemetry endpoint unreachable | Error alert banner with "Retry Connection" button |
| Screen | **Offline** | Browser loses internet connection | Persistent top warning banner: "Offline — telemetry paused" |

**Edge Cases:**
- Zero connected agents (fresh cluster deployment) → Display onboarding banner with link to Kubernetes Helm chart and local CLI setup guides.
- Rapid spike in violations (> 100/min) → Auto-refresh incident panel and sound optional audible alert for on-call operator.
- Release submitted without a reason, with a stale multi-factor re-confirmation, or in Enterprise production without a second distinct operator → the dialog shows the refusal and the lockdown stays in force.
