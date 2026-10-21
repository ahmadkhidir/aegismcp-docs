## Screen 1: Security Dashboard

**Route:** `/dashboard`
**Platform:** Web
**Presentation:** Page
**Purpose:** Provide security and platform administrators with an immediate overview of fleet health, real-time tool invocation volume, blocked security violations, and pending approvals.
**Visual Focus:** High-contrast Fleet Security Status indicator card flanked by real-time metric counters.
**Action Hierarchy:** Primary: Review Pending Approvals (when pending > 0); Secondary: Open Live Inspector; Tertiary: Export Daily Audit Summary.
**Entry Points:** Navigated directly on login or via main navigation sidebar.
**Exit Points:** [Live Traffic Inspector](02-live-inspector.md), [Approvals Queue](03-approvals-queue.md), [Policy Manager](04-policy-manager.md), [Audit Explorer](05-audit-explorer.md).

### Layout

**Web layout:**
- Top navigation bar with AegisMCP brand logo, environment badge (Production/Staging), cluster selector dropdown, and admin profile menu.
- Hero summary banner: Fleet Threat Level indicator (Normal / Elevated / Lockdown) with quick-access Emergency Kill-Switch button.
- Metric counter grid (4 columns): Active Agent Connections, 24h Tool Calls, Blocked Violations (24h), Pending Approvals.
- Main content grid (2 columns, 2:1 ratio):
  - Left column: Real-time Invocations vs. Violations trend graph (last 24 hours), followed by Recent Security Incident cards.
  - Right column: Urgent Approvals widget (top 3 pending requests with remaining TTL) and Agent Fleet Status list.

### Components

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Top bar | Navbar | Web | Displays branding, cluster selection, and user session | Auth context & cluster config |
| Threat banner | Panel | Web | Shows global security posture and emergency lockdown trigger | GET `/api/v1/system/status` |
| Metric card grid | Grid | Web | Displays key operational counters | GET `/api/v1/metrics/summary` |
| Invocations chart | Card | Web | Visualizes throughput and violation spikes | GET `/api/v1/metrics/timeseries` |
| Approvals widget | Card | Web | Lists actionable pending approvals with countdowns | GET `/api/v1/approvals/pending?limit=3` |
| Recent incidents | Table | Web | Lists recent blocked prompt injection / chaining violations | GET `/api/v1/audit/violations?limit=5` |

### Overlays

| Overlay | Pattern | Trigger | Behavior | States |
|---|---|---|---|---|
| Emergency Lockdown Confirmation | Dialog | "Emergency Kill-Switch" button | Modal dialog requiring MFA verification and explicit confirmation to trip cluster-wide circuit breaker | Default, Submitting, Error |

### Data Requirements

**Data Needed:**
- System threat status and connected gateway node count.
- Aggregated 24h metrics: total requests, blocked violations, active agents, pending approvals.
- High-priority pending approvals list (max 3).
- Recent violation incidents (max 5).

**Data Flow:**
1. Screen mounts → display skeleton loaders across metric cards and chart panels.
2. Fetch system status and metrics summary in parallel via REST API.
3. Open lightweight SSE stream for live approval count badge and threat level changes.
4. Render loaded cards and interactive trend chart.

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Loading** | Initial metrics fetching | Skeleton rectangular placeholders across all metric cards and chart |
| **Loaded** | Metrics successfully retrieved | Full dashboard with live counters and active trend visualization |
| **Elevated Threat** | Blocked violations exceed baseline threshold | Amber warning banner on threat panel with link to filtered Live Inspector |
| **Lockdown** | Global kill-switch tripped | High-visibility Crimson banner across top; proxy status marked Quarantined |
| **Error** | API telemetry endpoint unreachable | Error alert banner with "Retry Connection" button |
| **Offline** | Browser loses internet connection | Persistent top warning banner: "Offline — telemetry paused" |

**Edge Cases:**
- Zero connected agents (fresh cluster deployment) → Display onboarding banner with link to Kubernetes Helm chart and local CLI setup guides.
- Rapid spike in violations (> 100/min) → Auto-refresh incident panel and sound optional audible alert for on-call operator.
