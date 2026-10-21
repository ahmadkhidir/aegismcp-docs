# Analytics & Observability

## Overview

AegisMCP emits structured telemetry to observe gateway performance, detect emerging AI security threats, measure developer adoption funnels, and provide operators with real-time operational health signals. We track telemetry specifically to inform four critical decisions:
1. We track tool call latency and virtual thread concurrency to determine when to scale gateway pods before latency degrades ([NFR-001](nfr.md)).
2. We track blocked violation spikes and anomaly sequence triggers to automatically tune policy rules and identify active prompt injection campaigns ([FEAT-004](features/FEAT-004.md)).
3. We track Human-in-the-Loop approval turnaround times to optimize reviewer routing and prevent workflow bottlenecks ([FEAT-006](features/FEAT-006.md)).
4. We track open-core usage patterns to prioritize enterprise features and identify high-volume accounts ready for commercial tier upgrade ([monetization](monetization.md)).

---

## Event Taxonomy

**Common properties on every event:** `event_id`, `occurred_at` (UTC, ISO-8601), `gateway_id`, `agent_id` (pseudonymous), `session_id`, `environment` (`production` / `staging`), `runtime_version`, `cluster_id`.

| ID | Event | Trigger | Key properties | Feature |
|---|---|---|---|---|
| EVT-001 | `gateway_started` | Gateway instance initialized and policy loaded | `gateway_id`, `runtime_version`, `policy_version`, `transport_modes` | [FEAT-001](features/FEAT-001.md), [FEAT-002](features/FEAT-002.md) |
| EVT-002 | `agent_connected` | Agent client established streaming connection and authenticated | `agent_id`, `auth_type`, `transport` | [FEAT-001](features/FEAT-001.md), [FEAT-009](features/FEAT-009.md) |
| EVT-003 | `tool_call_intercepted` | JSON-RPC `tools/call` received and dispatched to policy engine | `agent_id`, `tool_name`, `request_size_bytes` | [FEAT-001](features/FEAT-001.md) |
| EVT-004 | `tool_call_allowed` | Tool call cleared policy and forwarded upstream | `agent_id`, `tool_name`, `policy_id`, `eval_latency_ms` | [FEAT-002](features/FEAT-002.md) |
| EVT-005 | `tool_call_blocked` | Tool call rejected by policy rule or schema check | `agent_id`, `tool_name`, `rule_id`, `violation_type` | [FEAT-002](features/FEAT-002.md), [FEAT-012](features/FEAT-012.md) |
| EVT-006 | `anomaly_sequence_quarantined` | Tool-chaining exfiltration anomaly detected | `agent_id`, `sequence_pattern`, `quarantine_duration_sec` | [FEAT-004](features/FEAT-004.md) |
| EVT-007 | `secret_redacted` | Outbound DLP scrubber masked credential or PII | `agent_id`, `tool_name`, `redaction_type`, `token_count` | [FEAT-005](features/FEAT-005.md) |
| EVT-008 | `approval_ticket_created` | High-impact tool call suspended and routed to HITL queue | `ticket_id`, `agent_id`, `tool_name`, `channel` | [FEAT-006](features/FEAT-006.md) |
| EVT-009 | `approval_ticket_resolved` | Human reviewer approved or rejected ticket | `ticket_id`, `decision`, `duration_seconds` | [FEAT-006](features/FEAT-006.md) |
| EVT-010 | `audit_block_sealed` | Immutable audit block appended with SHA-256 hash | `block_index`, `block_hash`, `parent_hash` | [FEAT-007](features/FEAT-007.md) |
| EVT-011 | `kill_switch_tripped` | Emergency lockdown or automated circuit breaker triggered | `trigger_type`, `scope`, `target_agent_id` | [FEAT-010](features/FEAT-010.md) |
| EVT-012 | `compliance_report_exported` | Signed compliance attestation generated | `framework`, `record_count`, `signature_status` | [FEAT-011](features/FEAT-011.md) |

---

## Funnels & Conversion

