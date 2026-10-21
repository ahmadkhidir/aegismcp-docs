# Support & Operations

## Overview

AegisMCP operates under a hybrid support and operational model. Open-core Community deployments are self-managed with peer and documentation support, while Team Pro and Enterprise tiers receive dedicated operational monitoring, incident response, and SLA-backed engineering escalation. Production operations are managed jointly by Platform Engineering (gateway uptime and performance), DevSecOps (threat response and policy governance), and Compliance Officers (audit integrity). Custom tool development and prompt engineering are out of scope for product support.

---

## Support Model & SLAs

| Tier | Target Audience | Channel | First Response | Resolution Target | Escalation Path |
|---|---|---|---|---|---|
| Community | Individual developers, OSS users | GitHub Discussions & Issues | Community / Best effort | Next release cycle | Community forum |
| Team Pro | Growth startups, mid-tier engineering teams | In-app support & shared Slack | < 4 business hours | < 1 business day | Support Lead → Platform Engineering |
| Enterprise | Mission-critical enterprise fleets | Dedicated 24/7 Slack/Teams + PagerDuty | < 1 hour (24/7 for SEV-1) | Per enterprise SLA | Dedicated Technical Account Manager |

**Out of scope:** Custom third-party MCP tool server coding, internal AI model prompt engineering, and on-premises hardware provisioning.
**Self-serve first:** Connection errors, YAML syntax validation, and local CLI wrapping issues are deflected to automated CLI linter diagnostics (`aegis-cli lint`) and documentation runbooks.

---

## Admin & Governance

**Roles and permissions:**
| Role | Can view | Can change | Approval required for |
|---|---|---|---|
| Platform Engineer | Gateway telemetry, active sessions, system health | Concurrency limits, HPA scaling, cluster config | Emergency Global Kill-Switch ([FEAT-010](features/FEAT-010.md)) |
| DevSecOps Engineer | Traffic streams, violation alerts, audit records | Security policies ([FEAT-002](features/FEAT-002.md)), approval tickets ([FEAT-006](features/FEAT-006.md)) | Production policy deployment |
| Compliance Auditor | Cryptographic ledger, compliance reports, verify proofs | Generate compliance exports ([FEAT-011](features/FEAT-011.md)) | Subject deletion / DSR requests |
| System Admin | All tenant metadata, user accounts, SSO/RBAC | Role assignments, billing tier, cluster binding | Tenant deletion |

**Governance controls:**
- **Least privilege:** All administrative operations are role-scoped via enterprise SSO and RBAC; shared administrator credentials are strictly prohibited.
- **Audit trail:** Every administrative action (policy deployment, human approval decision, emergency kill-switch activation) is appended to the tamper-evident cryptographic ledger ([FEAT-007](features/FEAT-007.md)) and retained for 365 days ([NFR-041](nfr.md)).
- **Break-glass procedure:** Tripping the Emergency Global Kill-Switch requires multi-factor authentication (MFA) and dual-operator authorization in Enterprise production clusters.

---

## Billing Operations

Workflows execute the commercial policies defined in [monetization](monetization.md):

| Trigger | Detection | Action | Owner | SLA |
|---|---|---|---|---|
| Failed subscription payment | Stripe webhook notification | Enter 14-day grace period; send automated notifications on days 1, 3, 7, and 14 | Billing Ops | Automated; downgrade on day 15 |
| Dispute / chargeback filed | Payment processor notification | Suspend disputed tenant tier; submit evidence pack (audit access logs and terms acceptance) | Billing Ops | Within 48 hours of notice |
| Refund request | Support ticket | Evaluate against [monetization](monetization.md) 14-day policy; issue credit or payment refund | Support Lead | 2 business days |
| Suspected fraudulent card use | Automated risk rule (velocity or country mismatch) | Immediate account hold, notify account owner, require identity verification | Risk Ops | Immediate hold |
| Enterprise seat expansion | Usage exceeds contracted agent fleet threshold | Notify account executive to initiate contract amendment | Account Manager | 3 business days |

---

## Incident & Escalation

| Severity | Definition | Response Target | Communication Cadence |
|---|---|---|---|
| SEV-1 | Core gateway offline or prompt injection bypass causing unauthorized data exfiltration | Page on-call immediately; active response < 15 minutes | Status page updated within 30 minutes; hourly stakeholder briefings |
| SEV-2 | Gateway latency exceeding SLA budget ([NFR-001](nfr.md) p99 > 5ms) or Slack HITL webhook integration failure | Page on-call; active response < 1 hour | Status banner updated if user-visible; update every 2 hours |
| SEV-3 | Minor Web Console UI glitch or non-blocking CLI logging issue | Assign during standard business hours; response < 1 business day | Jira ticket updates |

**On-call rotation:** 24/7 follow-the-sun rotation managed via PagerDuty across Platform Engineering and DevSecOps.
**Postmortem requirement:** Blameless postmortem published within 3 business days for all SEV-1 and SEV-2 incidents, tracking root cause, timeline, and corrective actions to completion. Rollback procedures follow [architecture](architecture.md).

---

## Operational Runbooks

| Runbook | Trigger / Covers | Owner |
|---|---|---|
| `RB-01: Gateway Latency Spike Mitigation` | Added latency exceeds 5ms ([NFR-001](nfr.md)); checks Java 21 Virtual Thread saturation and triggers HPA scale-out | Platform Engineering |
| `RB-02: Active Prompt Injection Containment` | Critical violation alert; immediately quarantines agent ID and severs compromised upstream tool connections | DevSecOps Engineering |
| `RB-03: Slack Approval Webhook Outage Fallback` | Slack webhook failure; reroutes pending tickets to Web Console Approvals Queue ([screens](screens.md)) | DevSecOps Engineering |
| `RB-04: Cryptographic Ledger Discrepancy Audit` | SHA-256 hash mismatch detected; identifies damaged block, isolates WAL storage, alerts compliance | Compliance & Security |
| `RB-05: Emergency Global Lockdown & Reset` | Rogue agent loop or zero-day vulnerability; trips cluster-wide circuit breaker and manages staged recovery | Platform Engineering |
| `RB-06: Data Subject Request (DSR) Execution` | GDPR/CCPA subject erasure request; anonymizes reviewer identifiers across audit and analytics stores ([schema](schema.md)) | Compliance Officer |

---

## Measurement & Verification

| Check | Method | When |
|---|---|---|
| Support SLA Compliance | Automated ticket turnaround analysis against target thresholds | Monthly |
| Administrative Audit Completeness | Verification of administrative action logs against cryptographic ledger entries | Monthly |
| Disaster Recovery & Kill-Switch Drill | Simulated emergency lockdown and failover dry-run in staging environment | Quarterly |
| Billing & Dunning Reconciliation | Verification of Stripe webhook handling against active entitlement status | Monthly |
