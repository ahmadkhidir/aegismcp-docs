# Release & Rollout Strategy

## Overview

**Release cadence:** Continuous integration to Development; weekly pre-release validation in Staging; bi-weekly scheduled release trains to Production; on-demand emergency hotfix releases.
**Deployment model:** Blue-Green and rolling upgrades with graceful connection draining for Kubernetes sidecars and ingress gateways.
**Release owner:** Platform Release Lead (Engineering Lead).
**Change approval:** Dual sign-off required for production releases: Platform Lead (infrastructure & latency verification) and DevSecOps Lead (security & policy verification).

---

## Environments & Promotion

The promotion pipeline advances builds through the environment chain defined in [architecture](architecture.md).

| Environment | Purpose | Data | Promotion Gate | Approver |
|---|---|---|---|---|
| Development | Feature development and unit testing | Synthetic mock tool requests | Automated CI unit tests, static analysis, and Trivy container scan pass | PR Author & Tech Lead |
| Staging | Pre-release validation and integration testing | Anonymized staging tools, test agent sessions | Automated Gatling load suite passes ([NFR-001](nfr.md)), audit verification clean ([NFR-026](nfr.md)), 48h soak without SEV-1/2 | Release Lead |
| Production | Live enterprise agent proxying | Production data, real audit ledger | 100% completion of Go-Live Checklist (Section 5) | Platform Lead & DevSecOps Lead |

**Promotion rule:** Builds advance strictly one environment at a time. The immutable container image hash validated in Staging is promoted to Production with zero code changes or rebuilds.

---

## Rollout Phases & Feature Flags

Production releases follow phased traffic exposure with defined soak durations and automated health verification.

| Phase | Audience | Duration | Exit Criteria |
|---|---|---|---|
| Canary | 5% of internal agent connections | 3 days | Zero SEV-1/2 incidents; P99 latency within budget (< 5ms per [NFR-001](nfr.md)) |
| Early Access | Opt-in enterprise design partners | 10 days | Error rate < 0.05%; zero unhandled protocol parsing crashes |
| General Availability | 100% of production traffic | — | Full fleet operation under standard SLA ([NFR-010](nfr.md)) |

### Feature Flags & Kill Switches

Every sensitive gateway subsystem is wrapped in a dynamic feature flag that can be toggled without container restarts.

| Flag | Feature | Default | Rollout Progression | Kill Switch Behavior |
|---|---|---|---|---|
| `enable_chaining_anomaly_detector` | [FEAT-004](features/FEAT-004.md) | Disabled | Canary → 50% → 100% | Disables multi-step sequence tracking; gateway reverts to stateless tool policy evaluation |
| `enable_outbound_dlp_scrubber` | [FEAT-005](features/FEAT-005.md) | Disabled | Canary → 25% → 100% | Disables streaming token redaction; streams raw tool outputs upstream without payload buffering |
| `enable_hitl_approval_gateway` | [FEAT-006](features/FEAT-006.md) | Disabled | Opt-in → 100% | Suspended execution bypassed; tools execute or fail-closed based on emergency override policy |
| `enable_dynamic_schema_sanitizer` | [FEAT-012](features/FEAT-012.md) | Disabled | Beta → 100% | Reverts to basic JSON type checking without deep SQL injection and command token filtering |

**Flag lifecycle:** Feature flags are permanently removed from the codebase within two release cycles after achieving 100% general availability soak without incident.

---

## Rollback & Roll-forward

Automated health monitors continuously evaluate runtime telemetry. When a rollback trigger breaches its defined threshold, the deployment automatically reverts to the previous stable release.

| Trigger | Threshold | Action | Owner |
|---|---|---|---|
| Proxy Added Latency | P99 exceeds 5ms ([NFR-001](nfr.md)) for 3 consecutive minutes | Automated pod rollback via Kubernetes rolling update revert | On-Call Platform Engineer |
| Protocol Error Rate | JSON-RPC 5xx or parser crash rate > 0.5% for 5 minutes | Automated rollback to previous container image tag | On-Call Platform Engineer |
| Audit Ledger Integrity | Cryptographic SHA-256 hash mismatch detected during write | Divert traffic to standby node; halt non-audit writes; trigger [operations](operations.md) `RB-04` | On-Call Platform Engineer |
| Security Policy False Positives | Legitimate tool call rejection rate > 3x baseline within 10 minutes | Revert active policy configuration to previous Git commit | DevSecOps Lead |

**Rollback procedure:**
1. Automated Kubernetes deployment rollback (`kubectl rollout undo deployment/aegis-gateway`) restores previous container image within < 30 seconds.
2. In-flight agent connections are drained gracefully over a 15-second termination window.
3. The persistent audit ledger write-ahead log (WAL) is strictly append-only and backward-compatible across adjacent releases, requiring zero database down-migrations.

---

## Go-Live Checklist

A production release cannot deploy until every gate below is certified:

- [ ] All `P0` launch features from [features](features.md) pass automated unit and acceptance tests.
- [ ] Added proxy latency verified under 5ms P99 at 10,000 RPS ([NFR-001](nfr.md)).
- [ ] Security vulnerability scan clean: zero critical or high CVEs in base images ([NFR-028](nfr.md)).
- [ ] Telemetry event taxonomy and conversion tracking verified in [analytics](analytics.md).
- [ ] Automated rollback drill executed and validated in Staging environment.
- [ ] Operational runbooks updated and on-call rotation active in [operations](operations.md).
- [ ] Web Admin Console navigation and live stream performance verified in [screens](screens.md).
- [ ] Stakeholders and enterprise design partners notified with release window and maintenance notice.

---

## Communication

- **Release notes:** Published to the customer documentation portal and enterprise changelog prior to rollout.
- **Status page:** Incident notifications and maintenance windows posted to the public status page per [operations](operations.md) protocols.
- **Stakeholder updates:** Platform and DevSecOps leads briefed 48 hours prior to scheduled production release trains.
- **Post-release review:** Retrospective held 3 business days post-GA to verify operational metrics against exit criteria.
