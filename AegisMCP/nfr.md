# Non-Functional Requirements

## Overview

This document specifies the quantitative quality attributes, security controls, performance budgets, and regulatory compliance targets for AegisMCP. It covers the core high-performance proxy engine, the command-line interface, and the web management console. The system is designed to run as a high-density Kubernetes sidecar or dedicated ingress gateway within enterprise cloud and on-premises environments.

---

## Performance & Scalability

AegisMCP operates inline on the critical execution path between AI agents and backend tools. To prevent developers from bypassing the gateway, strict latency budgets and non-blocking I/O throughput targets are enforced.

| ID | Operation class | Metric | Target (p50 / p95 / p99) | Load assumption | Verification |
|---|---|---|---|---|---|
| NFR-001 | Inline JSON-RPC proxying & policy evaluation | Added latency | 0.8ms / 2.2ms / 4.8ms | 10,000 rps sustained per gateway instance | Load test (Gatling) |
| NFR-002 | Outbound tool response secret & PII scrubbing | Added latency per scrubbed response | 1.2ms / 2.8ms / 5.5ms | 2,000 responses/s admitted on one gateway node at the response-size mix defined below, up to a 1 MB ceiling | Load test on the release's CI runner |
| NFR-003 | Multi-step tool-chaining sequence anomaly check | Evaluation latency | 0.4ms / 0.9ms / 1.8ms | Sliding-window evaluation, 50k sessions | Microbenchmark |
| NFR-004 | Tamper-evident audit ledger append | Commit latency | 0.6ms / 1.5ms / 3.2ms | Asynchronous WAL flush at 10k rps | Benchmark test |
| NFR-005 | Web Console live traffic broadcast | Event delivery | < 80ms / < 120ms / < 200ms | 500 concurrent SSE dashboard connections | Synthetic probe |
| NFR-006 | Local CLI proxy process startup (`aegis-cli`) | Startup time | < 150ms / < 250ms / < 400ms | Local workstation, cold start | CLI benchmark |

NFR-005 is measured from event emission at the gateway to the rendered update on an already-subscribed console; keepalive comments and reconnect attempts are transport liveness and are not counted as delivered events.

NFR-002 is a per-response latency requirement under an offered load, not an aggregate throughput target; the **Throughput & capacity** commitments own aggregate capacity. The 2,000 responses per second is the rate at which completed upstream responses enter outbound scrubbing on one gateway node, and the added latency is the interval from a response's complete upstream body entering the scrubber to its sanitized body becoming available for delivery — the delay the stage adds to a response that would otherwise be forwarded unscanned. The p50, p95, and p99 targets apply to the distribution of every admitted response in the run; they do not apply to a single 1 MB response, and the 1 MB figure is the ceiling of the run's size mix, not the size of every response.

The size mix is part of the requirement: 90% of responses at 4 KB, 8% at 64 KB, 1.75% at 256 KB, and 0.25% at the 1 MB ceiling. A run admits at least 4,000 responses at the stated rate, reports each size class's percentiles separately, and satisfies the requirement only when every admitted response is scrubbed and served rather than dropped or rejected.

**Throughput & capacity:**
- **Peak concurrency:** Minimum 50,000 concurrent active streaming JSON-RPC sessions (HTTP/SSE or WebSocket) per gateway node without thread starvation, utilizing Java 21 Virtual Threads.
- **Data volume:** Sustained processing of 500 MB/s aggregate tool payload throughput with heap memory constrained below 1.5 GB.
- **Scaling trigger:** Sustained CPU utilization > 70% or active connections exceeding 40,000 per node triggers horizontal pod autoscaling (Kubernetes HPA) within 30 seconds.

---

## Availability & Reliability

| ID | Availability target | Monthly error budget | Measurement |
|---|---|---|---|
| NFR-010 | 99.99% monthly for inline proxy path | 4m 23s | Liveness probe on `GET /health` ([FEAT-001](features/FEAT-001.md)) plus client telemetry |
| NFR-011 | 99.95% monthly for Web Console & Audit query APIs | 21m 36s | External monitoring service |

**Recovery objectives:**
- **RTO (Recovery Time Objective):** < 15 seconds for automated container pod restart and traffic re-routing.
- **RPO (Recovery Point Objective):** 0 seconds for cryptographically committed audit ledger entries (synchronous local write-ahead log).

**Degradation — behavior under dependency failure:**
| Failure | Expected behavior | User-visible effect |
|---|---|---|
| Central policy storage unavailable | Fail-closed for unknown routes; serve active in-memory compiled policy cache | Tool execution continues normally for existing policies; new policy sync paused |
| Human approval (Slack/Webhook) down | Buffer requests in durable queue; reject upon 5-minute timeout if unacknowledged | Agent receives `APPROVAL_SERVICE_UNAVAILABLE` error; non-approval tools unaffected |
| Audit export SIEM endpoint unreachable | Spool audit records to local encrypted disk buffer; auto-retry with exponential backoff | No impact on real-time tool execution; log backlog drains when SIEM restores |
| Outbound PII scrubber memory pressure | Stream payloads through sliding fixed-size window; enforce 20MB tool output ceiling | Tool responses exceeding ceiling truncated with `RESPONSE_TOO_LARGE` notice |
| Agent SSE or WebSocket stream lost | End the session, abandon its in-flight calls, and cancel and mark expired any pending approval ticket | The agent reconnects on a new session and reissues the call |

**Maintenance windows:** Zero-downtime rolling upgrades supported via Kubernetes rolling updates and graceful connection draining.

---

## Security