| Funnel | Steps (event IDs) | Success event | Flow |
|---|---|---|---|
| Developer Activation | EVT-001 → EVT-002 → EVT-003 → EVT-004 | `tool_call_allowed` | [user-flow](user-flow.md) Local Developer Wrapping & Debugging |
| Governance & Approval | EVT-008 → EVT-009 → EVT-010 | `audit_block_sealed` | [user-flow](user-flow.md) Human-in-the-Loop Tool Interception & Approval |
| Enterprise Audit Readiness | EVT-005 → EVT-010 → EVT-012 | `compliance_report_exported` | [user-flow](user-flow.md) Incident Investigation & Compliance Audit Export |

**Conversion definitions:**
- **Developer Activation:** A developer who runs `aegis-cli wrap` and completes at least one successfully allowed tool call within 10 minutes of binary download.
- **Governance Resolution Rate:** Percentage of suspended Human-in-the-Loop tickets resolved (approved or rejected) prior to the 5-minute timeout window.
- **Enterprise Conversion:** A free open-core installation that connects > 5 agent services, activates the Web Console, and exports a compliance report within 90 days.

---

## Error & Performance Monitoring

| Signal | Source | Captured fields | Purpose |
|---|---|---|---|
| Proxy Latency Spike | Ingress Transport | `latency_ms`, `concurrency`, `transport`, `correlation_id` | Enforce < 5ms P99 latency SLA ([NFR-001](nfr.md)) |
| Unhandled JSON-RPC Parse Error | Protocol Parser | `payload_snippet`, `transport`, `error_code`, `client_ip` | Catch protocol incompatibilities and malformed frames |
| Upstream Server Timeout | Upstream Client | `tool_name`, `server_host`, `elapsed_ms`, `correlation_id` | Identify slow, unresponsive, or crashed MCP tool servers |
| High-Severity Policy Violation | Policy Engine | `agent_id`, `rule_id`, `violation_type`, `correlation_id` | Alert on active indirect prompt injection attempts |
| DLP Scrubber Buffer Ceiling Breach | DLP Scrubber | `payload_size_bytes`, `tool_name`, `truncated_boolean` | Monitor responses exceeding memory threshold ([NFR-025](nfr.md)) |

**Correlation:** Every inbound request is assigned a unique `correlation_id` propagated through policy evaluation, upstream dispatch, DLP redaction, and audit ledger append. If an upstream tool call fails or is blocked, the `correlation_id` links the client response directly to the corresponding forensic entry in the audit log ([FEAT-007](features/FEAT-007.md)).

---

## Business Reporting

| Report / Dashboard | Audience | Metrics | Refresh |
|---|---|---|---|
| Fleet Threat Overview | SecOps, CISO | Blocked violations by category, prompt injection count, quarantined agents | Real-time |
| Gateway Performance & SLA | Platform Engineering | P50/P95/P99 latency, RPS throughput, active virtual thread count | 1 minute |
| Human Oversight Efficiency | Compliance, SecOps | Average approval latency, approval vs rejection ratio, timeout rate | Hourly |
| Commercial Adoption | Product, Founders | Active cluster nodes, community-to-paid conversion, enterprise seat utilization | Daily |

---

## Analytics Data Governance

- **Zero PII in Telemetry:** Telemetry payloads capture only cryptographic hashes, agent SPIFFE IDs, tool names, and latency numbers. Prompt text and raw parameter arguments are strictly excluded from analytics events.
- **Retention:** Real-time operational metric timeseries are retained in Prometheus/ClickHouse for 90 days, then downsampled to daily aggregates retained for 365 days ([NFR-041](nfr.md)).
- **Deletion Propagation:** Any GDPR/CCPA subject deletion request deletes the mapping between human reviewer identities and historical ticket actions across all analytics stores ([schema](schema.md)).
- **Sampling:** Operational latency metrics are captured with 100% fidelity up to 10,000 RPS. Telemetry from non-violating health check pings is sampled at 1%.
- **Access Control:** Raw analytics streams are accessible strictly to authenticated administrators with `telemetry:read` RBAC permissions via the Web Security Console ([screens](screens.md)).

---

## Measurement & Verification

| Check | Method | When |
|---|---|---|
| Event Schema Conformance | Automated validation against JSON Schema contracts in CI | Every build |
| Sensitive Data Leak Check | Automated scanner verifying zero plain text credentials in analytics streams | Every release |
| Funnel Conversion Health | Weekly automated reconciliation of activation funnels against unique cluster IDs | Weekly |
| Latency Budget Assertion | Automated Gatling test ensuring telemetry emission adds < 0.2ms overhead | Every CI build |
