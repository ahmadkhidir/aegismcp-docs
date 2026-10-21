# Risk Register & Risk Log

## Risk Assessment Scale

### Likelihood
| Score | Label | Definition |
|---|---|---|
| 1 | Rare | < 5% chance in the project timeline |
| 2 | Unlikely | 5-20% chance |
| 3 | Possible | 20-50% chance |
| 4 | Likely | 50-80% chance |
| 5 | Almost Certain | > 80% chance |

### Impact
| Score | Label | Definition |
|---|---|---|
| 1 | Negligible | Minor inconvenience, easily absorbed |
| 2 | Minor | Some disruption, manageable with existing resources |
| 3 | Moderate | Significant disruption, requires dedicated effort to resolve |
| 4 | Major | Threatens timeline, budget, or core functionality materially |
| 5 | Critical | Threatens product viability, security posture, or company reputation |

### Risk Score
**Risk Score = Likelihood × Impact**

| Score Range | Severity | Action |
|---|---|---|
| 1-4 | Low | Monitor during regular milestone reviews |
| 5-9 | Medium | Plan proactive mitigation, monitor regularly |
| 10-19 | High | Active mitigation required, escalate to leadership |
| 20-25 | Critical | Immediate action required, potential product pivot |

---

## Risk Register

### Market Risks

| ID | Risk | L | I | Score | Mitigation | Contingency | Owner |
|---|---|---|---|---|---|---|---|
| R-001 | Incumbent API gateway vendors (e.g. Kong, Cloudflare) bundle native MCP tool filtering into existing enterprise edge fabrics | 3 | 4 | 12 | Focus deeply on protocol-native features generic WAFs cannot replicate: stateful sequence anomaly detection ([FEAT-004](features/FEAT-004.md)), out-of-band Slack approvals ([FEAT-006](features/FEAT-006.md)), and sub-millisecond AST policy evaluation | Position AegisMCP as a specialized agentic security layer that integrates alongside existing edge gateways as a dedicated Kubernetes sidecar | Product Lead |
| R-002 | Zero initial enterprise brand awareness delays procurement security reviews and pilot-to-paid conversions against established vendors | 4 | 3 | 12 | Publish open-source benchmarks proving sub-5ms P99 performance; provide automated EU AI Act compliance export packs ([FEAT-011](features/FEAT-011.md)) to shorten compliance review cycles | Offer white-glove proof-of-concept deployment assistance and 60-day performance guarantees for early enterprise design partners | Product Lead |
| R-003 | Model providers advertise improved internal model alignment, creating enterprise complacency regarding external tool firewalls | 3 | 3 | 9 | Publish empirical security research demonstrating that prompt injection and confused-deputy attacks succeed regardless of model alignment; emphasize that legal frameworks (EU AI Act Article 14) mandate independent technical oversight | Pivot marketing messaging from "prompt guardrails" to "zero-trust tool perimeter defense and legal compliance" | DevSecOps Lead |

### Technical Risks

| ID | Risk | L | I | Score | Mitigation | Contingency | Owner |
|---|---|---|---|---|---|---|---|
| R-004 | Upstream Model Context Protocol specification changes introduce breaking changes to JSON-RPC transport and schema standards | 2 | 4 | 8 | Active participation in open MCP working groups; isolate transport parsers behind clean Java adapter interfaces so updates require only transport layer revisions | Maintain versioned protocol decoders supporting legacy and new MCP specifications concurrently | Tech Lead |
| R-005 | Core proxy added latency exceeds the 5ms P99 SLA budget ([NFR-001](nfr.md)) under high-concurrency streaming loads | 2 | 5 | 10 | Implement zero-copy buffer slicing via Netty and Java 21 Virtual Threads; run automated Gatling load tests on every build to catch latency regressions immediately | Re-profile hot paths with async profiling; offload audit hash computation to background worker threads | Tech Lead |
| R-006 | Tool schema coverage gaps for niche or proprietary enterprise internal MCP servers | 3 | 3 | 9 | Implement dynamic parameter schema inference and provide an open-source schema contribution repository with pre-built templates for common databases and enterprise APIs | Fall back to generic strict type checks and basic command injection heuristics when schema is undefined | Tech Lead |

### Operational Risks

| ID | Risk | L | I | Score | Mitigation | Contingency | Owner |
|---|---|---|---|---|---|---|---|
| R-007 | Lean engineering bandwidth divides focus between the high-performance core Java proxy and the Web Admin Console | 3 | 4 | 12 | Prioritize Phase 1 core proxy engine and CLI first ([roadmap](roadmap.md)); use modular frontend component tokens ([branding](branding.md)) to accelerate Web Console build in Phase 2 | Release early beta with CLI-only management, onboarding early adopters while Web Console completes | Tech Lead |

### Legal & Compliance Risks

| ID | Risk | L | I | Score | Mitigation | Contingency | Owner |
|---|---|---|---|---|---|---|---|
| R-008 | Regulatory updates to EU AI Act or NIST AI RMF question the sufficiency of automated tamper-evident audit records | 2 | 4 | 8 | Track European AI Office and NIST guidance continuously; design audit ledger schema with extensible metadata fields matching statutory record-keeping mandates ([NFR-041](nfr.md)) | Engage specialized European regulatory counsel to issue formal compliance opinion letters for enterprise buyers | Compliance Officer |

### Financial Risks

| ID | Risk | L | I | Score | Mitigation | Contingency | Owner |
|---|---|---|---|---|---|---|---|
| R-009 | Free open-core community users fail to convert to paid Team Pro or Enterprise tiers due to sufficiency of self-hosted local features | 3 | 4 | 12 | Establish clean feature gates ([monetization](monetization.md)): keep team-scale capabilities (centralized fleet policy sync, Slack HITL, compliance exports) strictly in paid tiers | Introduce usage-based pricing for high-volume tool call fleets or discount bundles for annual commitments | Product Lead |

---

## Risk Trends

| Date | Change | Risk ID | Description |
|---|---|---|---|
| 2026-10-20 | Added | R-001 | Initial entry: Incumbent API gateway competition |
| 2026-10-20 | Added | R-002 | Initial entry: Low initial enterprise brand awareness |
| 2026-10-20 | Added | R-003 | Initial entry: Model provider alignment complacency |
| 2026-10-20 | Added | R-004 | Initial entry: Upstream MCP protocol specification churn |
| 2026-10-20 | Added | R-005 | Initial entry: Proxy latency SLA breach risk under load |
| 2026-10-20 | Added | R-006 | Initial entry: Tool schema coverage gaps for custom tools |
| 2026-10-20 | Added | R-007 | Initial entry: Engineering bandwidth division between proxy and console |
| 2026-10-20 | Added | R-008 | Initial entry: EU AI Act regulatory compliance changes |
| 2026-10-20 | Added | R-009 | Initial entry: Open-core community conversion risk |

---

## Review Cadence

- **Weekly:** Review all High risks (R-001, R-002, R-005, R-007, R-009) in engineering standup.
- **Monthly:** Full risk register review with product and compliance stakeholders.
- **Per Milestone:** Re-assess likelihood, impact scores, and mitigation progress prior to launching each roadmap phase.