| ID | Control | Target / standard | Notes |
|---|---|---|---|
| NFR-020 | Agent Authentication | mTLS with SPIFFE/SPIRE x509 SVIDs or verified OIDC Bearer JWTs | Unauthenticated connections terminated at TLS/handshake layer |
| NFR-021 | Transport Encryption | TLS 1.3 required (TLS 1.2 minimum acceptable); HSTS enabled | Cleartext HTTP and unencrypted WebSockets prohibited in production |
| NFR-022 | Data at Rest Encryption | AES-256-GCM for persistent audit ledgers, configuration, and secret stores | Encryption keys rotated every 90 days |
| NFR-023 | Secrets Management | Zero hardcoded credentials; dynamic retrieval from HashiCorp Vault or Kubernetes Secrets | In-memory credentials scrubbed after use |
| NFR-024 | Input Validation & Injection Defense | Strict JSON schema enforcement; path traversal and command injection token filtering | Mitigates OWASP LLM03 and command injection vectors |
| NFR-025 | Outbound DLP & Redaction | Automated redaction of AWS, GitHub, GCP, OpenAI tokens, and credit card/SSN patterns | Aho-Corasick multi-pattern scanner with sub-2ms overhead |
| NFR-026 | Audit Tamper-Evidence | Continuous SHA-256 hash-chaining from the genesis block to the sealed head across all log entries | Each sealed entry covers every field it seals — the argument digest and resolving approver identity of a tool interaction, and the actor, action, target, and outcome of an administrative action; a modified, reordered, omitted, or uncovered historical entry fails verification at the first affected index and invalidates every later block. The deployment recalculates the full chain at least once every 24 hours and retains the most recent completed result for reporting |
| NFR-027 | Default Stance | Fail-closed: unmapped agent identities or undeclared tools denied execution | Explicit policy grant required for any tool invocation |
| NFR-028 | Supply Chain Security | Zero critical or high severity CVEs in container base images and dependencies | Weekly automated SBOM scanning and dependency vulnerability gates |
| NFR-029 | Administrative Access | Multi-Factor Authentication (MFA) and RBAC enforced for Web Console access | Session timeout after 15 minutes of inactivity; every console request and event-stream subscription carries the operator session |

---

## Model Quality & Cost

AegisMCP operates as a deterministic security proxy, policy evaluator, and pattern scanner. In version 1.0, all firewall inspection, anomaly detection, and data redaction algorithms execute deterministically in memory without invoking external large language models. Consequently, the core gateway incurs $0 in external model invocation costs and has zero non-deterministic refusal failure modes.

---

## Privacy & Compliance

| ID | Regulation / standard | Applies because | Obligation | Evidence |
|---|---|---|---|---|
| NFR-040 | EU AI Act (Article 14 - Human Oversight) | Governs autonomous AI systems operating in high-risk enterprise settings | Technical mechanisms enabling real-time human intervention and execution halting | Interactive HITL approval records and gate logs |
| NFR-041 | EU AI Act (Article 12 - Record-Keeping) | Requires automatic logging of events throughout AI system lifecycle | Immutable, tamper-evident recording of all tool calls, parameters, and decisions | Cryptographically verified audit ledger exports |
| NFR-042 | SOC 2 Type II | Enterprise customers require independent trust criteria validation | Verifiable access control, change tracking, continuous monitoring, and auditability | Annual SOC 2 Type II attestation report |
| NFR-043 | GDPR / CCPA | Tool outputs may retrieve personal data from internal corporate databases | Automated PII redaction preventing sensitive customer data transmission to LLMs | DLP scan metrics and redaction event logs |

**Data lifecycle:**
| Data class | Retention | Residency | Deletion |
|---|---|---|---|
| Tamper-evident audit ledger | 365 days (statutory default) | Customer Private VPC / Designated Region | Cryptographically sealed archive prior to secure purge |
| Active agent session state | Duration of session + 15 minutes | In-memory RAM only | Instant purge upon session termination |
| Security violation alerts | 90 days | Customer Private VPC / Designated Region | Anonymized and purged after 90 days |
| Admin console access logs | 180 days | Customer Private VPC / Designated Region | Hard delete on expiration |

---

## Measurement & Verification

| ID | Verified by | When | Owner |
|---|---|---|---|
| NFR-001 | Automated Gatling load test suite (latency p99 < 5ms) | Every CI/CD release build | Platform Engineering |
| NFR-002 | Automated load test on the release's CI runner that admits 2,000 responses/s of the declared size mix on one gateway node, asserts the run's p50, p95, and p99, and reports each size class separately | Every CI/CD release build | DevSecOps Engineering |
| NFR-010 | Continuous synthetic liveness and readiness probes across staging and production, using the endpoints defined in [FEAT-001](features/FEAT-001.md) | Continuous (24/7) | Platform Engineering |
| NFR-020 | Automated mTLS and OIDC test harness with invalid credential injection | Every PR build | Security Engineering |
| NFR-025 | Golden test suite of 500+ credential formats and PII edge-case samples | Every PR build | Security Engineering |
| NFR-026 | The gateway's scheduled recalculation covers every sealed block from genesis to the sealed head and completes at least once every 24 hours; `aegis-cli verify-audit` reports the most recent completed result and exits non-zero when that result reports the chain broken | Automated daily scheduled job | Compliance & Audit |
| NFR-028 | Container vulnerability scan (Trivy / Snyk) | Automated on container build | Platform Engineering |
| NFR-040 | End-to-end integration test of Human-in-the-Loop Slack approval workflow | Every CI/CD release build | QA Engineering |
